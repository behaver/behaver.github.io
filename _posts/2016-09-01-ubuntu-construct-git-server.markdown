---
layout: post
title: "在ubuntu上布置git协作服务器"
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

以上设定的仓库位置也可以自定义路径存放，先选定一个目录作为Git仓库，假定是/srv/sample.git， 在/srv目录下输入命令：

`$ sudo git init --bare sample.git`  
`$ sudo chown -R git:git sample.git --把owner改为git`

接下来初始化全局设置：

`$ git config --global user.name “git”`  
`$ git config --global user.email “git@本机IP或域名”`

## 四、客户端的配置

### 创建管理ssh密钥

由于在ssh传输协议中，是通过密钥进行传输认证的，所以我们需要为每一个参加传输的主机创建ssh密钥。

在默认用户的主目录路径下，运行以下命令，按照提示创建（可直接回车略过）  
`$ ssh-keygen -t rsa`

公钥和私钥默认会保存在~/.ssh目录下，如下所示：  
id_rsa(私钥)  
id_rsa.pub(公钥)  
known_hosts(已知传输主机列表)

将客户端的公钥文件通过ssh复制到服务器的临时目录下：

`$ ssh-copy-id git@git服务器IP`

或通过以下命令：

`ssh gitadmin@host 'mkdir -p .ssh && cat >> .ssh/authorized_keys' < ~/.ssh/id_rsa.pub`

### 测试git传输

上述配置完成之后，我们clone一个服务器上的仓库来测试配置是否成功：

`$ git clone git@git服务器IP:/home/git/repositories/sample.git`

## 五、安全处理

出于安全考虑，可以设置服务器端的git用户不允许登录shell，可通过：

`$ vim /etc/passwd`

并找到下面的一行，改为其下一行：

git:x:1001:1001:,,,:/home/git:/bin/bash  
git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell

这样，在服务器端，git用户可以正常通过ssh使用git服务，但无法登录shell。因为我们为git用户指定的git-shell每次一登录就自动退出。