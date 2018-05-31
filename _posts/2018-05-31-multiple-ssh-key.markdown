---
layout: post
title: "ssh多密钥的使用方法"
date: 2016-09-14
author: 董三碗
category: 服务器
tags: ssh Linux 服务器
finished: true
---


在使用 git 管理项目的时候，其远端仓库地址分为 https 和 ssh，使用 https 地址作为 git 远端仓库地址时，无需额外的设置，但在你使用 push 操作的时候，需要验证用户名和密码，而使用 ssh 地址设置为 git 远端仓库地址，你必须要先添加好 ssh 密钥，如果你在生成 ssh 密钥时，passphrase 为空，则 push 时不需要输入密码，否则输入你设置的 passphrase。因为不同远端仓库 host 对应的 ssh 不能相同，所以我们需要为不同的远端 host 生成一个唯一对应的 ssh 密钥。

## 生成密钥

1、生成 ssh 密钥：

`ssh-keygen -t rsa`

-t 指定密钥类型，默认是 rsa ，可省略。
-C 设置注释文字。
-f 指定密钥文件存储文件名。

2、在其询问密钥的保存路径时，更改为对应的文件，这里示例文为 id_rsa_github ：

`Enter file in which to save the key (/root/.ssh/id_rsa):/root/.ssh/id_rsa_github`

3、询问是否为 ssh 设置密码时，若设置，在 push 操作时需要输入该密码。这里可为空，则 push 时无需输入密码：

`Enter passphrase (empty for no passphrase)`

4、按照上面的步骤，我们可以继续生成其他 ssh 密钥，例如 id_rsa_oschina。

5、在 /root/.ssh 执行 ls 命令，可以看到刚刚生成的公钥与私钥

## 设置 ssh 密钥代理

ssh-agent 是管理多个 ssh key 的代理，用户使用 `ssh-add` 命令将私钥交给 ssh-agent 保管，其他程序需要身份验证的时候，ssh-agent会接受其验证申请，并完成整个认证的过程。

1、查看系统 ssh 密钥代理，执行：

`ssh-add -l`

如果提示 `Could not open a connection to your authentication agent.`

执行 `exec ssh-agent bash`

2、添加你的私钥至 ssh-agent：

`ssh-add ~/.ssh/id_rsa_github`

如上操作添加你的全部私钥。

3.1、打开 github 或者 oschina 的 ssh 管理页面，将对应的公钥添加到 ssh 公钥列表中。

3.2、如果是私人服务器，则通过命令`ssh-copy-id -i ~/.ssh/id_rsa_xxx username@服务器IP`，将本地主机的公钥复制到远程主机的 authorized_keys 文件中。

## ssh 配置文件

1、打开 .ssh 目录下的 config 文件，若没有改文件，则新建一个

`vi ~/.ssh/config`

2、在文件中输入如下内容

`Host github
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_github

Host xxx
    HostName 服务器ip
    User git
    IdentityFile ~/.ssh/id_rsa_xxx

...`

到此就完成了多个 ssh 密钥同时使用的配置

