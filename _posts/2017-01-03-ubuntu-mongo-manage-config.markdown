---
layout: post
title: "ubuntu下的mongo安装及配置管理"
date: 2017-01-03
author: Vincent, Dong
category: 服务器
tags: MongoDB Linux 服务器
finished: true
---

## 安装 MongoDB

1. 安装key
`$ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6`

2. 添加源
`$ echo "deb [ arch=amd64 ] http://repo.mongodb.com/apt/ubuntu trusty/mongodb-enterprise/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-enterprise.list`

3. 更新源
`$ sudo apt-get update`

4. 安装mongodb
`$ sudo apt-get install -y mongodb-enterprise`

具体详情可见官方文档:
https://docs.mongodb.com/master/tutorial/install-mongodb-on-ubuntu/?_ga=1.104487360.1319058345.1434008003

`sudo service mongod start`

### 关闭／启动

`$ sudo service mongodb stop`
`$ sudo service mongodb start`

### 安装 PHP Mongodb Driver

安装 PECL
`$ sudo apt-get install php-pear php5-dev`

安装 Driver
`$ sudo pecl install mongo`

遇到询问提示，输入no

开启 PHP 扩展
`$ echo 'extension=mongo.so' | sudo tee /etc/php5/mods-available/mongo.ini`

`$ sudo ln -s /etc/php5/mods-available/mongo.ini /etc/php5/apache2/conf.d/mongo.ini`

`$ sudo ln -s /etc/php5/mods-available/mongo.ini /etc/php5/cli/conf.d/mongo.ini`

重启服务器
`$ sudo service apache2 restart`

验证是否安装成功
`$ php -i | grep 'Mongo'`

MongoDB Support => enabled

### 创建数据库用户

db.createUser({
    user: "root",
    pwd: "111111",
    roles: [ { role: "root", db: "admin" } ]
})

#### MongoDB 内置角色

    1. 数据库用户角色：read、readWrite;
    2. 数据库管理角色：dbAdmin、dbOwner、userAdmin；
    3. 集群管理角色：clusterAdmin、clusterManager、clusterMonitor、hostManager；
    4. 备份恢复角色：backup、restore；
    5. 所有数据库角色：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、dbAdminAnyDatabase
    6. 超级用户角色：root  
    // 这里还有几个角色间接或直接提供了系统超级用户的访问（dbOwner 、userAdmin、userAdminAnyDatabase）
    7. 内部角色：__system

    Read：允许用户读取指定数据库
    readWrite：允许用户读写指定数据库
    dbAdmin：允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile
    userAdmin：允许用户向system.users集合写入，可以找指定数据库里创建、删除和管理用户
    clusterAdmin：只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限。
    readAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读权限
    readWriteAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读写权限
    userAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的userAdmin权限
    dbAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限。
    root：只在admin数据库中可用。超级账号，超级权限

### 开启mongo的用户权限认证

`$ sudo vim /etc/mongod.conf`

/etc/mongodb.conf文件中

net:                                                                     
  port: 27017
  #bindIp:  127.0.0.1 如果你需要远程访问mongo数据库，这里需要注释掉bindIp
security:
  authorization: enabled # 添加用户权限验证访问

更改并保存配置文件后，重新启动mongod服务:

`$ sudo service mongodb restart`

开启用户权限认证后，进入admin数据库查询信息则提示权限错误

此时需要我们在当前数据库下进行用户验证：
`> db.auth("root", "111111")`

之后，再次查询，则成功显示查询信息。

至此，表明我们的mongo数据库权限认证功能已经成功开启。

### 数据库的备份与恢复

备份:
`mongodump -d database -c collection -o /path/to/dump`

恢复:
`mongorestore -d database -u user -p pwd --authenticationDatabase=admin /path/to/restore`

### 数据库迁移

`db.copyDatabase(fromdb, todb, fromhost, username, password);`