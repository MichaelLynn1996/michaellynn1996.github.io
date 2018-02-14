---
title: 记 在Ubuntu下安装Nginx
date: 2018-02-13 11:43:41
tags: 
    - Nginx
    - Ubuntu
---

前段时间我已经将 WordPress 从服务器上卸载，并且完全依靠 Hexo + GithubPage 来部署我的博客，所以我的服务器里是什么都没了。以前我在服务器里安装 Apache，但是后来了解到 Nginx有 ***轻量，抗并发*** 的特点，似乎更适合虚拟主机（毕竟我的服务器是腾讯云最便宜那款）。

荒废已久的服务器我已经不记得我做过什么，所以干脆就重装一遍好了。

<!-- more -->

## 开启Root账户

腾讯云新安装的Ubuntu Server默认登陆的不是Root账户（其他Linux发行版好像不会，比如说CentOS），如果有时遇到需要权限的操作就需要Root账户，不然会出错。

### 1.手动开启Root账户的方法：

```
$ sudo passwd root
```

### 2.按照输入两次Root账户的密码,然后修改ssh设置：

```
$ sudo vi /etc/ssh/sshd_config
```

在「命令行模式（command mode）」下按一下字母「i」就可以进入「插入模式（Insert mode）」，这时候你就可以开始输入文字了。   
找到  PermitRootLogin 这项，将其改为 yes  
输入完成后按ESC，再输入 :wq 退出vi的插入模式（Insert mode）。  
  
### 3.最后重启ssh服务既可。

```
$ sudo service ssh  restart
```

### 4.Something else：

我再这里重启ssh的时候遇到了这个报错：

    ubuntu@VM-110-44-ubuntu:~$ sudo service ssh  restart
    Job for ssh.service failed because the control process exited with error code. See "systemctl status ssh.service" and "journalctl -xe" for details.

然后我去百度了一下，run下面这个命令可以知道ssh服务出错的原因：

```
$ sudo service ssh  restart
```

然后发现原来是我把yes打成了yse...（辣鸡）

    ubuntu@VM-110-44-ubuntu:~$ sudo /usr/sbin/sshd -T
    /etc/ssh/sshd_config line 28: unsupported option "yse".

## 安装Nginx

### 1.重新以root账户登陆服务器。

### 2.更新软件源

输入以下两个命令更新软件源:

```
$ sudo apt-get update
$ sudo apt-get dist-upgrade
```

在出现“你希望继续执行嘛？[Y/N]”,请输入“Y”，进入自动更新Ubuntu系统及你安装的软件。直到完成为止

### 3.基于APT源安装

```
$ sudo apt-get install nginx
```

### 4.防火墙设置

#### 查看列表 输入:

```
$ sudo ufw app list
```

    root@VM-110-44-ubuntu:~# sudo ufw app list
    Available applications:
      Nginx Full
      Nginx HTTP
      Nginx HTTPS
      OpenSSH

* Nginx Full：此配置文件打开端口80（正常，未加密的Web流量）和端口443（TLS / SSL加密流量）
* Nginx HTTP：此配置文件仅打开端口80（正常，未加密的Web流量）
* Nginx HTTPS：此配置文件仅打开端口443（TLS / SSL加密流量）

#### 键入以下命令来启用此功能：

```
$ sudo ufw allow 'Nginx HTTP'
```

验证：

```
$ sudo ufw status
```

    root@VM-110-44-ubuntu:~# sudo ufw status
    Status: inactive

#### 启动 Nginx

```
$ systemctl status nginx
```

最后在浏览器的地址输入服务器的域名或者网址出现以下页面就说明启动成功了

 ![](install-nginx-in-ubuntu/1.png)

至于 Nginx 的配置可以看[
Nginx中文文档](http://www.nginx.cn/doc/index.html)

Nginx的安装就到此结束啦。