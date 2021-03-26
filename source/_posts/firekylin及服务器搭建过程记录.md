---
title: firekylin及服务器搭建过程记录
date: 2016-06-16 14:28:15
tags: 
	- firekylin
	- blog搭建
categories: 环境搭建
---

断断续续两天的时间，由之前的wordpress切换到firekylin，记录一下配置过程。
大致过分以下几步：
- 安装CentOS系统
- 搭建Nginx（考虑到我的服务器上不只要跑一套网站，这里采用了虚拟主机面板Wdcp）
- 安装mysql数据库（WDCP面板已涵盖，不做阐述）
- 配置Ftp工具（WDCP面板已涵盖，不做阐述）
- 安装nodeJS
- 安装Pm2
- 安装firekylin
- 配置PM2
- 配置nginx

<!--more-->
## 安装CentOS
由于服务器是阿里云，可在阿里云控制台傻瓜式切换操作系统，我这里选择了公共镜像CentOS X64

注：其他的主机供应商安装系统可自行百度

## 搭建nginx
单独搭建nginx请自行百度，我这里使用的wdcp,安装命令如下：

```
wget http://dl.wdlinux.cn:5180/lanmp_laster.tar.gz
tar zxvf lanmp_laster.tar.gz
sh install.sh
```
安装完wdcp主机面板，可通过浏览器访问：ip:8080
默认用户名：admin
密码：wdlinux.cn

进入到后台，在mysql管理中打开PHPmyadmin工具，新建数据库，后边会使用到。

## 安装nodeJS

安装nodeJS的方法有多种，更多方法自行百度，我使用的yum安装方法
直接在服务器命令行终端敲入一下命令：

```
yum install nodejs
```
稍等片刻，即可自行安装完毕

```
node -v
```
查看下node版本，版本如果过低，需要升级nodeJS版本，升级nodeJS可使用npm安装n模块，如果nodeJS安装完没有npm,需要先安装npm包管理工具。安装n模块，敲入一下命令：

```
npm install -g n
```
升级nodejs

```
n stable
```
n后面也可以跟随版本号比如：

```
n v0.10.26
```
或

```
n 0.10.26
```
至此，nodeJS安装完毕

## 安装pm2
在服务器上推荐使用 pm2`来管理 Node.js 服务，可以通过以下命令来安装 pm2。

```
sudo npm install -g pm2
```
至此，所有需要的工具软件安装完毕，接下来是配置过程。

## 安装firekylin

- 安装方式一

下载到本地，下载链接传送门：[firekylin](http://firekylin.org/release/firekylin_0.12.4.tar.gz)，下载完毕后，通过FTP工具上传至服务器。

- 安装方式二

通过远程服务器链接终端，直接下载并解压缩，命令如下：

```
wget http://firekylin.org/release/firekylin_0.12.4.tar.gz
tar xvf firekylin_0.12.4.tar.gz
```
`ls -l `查看当前目录，会看到解压缩之后的`firekylin`，`cd firekylin_0.12.4/firekylin`，切换至压缩之后的源码目录，在这个目录下会看到pakeage.json文件，通过npm安装依赖包，在这个目录下敲入一下命令：

```
npm install
```
等待，等待~~~~
安装完毕后，启动服务，输入命令：

```
npm start
```
服务启动之后，然后访问127.0.0.1:8360（服务器是IP，本地是本地的IP），根据提示填写相关信息进行安装。
注：数据库一栏可填入我们之前的通过phpMyadmin创建的数据库名称。

此时，firekylin便搭建并运行起来了，可通过：IP：8360进行访问。

## 配置PM2

在服务器上推荐使用 pm2 来管理 Node.js 服务，将项目下的 pm2_default.json 文件改为 pm2.json，将文件中的 cwd 配置值改为项目的当前路径。

然后通过以下命令来启动项目。

```
pm2 start pm2.json
```

## 配置nginx

将项目下的 nginx_default.conf 改为 nginx.conf，修改文件中的 `server_name`、`root` 和 `set $node_port` 等配置值，然后将该文件软链到 nginx的配置目录下。

假设nginx 的配置目录为` /usr/local/nginx/conf/include`，那么可以通过下面的命令设置软链：

```
sudo ln -s path/to/nginx.conf /usr/local/nginx/conf/include/www.mnzone.com.conf
```
需要将` path.to` 改为当前的项目路径，` www.mnzone.com` 改为对应的域名。

需要注明的是：wdcp的nginx配置目录，在vhost目录下。



