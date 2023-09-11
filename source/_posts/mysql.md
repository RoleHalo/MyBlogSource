---
title: MySQL学习笔记
date: 2020-08-03 18:18:12
tags: MySQL
toc: true
---
### MySQL
1. MySQL可以解压到U盘里
2. 打开解压的文件夹`K:/softwareInstall/mysql-8.0.20-winx64` ，在该文件夹下创建 my.ini 配置文件，编辑 my.ini 配置以下基本信息：
```bash
[client]
# 设置mysql客户端默认字符集
default-character-set=utf8
 
[mysqld]
# 设置3306端口
port = 3306
# 设置mysql的安装目录
basedir=K:/softwareInstall/mysql-8.0.20-winx64
# 设置 mysql数据库的数据的存放目录，MySQL 8+ 不需要以下配置，系统自己生成即可，否则有可能报错
datadir=K:/softwareInstall/mysql-8.0.20-winx64/data
# 允许最大连接数
max_connections=20
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
```
3. 启动下 MySQL 数据库：`cd K:\softwareInstall\mysql-8.0.11\bin` `mysqld --initialize --console`

在 Windows 系统下，以`管理员`身份打开命令窗口(cmd)，进入 MySQL 安装目录的 bin 目录。
```
启动：
cd K:/softwareInstall/mysql-8.0.20-winx64/bin
mysqld --console

关闭：
cd K:/softwareInstall/mysql-8.0.20-winx64/bin
mysqladmin -uroot shutdown
```
4. 执行完成后，会输出 root 用户的初始默认密码，如：
```bash
2018-04-20T02:35:05.464644Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: RqUDv!g8j%D;
```

`RqUDv!g8j%D;`就是初始密码，后续登录需要用到，你也可以在登陆后修改密码。
5. 输入安装命令：`mysqld install`将`mysql`服务添加到系统的`服务`中，这样下面的命令`net start mysql`才会生效。
6. 启动输入命令即可：`net start mysql` 停止服务`net stop mysql`
7. 注意: 在`5.7`版本中需要初始化 data 目录:
```bash
cd K:/softwareInstall/mysql-8.0.20-winx64/bin 
mysqld --initialize-insecure 
```
8. 登录MySQL，当 MySQL 服务已经运行时, 我们可以通过 MySQL 自带的客户端工具登录到 MySQL 数据库中, 首先打开命令提示符, 输入以下格式的命名:
```bash
mysql -h 主机名 -u 用户名 -p -h myserver -P 端口

参数说明：
    -h : 指定客户端所要登录的 MySQL 主机名, 登录本机(localhost 或 127.0.0.1)该参数可以省略;
    -u : 登录的用户名;
    -p : 告诉服务器将会使用一个密码来登录, 如果所要登录的用户名密码为空, 可以忽略此选项。
    
如果我们要登录本机的 MySQL 数据库，只需要输入以下命令即可：
mysql -u root -p
输入 exit 或 quit 退出登录。
```
