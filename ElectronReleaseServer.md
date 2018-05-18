本文讲解基于'Ubuntu 16.04 Server'搭建'electron-release-server'服务器，用于产品环境

## 1. 安装Git 
```Bash
apt-get install git -y
git config --global user.name tom
git config --global user.email tom@21esn.com
```

## 2.下载`electron-release-server`的源代码
```Bash
git clone https://github.com/ArekSredzki/electron-release-server.git
cd electron-release-server
git checkout v1.4.3
```
执行以下命令将文件中的`port:80`改为`port:81`<br/>
```Bash
vi config/env/production.js
```
执行以下命令将文件中的`port:80`改为`port:81`，将第47行的`DB_NAME`改为`DB_SESSION_NAME`<br/>
```Bash
vi config/docker.js
```
