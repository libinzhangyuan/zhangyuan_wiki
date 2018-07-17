## 列表
* [java](java/index)
* [unix](unix/index)
* [mongodb](mongodb/index)
* [算法](algorith/index)

<br/>
<br/>
<br/>
-------------------------------------------------------------------
<br/>

## 搭建wiki:

依赖:
Mac:brew install icu4c
Ubuntu:sudo apt-get install libicu-dev
安装ruby:

- 安装rvm:

* gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3

* \curl -sSL https://get.rvm.io | bash

- 安装ruby 2.3.1

* rvm install ruby 2.3.1

* 进入clone下来的wiki根项目执行:

>
```
rvm use 2.3.1 && rvm gemset create wiki && cd . && gem install bundle && bundle
```

- 在wiki根目录下执行"**gollum -p 4569**"启动wiki, 浏览器**localhost:4569**访问即可
- 可以配置nginx将80端口导向4569

- 每次启动在wiki根目录下执行：git pull origin master && gollum ., 每次编辑完，需要执行git push到仓库和大家同步
- 因为是基于git搭建，所以自己要搭建好git的username和email,配置用户信息可参考[git官网](https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%88%9D%E6%AC%A1%E8%BF%90%E8%A1%8C-Git-%E5%89%8D%E7%9A%84%E9%85%8D%E7%BD%AE),不然不能识别编辑者
- gollum .常见配置参数(gollum . --??):

> --live-preview:启动实时预览

- 更多请查看gollum --help或者[官方文档](https://github.com/gollum/gollum)


## 搭建wiki (docker版): 1
