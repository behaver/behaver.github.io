---
layout: post
title: "Linux上Apache2.4+的站点配置"
date: 2016-09-19
author: Vincent, Dong
category: 服务器
tags: Apache Linux 服务器
finished: true
---

这里我们假设需要在apache下新建一个www.wizard.com的站点。

## Host文件配置

修改系统Host配置文件：

`$ sudo vim /etc/hosts`

在其中添加一行：  
127.0.0.1     www.wizard.com

重新启动网络服务：

`$ sudo service networking restart`

最后，在浏览器中输入 **www.wizard.com** 测试访问

## 新建Apache站点配置文件

进入到目录 /etc/apache2/sites-available/ 下

`$ cd /etc/apache2/sites-available`

查看现有的站点配置文件

`$ la`

依据已有站点配置的序号(此处假设之前已有000-003)，自增1，新建一个站点配置文件

`$ sudo vim 004-wizard.conf`

## Apache站点配置文件说明

{% highlight html %}
#domain 通常设为*
<VirtualHost domain:80> 
    # 主站点名称，用它也可以访问到服务器，可以定义多个，用空格隔开即可。
    ServerName wizard 
    # 如果服务器有任何问题将发信到这个邮箱， 这个邮箱会在服务器产生的某些页面中出现，例如，错误报告
    ServerAdmin qianxing@yeah.net 
    CustomLog   /var/log/apache2/wizard-access.log combined 
    DocumentRoot /var/Wizard/ 
    <Directory /var/Wizard/> 
        # 配置在目录使用哪些特性，常用的值和基本含义如下： 
        #    ExecCGI: 在该目录下允许执行CGI脚本。 
        #    FollowSymLinks: 在该目录下允许文件系统使用符号连接。 
        #    Indexes: 当用户访问该目录时，如果用户找不到DirectoryIndex指定的主页文件(例如index.html),则返回该目录下的文件列表给用户。 
        #    SymLinksIfOwnerMatch: 当使用符号连接时，只有当符号连接的文件拥有者与实际文件的拥有者相同时才可以访问。
        Options Indexes FollowSymLinks MultiViews 

        # 允许存在于.htaccess文件中的指令类型(.htaccess文件名是可以改变的，其文件名由AccessFileName指令决定)
        AllowOverride all 

        # 控制在访问时Allow和Deny两个访问规则哪个优先
        Order allow,deny 

        # 允许访问的主机列表
        Allow from all 

        # 拒绝访问的主机列表（可用域名或子网，例如：Deny from 192.168.0.0/16）
        # Deny from all

        # 站点默认访问文件设置
        DirectoryIndex server.php
    </Directory> 
</VirtualHost>
{% endhighlight %}

配置完成后保存该文件。

## 链接Apache站点配置文件

`$ sudo ln -s /etc/apache2/sites-available/004-wizard.conf /etc/apache2/sites-enabled/004-wizard.conf`

## 修改Apache主配置文件

`$ sudo vim apache2.conf`

在其中添加下面一段代码，以给网站存放路径访问权限：

{% highlight html %}
<Directory /var/Wizard/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>
{% endhighlight %}

## 重启Apache服务

`$ sudo service apache2 restart`

到此，即可测试访问 www.wizard.com