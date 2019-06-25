---
layout: post
title: "ownCloud and Gitlab"
categories: System_Maintenance
tags: ownCloud Gitlab
--- 

* content
{:toc}

In this post, I mainly record the process of building ownCloud and Gitlab　server, with the help of Internet, there are so many good blogs.





## **Git**

```
sudo apt-get install git
sudo apt-get install openssh-server openssh-client //然后可以ssh-keygen避免每次输入密码
sudo useradd git　　　//useradd 和 adduser有区别,其实推荐adduser
sudo passwd git
sudo mkdir /home/git/repositories //创建git仓库存储目录和权限
sudo chown git:git /home/git/repositories
sudo chmod 755 /home/git/repositories
su git
cd  /home/git/repositories
git init --bare gitName.git
```

## **Gitlab**

GitLab是利用 Ruby on Rails 一个开源的版本管理系统，实现一个自托管的Git项目仓库，可通过Web界面进行访问公开的或者私人项目。它拥有与Github类似的功能，能够浏览源代码，管理缺陷和注释。安装参考链接:[Gitlab](https://about.gitlab.com/installation/), 其实安装过程还是比较简单的．

软件集成是软件开发过程中的一个环节，这个环节的工作一般会包括以下流程：合并代码---->安装依赖---->编译---->测试---->发布, 软件集成的工作细碎繁琐，以前是由人工完成的。但是现在鼓励持续集成(持续集成是一种软件开发实践，即团队开发成员经常集成他们的工作，通常每个成员每天至少集成一次，每次集成都通过自动化的构建)，　因此，持续集成系统应运而生．
GitLab-CI就是一套配合GitLab使用的持续集成系统, 一般地，GitLab里面的每一个工程都会定义一个属于这个工程的软件集成脚本，用来自动化地完成一些软件集成工作。当这个工程的仓库代码发生变动时，比如有人push了代码，GitLab就会将这个变动通知GitLab-CI。这时GitLab-CI会找出与这个工程相关联的Runner，并通知这些Runner把代码更新到本地并执行预定义好的执行脚本。所以，GitLab-Runner就是一个用来执行软件集成脚本的东西.

其实除了Github, Gitlab外，还有Bitbucket作为代码仓库管理．

## **ownCloud**

ownCloud是一款用来创建属于自己的私有云服务的工具，可以完全掌控数据，能在纯局域网内使用。支持文件预览、版本控制、链接分享，还可以加载第三方储存、API 支持等等。服务器端与客户端均全平台支持。首先要安装LAMP套装; 然后安装ownCoud,配置数据库．

参考链接: [csdnblog](https://blog.csdn.net/And_w/article/details/71238266)  [digitalocean](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-owncloud-on-ubuntu-16-04)

## **IP and DNS**

学校外的网络真的很复杂，公司IP的话只能去运营商申请固定IP，否则都是动态。可以用花生壳或路由器域名绑定动态IP，然后设置DMZ主机．宽带外网IP显示为10、100、172开头，没有公网IP,　原因是在当前IPV4下，公网IP作为不可再生资源已经捉襟见肘，而上网用户和上网设备越来越多，所以运营商已经无法给新用户提供充足的IP地址资源了，因此运营商只能通过NAT转发技术（网络地址转换）的方法，使部分用户都处于一个大局域网内，公用少量的外网出口，从而缓解IP压力，就好比小区一样，您和同小区的所有居民公用一个小区大门．解决办法当然是投诉了或者花更多的钱固定IP．

#### **Firewall**
```
sudo ufw app list　//registed a few profiles with ufw
sudo ufw status
sudo ufw allow 'Apache Full'
sudo ufw delete allow 'Apache'
```

#### **Port**

一个原因是防黑客，另一个原因是80端口被电信封了...

针对Apache Web服务器:
```
/etc/apache2/ports.conf //将 NameVirtualHost *:80和Listen 80修改为自己需要的端口
/etc/apache2/sites-available/default //将第一行的 VirtualHost :80 改为自己需要的端口VirtualHost :1000
/etc/init.d/apache2/httpd.conf 添加 servername localhost
sudo /etc/init.d/apache2 restart 
```

#### **SSL**

实现https就需要一个SSL证书，云服务提供商如腾讯云，七牛云，阿里云，但是大部分都很贵．本着给公司省钱的原则，找了很多办法自制证书，因为我的443端口也被封了，这个真的是搞哭我了．

后来找到了非80端口不支持Http验证域名签发let’s encrypt ssl证书的[方法](https://blog.nbhao.org/2633.html)．最后明确我们的问题是要做一个[自签名SSL证书](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-16-04)

```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt
sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
sudo nano /etc/apache2/conf-available/ssl-params.conf
sudo a2enmod ssl
sudo a2enmod headers
sudo a2ensite default-ssl
sudo a2enconf ssl-params
sudo apache2ctl configtest
```

遇到的一个问题(ERR_SSL_PROTOCOL_ERROR when accessing this server)，参考[链接](https://stackoverflow.com/questions/34304022/change-ssl-port-of-apache2-server-err-ssl-protocol-error):

I summarize all procedures to set up ssl and changing its port. In this example, I change ssl port to 18443 from 443.

```
sudo apt-get install apache2
sudo a2enmod ssl
sudo a2ensite default-ssl
sudo service apache2 restart
ss -lnt
```
Then, trying to change ssl port:
```
sudo vi /etc/apache2/ports.conf
<IfModule ssl_module>
        Listen 18443
</IfModule>
<IfModule mod_gnutls.c>
        Listen 18443
</IfModule>
```
In this setting, I use default-ssl, so I also have to modify this file.
```
sudo vi /etc/apache2/sites-available/default-ssl.conf
 <IfModule mod_ssl.c>
   <VirtualHost _default_:18443>
```