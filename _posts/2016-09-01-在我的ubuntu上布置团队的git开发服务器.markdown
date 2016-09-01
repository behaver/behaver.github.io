---
layout: post
title: "在我的ubuntu上布置团队的git开发服务器"
subtitle: "在项目文件正式上传前的git服务器. "
date: 2016-09-01
author: Vincent, Dong
category: 服务器
tags: git ubuntu 服务器
finished: true
---

## 创建提供git服务的专有账户

`sudo useradd -m git`
添加一个名为git的用户，-m表示为用户在/home下创建用户目录。
`sudo passwd git`
为git用户设置密码。
