在多台机器（如2台Ubuntu + 1台macOS）之间通过**网络文件系统（NFS/SMB）** 实现目录共享时，核心思路是构建“**中心化共享架构**”——选择一台机器作为“文件服务器”，将目标目录共享出去，其他机器作为“客户端”，通过网络挂载该共享目录，实现像操作本地文件一样读写共享内容，且所有客户端的修改会实时反映在服务器和其他客户端上。

以下是针对NFS和SMB两种协议的**详细实现步骤**（以“1台Ubuntu作为服务器，1台Ubuntu + 1台macOS作为客户端”为例），包含环境准备、服务器配置、客户端挂载、开机自动挂载等关键环节。


## 一、核心概念与架构说明
在开始配置前，需明确以下基础信息，避免后续操作出错：
```
| 角色       | 设备类型   | IP地址（示例） | 核心作用                     |
|------------|------------|----------------|------------------------------|
| 服务器     | Ubuntu 22.04 | 192.168.1.100  | 存储共享目录，提供NFS/SMB服务 |
| 客户端1    | Ubuntu 22.04 | 192.168.1.101  | 挂载共享目录，读写文件       |
| 客户端2    | macOS Sonoma | 192.168.1.102  | 挂载共享目录，读写文件       |
```
**核心逻辑**：所有客户端不直接交互，而是通过服务器“中转”——客户端1修改文件后，修改会实时写入服务器；客户端2访问时，直接从服务器读取最新版本，实现“双向同步”的效果（本质是“共享存储”而非“文件复制同步”）。


## 二、方案1：NFS协议（推荐Linux/macOS跨平台）
NFS（Network File System）是Linux系统原生支持的网络文件协议，稳定性高、性能好，macOS也内置了NFS客户端，适合Linux与macOS混合环境。


### 步骤1：配置NFS服务器（Ubuntu）
#### 1.1 安装NFS服务器软件
Ubuntu默认未预装NFS服务器，需手动安装：
```bash
# 更新软件源
sudo apt update
# 安装NFS服务器（nfs-kernel-server是核心服务）
sudo apt install -y nfs-kernel-server
```

#### 1.2 创建共享目录
选择一个目录作为共享存储（例如`/home/user/shared_code`，用于存放代码），并设置正确的权限（确保客户端有读写权限）：
```bash
# 1. 创建共享目录（路径可自定义，建议放在用户目录下，避免权限问题）
mkdir -p /home/user/shared_code

# 2. 设置权限（关键！确保客户端用户能读写）
# 方式1：开放所有权限（测试用，不推荐生产环境）
sudo chmod -R 777 /home/user/shared_code
# 方式2：指定用户组权限（推荐，假设服务器用户UID/GID为1000）
sudo chown -R 1000:1000 /home/user/shared_code
sudo chmod -R 755 /home/user/shared_code  # 所有者可读写，其他用户只读
# 若需客户端可写，可调整为775（需确保客户端用户属于同一组）
```

#### 1.3 配置NFS共享规则
NFS通过`/etc/exports`文件定义“哪些目录共享给哪些客户端”，以及共享权限：
```bash
# 编辑exports文件
sudo nano /etc/exports
```

在文件末尾添加以下内容（根据实际IP和目录修改）：
```ini
# 格式：共享目录  客户端IP1(权限)  客户端IP2(权限)
/home/user/shared_code  192.168.1.101(rw,sync,no_subtree_check,all_squash,anonuid=1000,anongid=1000)
/home/user/shared_code  192.168.1.102(rw,sync,no_subtree_check,all_squash,anonuid=1000,anongid=1000)
```

