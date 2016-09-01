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

## 一、创建服务器git用户

在ubuntu上我们需要创建一个用于对外提供一切git服务的linux用户，等同于github的ssh地址中的`__git__`@github.com/behaver/xxx.git
中的@前的git，这就是github服务器上用来提供git服务的系统用户。

`$ sudo useradd -m git`

添加一个名为git的用户，-m表示为用户在/home下创建用户目录。

`$ sudo passwd git`

为git用户设置密码。

git用户至此创建完毕，我们还需要将其设为管理员账户，我们通过修改sudoers文件来实现。

`$ sudo vim /etc/sudoers`

`root ALL=(ALL:ALL)  ALL` 之后添加一行：

`git  ALL=(ALL:ALL)  ALL`

文件保存后，通过su命令切换当前用户到git用户：

`$ su git`

## 二、安装配置SSH服务

由于git对文件在服务端与客户端间的传输中使用到了ssh协议，在服务端通过人工采集，来添加保存可信客户端的用户公钥，以建立安全可信的通讯传输。通过ssh方式传输可以使客户端用户不必输入服务器的用户名（这里是git用户）和密码，因为服务器保存有客户端公钥。这样也就省去了用户每次git push的时候都需要输入用户名、密码的麻烦，而且可以防止服务器用户密码的泄露。

安装ssh服务命令：

`$ sudo apt-get install openssh-server openssh-client`           - OpenSSH为自由软件，是ssh的开源实现。

修改一下ssh的原有配置文件：

`$ sudo vim /etc/ssh/sshd_config`

找到下面几行，去掉前面"#"注释，并设置：

StrictModes  no     - 在用户名和其公钥文件名不匹配时将通过验证。  
RSAAuthentication yes   - 使用纯的RSA认证。  
PubkeyAuthentication yes    - 允许Public Key。  
AuthorizedKeysFile     %h/.ssh/authorized_keys  - 设置客户端公钥的存储位置。

这里同时需要我们把每个git使用者的公钥（id_rsa.pub）收集起来放到/home/git/.ssh/authorized_keys文件夹里。

## 三、安装配置git服务

安装git服务器：

`$ sudo apt-get install git git-core`

`$ sudo mkdir /home/git/repositories`      --创建git仓库存储目录  
`$ sudo chown git:git /home/git/repositories`     --设定所有者  
`$ sudo chmod 755 /home/git/repositories`     --设置仓库访问权限  

以上设定的仓库位置也可以自定义路径存放

接下来初始化全局设置：

$ git config --global user.name “git”  
$ git config --global user.email “git@本机IP或域名”

## 四、客户端的配置

