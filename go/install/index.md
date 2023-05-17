### 用gvm来控制多版本。
```
# 下面时笔记本电脑ubuntu 20.04.3 上的安装过程

需要先安装老版本，再安装新版本，不然会失败:
gvm install go1.7.1 -B
gvm use go1.7.1
export GOROOT_BOOTSTRAP=$GOROOT

再
gvm install  go1.18.10
gvm use  go1.18.10

再
gvm install  go1.20.4



```