---
layout: post
title: "Linux上MySQL的管理配置"
date: 2016-09-14
author: Vincent, Dong
category: 服务器
tags: MySQL Linux 服务器
finished: true
---

## 配置MySQL远程登陆

先登陆到MySQL服务

`$ mysql -u root -p`

然后选择使用mysql数据库

`mysql> use mysql;`

### 创建/删除用户：

团队开发过程中，可能会需要共同使用一个数据库，这就需要一个专有数据库用户用于团队开发。或在生产环境中，我们应有专门的用户来提供服务，而不应该直接使用root用户提供数据库服务。

这里我们新建一个dev的用户用于团队开发。

`mysql> CREATE USER 'dev'@'%' IDENTIFIED BY 'password';`

同时删除可远程登陆的root用户，使得root用户只可本机登陆：

`mysql> DROP USER 'root'@'%';`

### 用户授权：

使用GRANT命令修改 用户/权限 信息

`mysql> GRANT ALL ON *.* TO 'root'@'localhost' IDENTIFIED BY 'pwd' WITH GRANT OPTIONS;`

这个命令的基本格式是这样的：

`mysql> GRANT 权限 ON 库.表 TO '用户'@'服务器IP' IDENTIFIED BY '密码' WITH GRANT OPTIONS;`

命令最后的WITH GRANT OPTIONS表示授予该用户授权其他用户的权限。这里如果是给普通用户授权，则不需要加该部分。

这里我们来给刚创建的dev授予权限：

`mysql> GRANT ALL ON myuniuni.* TO 'dev'@'%';`

修改用户权限之后，我们需要刷新权限缓存，才能够使刚刚的修改生效。

`mysql> flush privileges;`

当然，你也可以收回你授予用户的权限，通过revoke命令来实现，它的格式是这样：

`mysql> REVOKE 权限 ON 库.表 FROM '用户'@'服务器IP';`

### 配置访问ip

配置MySQL不再只允许本地访问：

`$ sudo vim /etc/mysql/my.cnf`

找到bind-address  = 127.0.0.1

将其注释掉，或改为：bind-address  = 0.0.0.0

重启mysql： 

`$ sudo service mysql restart`

### 远程登陆测试

在另一台计算机上远程登陆mysql：

`$ mysql -h 服务器IP -u dev -P 3306 -p`

输入密码后，即可进入mysql命令行。

## 修改用户密码

如果我们需要修改某一个用户的密码，可以直接使用SET PASSWORD命令进行修改。

`SET PASSWORD FOR 'dev'@'%' = PASSWORD('newpassword');`

如果是修改当前登陆用户，可以直接使用：

`SET PASSWORD = PASSWORD('newpassword');`

## MySQL备份与恢复

### MySQL数据备份

#### 备份一个数据库

mysqldump基本语法：

mysqldump -u username -p dbname table1 table2 ...-> BackupName.sql

dbname参数表示数据库的名称；
table1和table2参数表示需要备份的表的名称，为空则整个数据库备份；
BackupName.sql参数表设计备份文件的名称，文件名前面可以加上一个绝对路径。通常将数据库被分成一个后缀名为sql的文件；
使用root用户备份test数据库下的person表

`mysqldump -u root -p test person > D:\backup.sql`

#### 备份多个数据库

mysqldump -u username -p --databases dbname2 dbname2 > Backup.sql
加上了--databases选项，然后后面跟多个数据库

`mysqldump -u root -p --databases test mysql > D:\backup.sql`

#### 备份所有数据库

mysqldump命令备份所有数据库的语法如下：

mysqldump -u username -p -all-databases > BackupName.sql

`mysqldump -u -root -p -all-databases > D:\all.sql`

### MySQL数据还原

还原使用mysqldump命令备份的数据库的语法如下：

mysql -u root -p [dbname] < backup.sql

`mysql -u root -p < C:\backup.sql`