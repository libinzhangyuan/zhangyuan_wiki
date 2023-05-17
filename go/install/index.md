### 用gvm来控制多版本。
```
注意，ubuntu笔记本下，需要先安装老版本，再安装新版本，不然会失败:
gvm install go1.7.1 -B
gvm use go1.7.1
export GOROOT_BOOTSTRAP=$GOROOT

gvm install  go1.18.10

再
gvm install  go1.20.4



```