**权限参数说明**（避免权限拒绝的关键）：
```
- `rw`：允许客户端读写共享目录（`ro`为只读）；
- `sync`：客户端写入时，服务器先将数据写入磁盘再返回成功（保证数据安全，避免丢数据）；
- `no_subtree_check`：关闭子目录检查，提升性能（共享整个目录时推荐）；
- `all_squash`：将客户端所有用户映射为服务器的“匿名用户”（避免客户端用户ID与服务器不匹配导致的权限问题）；
- `anonuid=1000`/`anongid=1000`：指定匿名用户的UID/GID（需与服务器共享目录的所有者UID/GID一致，通过`id user`命令查看）。
```
#### 1.4 生效NFS配置并启动服务
```bash
# 重新加载exports配置（无需重启服务，即时生效）
sudo exportfs -ra

# 启动NFS服务并设置开机自启
sudo systemctl start nfs-kernel-server
sudo systemctl enable nfs-kernel-server

# 验证NFS服务状态（确保active）
sudo systemctl status nfs-kernel-server
```

#### 1.5 开放防火墙端口（关键！避免客户端连接失败）
Ubuntu默认启用`ufw`防火墙，需开放NFS相关端口（NFS使用2049端口，且依赖rpcbind服务的111端口）：
```bash
# 开放NFS和rpcbind端口
sudo ufw allow 111/tcp
sudo ufw allow 2049/tcp
sudo ufw allow 111/udp
sudo ufw allow 2049/udp

# 重新加载防火墙规则
sudo ufw reload

# 验证端口是否开放
sudo ufw status
```


### 步骤2：配置NFS客户端（Ubuntu客户端）
#### 2.1 安装NFS客户端
Ubuntu客户端需安装NFS客户端工具才能挂载共享目录：
```bash
sudo apt update
sudo apt install -y nfs-common
```

#### 2.2 创建本地挂载点
在客户端上创建一个目录，用于“挂载”服务器的共享目录（例如`/home/user/mnt/shared_code`）：
```bash
mkdir -p /home/user/mnt/shared_code
```

#### 2.3 手动挂载NFS共享目录
```bash
# 格式：sudo mount -t nfs 服务器IP:共享目录 本地挂载点
sudo mount -t nfs 192.168.1.100:/home/user/shared_code /home/user/mnt/shared_code
```

#### 2.4 验证挂载是否成功
```bash
# 1. 查看挂载状态（若有192.168.1.100:/home/user/shared_code的记录，说明成功）
df -h

# 2. 测试读写（在客户端挂载点创建文件，查看服务器是否同步）
echo "test from ubuntu client" > /home/user/mnt/shared_code/test_ubuntu.txt
# 登录服务器，查看文件是否存在：ls /home/user/shared_code/
```

#### 2.5 配置开机自动挂载（避免重启后需手动挂载）
通过`/etc/fstab`文件实现开机自动挂载：
```bash
# 编辑fstab文件
sudo nano /etc/fstab

# 在文件末尾添加以下内容（格式：服务器IP:共享目录 本地挂载点 nfs defaults 0 0）
192.168.1.100:/home/user/shared_code  /home/user/mnt/shared_code  nfs  defaults  0  0
```

**参数说明**：
- `defaults`：包含`rw`（读写）、`suid`（保留SUID权限）、`dev`（保留设备文件）等默认选项，满足大多数场景；
- 最后两个`0`：分别表示“是否dump备份”和“是否开机检查磁盘”，NFS共享目录无需这两项，设为0即可。

```bash
# 验证fstab配置是否正确（无报错则正常）
sudo mount -a
```


### 步骤3：配置NFS客户端（macOS客户端）
macOS内置NFS客户端，无需额外安装，直接挂载即可。

#### 3.1 创建本地挂载点
在macOS的`/Volumes`目录下创建挂载点（macOS推荐将外部挂载目录放在`/Volumes`下，便于管理）：
```bash
# 打开终端，执行以下命令（路径可自定义）
mkdir -p /Volumes/shared_code
```

#### 3.2 手动挂载NFS共享目录
```bash
# 格式：sudo mount -t nfs -o resvport 服务器IP:共享目录 本地挂载点
sudo mount -t nfs -o resvport 192.168.1.100:/home/user/shared_code /Volumes/shared_code
```

**关键参数`-o resvport`**：macOS默认使用“非保留端口”连接NFS服务器，而Ubuntu NFS服务器默认拒绝非保留端口的连接，添加`resvport`参数可强制使用保留端口（1024以下），避免连接失败。

