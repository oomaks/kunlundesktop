本文讲解基于'Ubuntu 16.04 Server'搭建'electron-release-server'服务器，用于产品环境

## 1. 安装Git 
```Bash
apt-get update
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
ENV APP_USERNAME admin # electron-release-server 用户名
ENV APP_PASSWORD admin # electron-release-server 密码
ENV APP_URL http://nocknock.cn:81 # electron-release-server 所在服务器域名和端口
ENV DB_HOST nocknock.cn # postgres docker 所在服务器域名
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
## 3.安装`Docker`,配置并启动`Postgres`数据库
```Bash
apt-get update
apt-get install docker
docker pull postgres:9.6
# 此处应将`123456`修改为自己设定的密码，此密码为数据库管理员密码
docker run --name postgres-for-electron-release-server -e POSTGRES_PASSWORD=123456 -p 5431:5432 -d postgres:9.6
```
在本地操作系统安装`Postgres 9.6+`并打开`pgAdmin`连接到Docker数据库，用户名为`postgres`,密码为`123456`，并执行以下SQL:
```SQL
CREATE ROLE electron_release_server_user ENCRYPTED PASSWORD '123456' LOGIN;
CREATE DATABASE electron_release_server OWNER "electron_release_server_user";
CREATE DATABASE electron_release_server_sessions OWNER "electron_release_server_user";
```
选中`electron-release-server-sessions`数据库,并执行以下代码
```SQL
/*
minimal support table and functions in PostgreSQL database
for real use you probably want to at least implement some kind of expiry policy
and additionally put some data fields in their own table fields for easier
manipulation
*/


-- minimal table for session store
CREATE TABLE sails_session_store(
    sid text PRIMARY KEY,
    data json NOT NULL,
    created_at timestamp  NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- set data for session
CREATE OR REPLACE FUNCTION sails_session_store_set(sid_in text, data_in json)
RETURNS void
LANGUAGE plpgsql
AS $$
BEGIN
  -- delete current session data if it exists so the next insert succeeds
  DELETE FROM sails_session_store WHERE sid = sid_in;
  INSERT INTO sails_session_store(sid, data) VALUES(sid_in, data_in);
END;
$$;

-- get stored session
CREATE OR REPLACE FUNCTION sails_session_store_get(sid_in text, OUT data_out json)
LANGUAGE plpgsql
AS $$
BEGIN
  SELECT data FROM sails_session_store WHERE sid = sid_in INTO data_out;
END;
$$;

-- destroy session
CREATE OR REPLACE FUNCTION sails_session_store_destroy(sid_in text)
RETURNS void
LANGUAGE plpgsql
AS $$
BEGIN
  DELETE FROM sails_session_store WHERE sid = sid_in;
END;
$$;

-- count sessions
CREATE OR REPLACE FUNCTION sails_session_store_length(OUT length int)
LANGUAGE plpgsql
AS $$
BEGIN
  SELECT count(*) FROM sails_session_store INTO length;
END;
$$;

-- delete all sessions
CREATE OR REPLACE FUNCTION sails_session_store_clear()
RETURNS void
LANGUAGE plpgsql
AS $$
BEGIN
  DELETE FROM sails_session_store;
END;
$$;
```



