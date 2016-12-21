---
layout: post
title: "Linux上配置PHP使用Redis共享Session"
date: 2016-09-14
author: Vincent, Dong
category: 服务器
tags: Redis Linux 服务器
finished: true
---

## 安装Redis

`$ sudo apt-get update`
`$ sudo apt-get install redis-server`

安装完成后，Redis服务器会自动启动，我们检查Redis服务器程序，执行

`$ ps -aux|grep redis`

在终端中通过启动命令检查Redis服务器状态

`$ netstat -nlt|grep 6379`

显示: tcp 0 0 127.0.0.1:6379 0.0.0.0:* LISTEN

`$ redis-cli`

`redis 127.0.0.1:6379> ping`
`PONG`

## 配置Redis

`$ sudo vi /etc/redis/redis.conf`

默认情况下，访问Redis服务器是不需要密码的，为了增加安全性我们需要设置Redis服务器的访问密码。

取消注释requirepass

`requirepass my_password`

默认情况下，Redis服务器不允许远程访问，只允许本机访问，所以我们需要设置打开远程访问的功能。

`bind 127.0.0.1` 改为 `bind 0.0.0.0`

修改后，重启Redis服务器。

`$ sudo /etc/init.d/redis-server restart`

登陆Redis服务器，输入密码

`$ redis-cli -a my_password`

## 安装Redis的PHP扩展

`$ sudo apt-get install php5-redis`

重启Apache服务器

`sudo service apache2 restart`

在phpinfo()中查看redis是否正常启动

## 配置php的session

### 配置服务器的session设置

`$ sudo vim /etc/php5/apache2/php.ini`

在命令行模式下搜索字符串session
`/session`

按 n 查找下一个匹配单词

修改:
`session.save_handler = redis`
`session.save_path = "tcp://127.0.0.1:6379/0?auth=my_password"`

为了在二级域名之间共享session，需要设置
`session.cookie_domain = ".domain.com"`

重启Apache服务器

`sudo service apache2 restart`

### Laravel框架下的session设置

安装laravel的redis扩展包

在composer.json的require中添加
`"predis/predis": "~1.0"`

然后在项目路径下执行
`$ composer update`

修改 config/database.php，在 redis 选项内增加 session 选项，并把 database 修改为 0：
'session' => [
     'host'     => env('REDIS_HOST', '127.0.0.1'),
     'password' => env('REDIS_PASSWORD', 'my_password'),
     'port'     => env('REDIS_PORT', 6379),
     'database' => 0,
],

修改 config/session.php

`'driver' => env('SESSION_DRIVER', 'redis'),`
`'connection' => "session"`
`'domain' => ".domain.com",`

修改.env文件:
将其中的redis driver 从files改成redis

### laravel 与其他框架共享redis session
如果你需要让你的session在两个不同的框架下共享，例如laravel和tp之间，那么就请不要使用laravel提供的Session接口，这里需要使用原生php的session操作才能实现：

`session_start();`
`print_r($_SESSION);`

并且，laravel框架在读取cookie中的`session_id`时，使用了中间件进行了加密，所以需要屏蔽Kernel.php中的`\App\Http\Middleware\EncryptCookies::class`,或者解密读取到`session_id`。