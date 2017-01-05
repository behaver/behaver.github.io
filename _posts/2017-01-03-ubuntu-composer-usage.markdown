---
layout: post
title: "Ubuntu下Composer的使用"
date: 2017-01-03
author: Vincent, Dong
category: 服务器
tags: Composer Linux Php
finished: false
---

## 下载与安装

下载并安装Composer：

`$ curl -sS https://getcomposer.org/installer | php`

查看Composer版本信息：

`$ /usr/bin/php composer.phar --version`

设置全局命令：

`$ sudo mv composer.phar /usr/local/bin/composer`

查看是否安装与设置成功：

`$ composer -version`