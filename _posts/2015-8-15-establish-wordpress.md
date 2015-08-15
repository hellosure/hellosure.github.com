---
layout: post
title: 如何快速搭建一个WordPress网站
category: WordPress
tags: [WordPress,Ubuntu]
---

1. ssh连接虚拟主机

2. 安装Lamp套装的准备工作

    $ sudo apt-get install tasksel

报错locale: Cannot set LC_CTYPE to default locale: No such file or directory 

解决方法：
（1）第一步

    $ sudo vim /etc/default/locale

添加LC_ALL="en_US.UTF-8"
LANG="en_US.UTF-8"

（2）第二步

注释掉`/etc/ssh/ssh_config`中的`SendEnv LANG LC_*`和`/etc/ssh/sshd_config`中的`AcceptEnv LANG LC_*`，这样可以不让远程连接去影响LANG

3. 接着开始安装Lamp

    $ sudo apt-get update
    $ sudo tasksel install lamp-server 

4. mysql创建数据库

    $mysql -u root -p

    mysql> CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
    mysql> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER, CREATE TEMPORARY TABLES, LOCK TABLES ON wordpress.* TO 'hellosure'@'localhost' IDENTIFIED BY 'hellosure';

5. 安装wordpress

（1）下载wordpress

    $ wget http://wordpress.org/latest.tar.gz

（2）解压

    $ tar -xzvf latest.tar.gz

（3）放到apache路径下

    $ sudo mv /home/ubuntu/wordpress/wordpress/* /var/www/html/

（4）删除`/var/www/html/`中默认的`index.html`，这样才能访问到wordpress的`index.php`

    $ sudo rm -f index.html

6. 配置wordpress

（1）刷新页面，利用虚拟主机的host作为域名

    http://虚拟主机域名/wp-admin/setup-config.php

（2）用页面上的提示创建

    $ sudo vim wp-config.php

现在访问页面就是wordpress模板了。

7. 绑定到自己的域名，将DNS指向虚拟主机的public IP

-EOF-
