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
编辑当前目录中的`Dockfile`文件并将内容替换为以下内容：<br/>
```Bash
FROM node:8.11.1

ENV LOG_LEVEL INFO
ENV APP_USERNAME admin
ENV APP_PASSWORD admin
ENV APP_URL http://nocknock.cn:81
ENV DB_HOST nocknock.cn
ENV DB_PORT 5431
ENV DB_USERNAME electron_release_server_user
ENV DB_PASSWORD 123456
ENV DB_NAME electron_release_server
ENV DB_SESSION_NAME electron_release_server_sessions
ENV TOKEN_SECRET Cnnv9Xn0TIUcuTx9IbnOptphKdZDmK9lzBIwp3BXBQYzQ3NkCUuS5QOFD64kwLW

RUN echo "Asia/Shanghai" > /etc/timezone && dpkg-reconfigure -f noninteractive tzdata

# Create app directory
RUN mkdir -p /usr/src/electron-release-server
RUN mkdir -p /home/postgres/tmp
WORKDIR /usr/src/electron-release-server

# Install app dependencies
COPY . /usr/src/electron-release-server

COPY config/docker.js config/local.js

EXPOSE 81

CMD [ "npm", "start"]
```