#### 3.3 验证挂载是否成功
```bash
# 测试读写（在macOS挂载点创建文件，查看服务器是否同步）
echo "test from mac client" > /Volumes/shared_code/test_mac.txt
# 登录服务器，查看文件是否存在：ls /home/user/shared_code/
```

#### 3.4 配置开机自动挂载（macOS）
macOS不推荐修改`/etc/fstab`（可能导致系统启动异常），推荐使用“自动操作”或`mount_nfs`脚本实现开机挂载，这里介绍更简单的“登录时自动挂载”：
```
1. 打开macOS的“启动台” → “其他” → “自动操作”；
2. 新建“应用程序”，在左侧搜索“运行Shell脚本”并拖入右侧；
3. 在脚本框中输入：
   ```bash
   # 先检查挂载点是否已挂载，未挂载则执行挂载
   if ! mount | grep -q "/Volumes/shared_code"; then
     sudo mount -t nfs -o resvport 192.168.1.100:/home/user/shared_code /Volumes/shared_code
   fi
   ```
4. 保存应用程序（例如命名为`MountNFS`），放在`/Applications`目录下；
5. 打开“系统设置” → “通用” → “登录项”，点击“+”，添加刚才保存的`MountNFS`应用；
6. 首次运行时，会提示“是否允许终端执行”，选择“允许”，并输入macOS用户密码（sudo需要权限）。
```

## 三、方案2：SMB协议（适合Windows/macOS/Linux混合环境）
SMB（Server Message Block）是微软开发的协议，Windows原生支持，macOS和Linux也可通过工具支持。若后续可能加入Windows设备，推荐使用SMB；若仅Linux/macOS，NFS性能更优。

以下以“Ubuntu作为SMB服务器，Ubuntu + macOS作为客户端”为例，介绍关键配置步骤。


### 步骤1：配置SMB服务器（Ubuntu）
#### 1.1 安装SMB服务器软件
```bash
sudo apt update
sudo apt install -y samba samba-common-bin
```

#### 1.2 创建共享目录并设置权限
```bash
# 创建共享目录（与NFS共用同一目录即可）
mkdir -p /home/user/shared_code
# 设置权限（SMB用户需有读写权限）
sudo chown -R 1000:1000 /home/user/shared_code
sudo chmod -R 775 /home/user/shared_code
```

#### 1.3 添加SMB用户并配置共享规则
SMB需要独立的用户（需是Ubuntu系统已存在的用户），并通过`/etc/samba/smb.conf`配置共享：
```bash
# 1. 添加SMB用户（例如将ubuntu用户“user”添加为SMB用户，需设置SMB密码）
sudo smbpasswd -a user
# 按提示输入密码（与系统密码可不同，用于SMB登录）

# 2. 编辑SMB配置文件
sudo nano /etc/samba/smb.conf
```

在文件末尾添加共享规则：
```ini
[shared_code]  # 共享名称（客户端看到的名称）
    path = /home/user/shared_code  # 服务器共享目录路径
    available = yes  # 是否允许访问
    valid users = user  # 允许访问的SMB用户（即刚才添加的user）
    read only = no  # 允许读写（yes为只读）
    browsable = yes  # 允许在网络中被发现
    create mask = 0755  # 客户端创建文件的默认权限
    directory mask = 0755  # 客户端创建目录的默认权限
```

#### 1.4 重启SMB服务并开放防火墙
```bash
# 重启SMB服务
sudo systemctl restart smbd nmbd
sudo systemctl enable smbd nmbd  # 开机自启

# 开放SMB端口（SMB使用445端口）
sudo ufw allow 445/tcp
sudo ufw reload
```


### 步骤2：配置SMB客户端（Ubuntu）
#### 2.1 安装SMB客户端
```bash
sudo apt install -y cifs-utils
```

#### 2.2 手动挂载SMB共享
```bash
# 创建本地挂载点
mkdir -p /home/user/mnt/shared_code_smb

# 挂载SMB共享（格式：sudo mount -t cifs //服务器IP/共享名称 本地挂载点 -o username=SMB用户名）
sudo mount -t cifs //192.168.1.100/shared_code /home/user/mnt/shared_code_smb -o username=user
# 按提示输入SMB密码（刚才通过smbpasswd设置的密码）
```

#### 2.3 开机自动挂载（通过fstab）
```bash
sudo nano /etc/fstab

# 添加以下内容（格式：//服务器IP/共享名称 本地挂载点 cifs username=SMB用户,password=SMB密码,defaults 0 0）
//192.168.1.100/shared_code  /home/user/mnt/shared_code_smb  cifs  username=user,password=123456,defaults  0  0
```

**安全提示**：`fstab`文件是明文存储，直接写密码有安全风险。可将密码存放在单独文件中并设置权限：
```bash
# 1. 创建密码文件
sudo nano /etc/smb_credentials
# 写入：
username=user
password=123456

# 2. 设置权限（仅root可读）
sudo chmod 600 /etc/smb_credentials

# 3. 修改fstab
//192.168.1.100/shared_code  /home/user/mnt/shared_code_smb  cifs  credentials=/etc/smb_credentials,defaults  0  0
```


### 步骤3：配置SMB客户端（macOS）
```
macOS通过“访达”即可可视化挂载SMB共享，无需命令行：
1. 打开“访达”，按下`Cmd + K`（或点击菜单栏“前往” → “连接服务器”）；
2. 在“服务器地址”中输入：`smb://192.168.1.100/shared_code`，点击“连接”；
3. 选择“注册用户”，输入SMB用户名和密码，点击“连接”；
4. 挂载成功后，会在桌面显示“shared_code”磁盘，直接双击即可访问。
```
**开机自动挂载**：
```
1. 挂载成功后，打开“系统设置” → “通用” → “登录项”；
2. 将桌面的“shared_code”磁盘拖入“登录项”列表，下次登录时会自动挂载。
```

## 四、常见问题与解决方案
1. **客户端挂载时提示“权限被拒绝”**
   - 检查服务器共享目录权限：确保`chown`和`chmod`设置正确，UID/GID与客户端匹配（NFS的`anonuid`/`anongid`需对应）；
   - 检查防火墙：确保服务器开放了NFS（111、2049）或SMB（445）端口；
   - macOS NFS挂载：必须添加`-o resvport`参数，否则端口被拒绝。

2. **开机后未自动挂载**
   - 检查`/etc/fstab`（Linux）语法：确保路径、IP、参数无拼写错误，执行`sudo mount -a`验证；
   - macOS登录项：确保“自动操作”应用有执行权限，且sudo密码正确（可在“钥匙串访问”中保存密码）。

3. **客户端修改文件后，其他客户端未实时看到**
   - NFS缓存问题：客户端默认会缓存文件，可在挂载时添加`noac`参数（`-o noac`）关闭缓存，牺牲部分性能换取实时性；
   - SMB默认实时同步：若仍有延迟，检查服务器`smb.conf`的`oplocks`参数（关闭`oplocks = no`）。


## 五、方案对比与选择建议
```
| 协议 | 优点                          | 缺点                          | 适用场景                          |
|------|-------------------------------|-------------------------------|-----------------------------------|
| NFS  | Linux/macOS原生支持，性能高    | 不支持Windows（需额外工具）    | 纯Linux/macOS环境，对性能要求高    |
| SMB  | 跨Windows/macOS/Linux，兼容性好 | 性能略低于NFS，配置稍复杂      | 混合系统环境（含Windows）          |
```
**最终建议**：
- 若团队仅使用Ubuntu和macOS开发，优先选择**NFS**，配置简单且性能更优；
- 若需兼容Windows设备，或未来可能扩展Windows客户端，选择**SMB**，兼容性更强；
- 所有设备需在同一局域网内（或通过VPN打通内网），否则公网传输速度慢且不安全（需额外配置加密，如NFS over SSH、SMB over TLS）。