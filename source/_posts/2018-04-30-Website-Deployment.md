---
author: jecyu
layout: post
title: 网站线上部署
date: 2018-04-30
comments: true
categories: 后端相关
tags: [Linux，Node.js，Mongodb，Nginx]
---

本文要实现一个前后端分离的网站项目的线上部署，[网站预览地址](ebio.linjiyu.com)，[源码地址](https://github.com/Jecyu/E-Biotech-website)，在部署的过程踩了不少坑，现在总结实现步骤如下，文章涉及到 local$ 和 remote$ 等标识符为区分本地环境和远程服务器所用，后端配置 Node.js + Mongodb + Nginx。 

## 一、 连接远程服务器, 配置 Node.js 环境

### 连接服务器

在这里花森煜米的本地系统是 win10。 配合使用命令行工具 [cmder](http://cmder.net/)，给予你在 window 下的 Linux 环境。然后通过 ssh 进行连接远程服务器 CentOS 7（这里是你自己的服务器。) SSH Client 的基本用法如下：

    ssh user@remote -p port // 执行该行命令后，会提示输入你的用户密码
    user@remote's password: 

成功连接服务器后，这里是 linux 目录结构。搞清楚每个目录的作用，把你自定义的文件放到最合适的地方。

|  目录    |  描述      |  
|----------|:-------------|
|  /       |  第一层次结构 的根、 整个文件系统层次结构的根目录。                  |
|  /bin    |  计算机的可执行二进制文件，如 cat，ls，cp。                       | 
|  /boot   |  引导程序文件，例如： kernel、initrd。                            |
|  /dev    |  必要设备，如 /dev/null。                                       |
|  /etc    |  特定主机，系统范围内的配置文件。                                 |
|  /home   |  当前用户的主目录，包含保存的文件，个人设置等。                     |
|  /lib    |  /bin 和 /sbin 中的二进制文件依赖的库文件。                       |
|  /media  |  可移除媒体的挂载点（如 CD-ROMS）。                               |
|  /mnt    |  临时挂载的文件系统。                                            |
|  /opt    |  可选的应用程序软件包。                                          |
|  /proc   |  虚拟文件系统，将内核与进程状态归档为文本文件。                     |
|  /root   |  超级用户的主目录。                                             |
|  /run    |  自最后一个启动以来运行中的系统的信息。例如：当前登录的用户和守护进程。|
|  /sbin   |  必要的系统二进制文件，如 fsck，init，route。     |
|  /srv    |  站点的具体数据，由系统提供。例如 web 服务器的数据和脚本。 |
|  /sys    |  包含有关设备、驱动程序和一些内核功能的信息。 |
|  /tmp    |  临时文件，通常在系统重启时不会被保留。      |
|  /usr    |  用于存储只读的用户数据第二层次结构，包含绝大多数的（多）用户工具和应用。 |
|  /var    |  变量文件——在正常运行的系统中其内容不断变化的文件，如日志，脱机文件和临时电子邮件文件。|


### 配置 Node.js

使用 [Node.js](https://nodejs.org/en/) 不仅仅是实现后端应用，还实现了整个 HTTP 服务器，服务器可以监听客户端的请求，类似 Apache、Nginx等 HTTP 服务器。首先把 Node.js 安装到 /usr/local 目录下，然后分别为 `node` 和 `npm` 命令建立软连接，以便在任何地方都可以调用它们。

    # 这里安装的是当前 node.js 的稳定版本
    wget https://npm.taobao.org/mirrors/node/v8.11.1-linux-x64.tar.xz
    # 两次解压
    xz -d node-v8.11.1-linux-x64.xz
    tar xzvf node-v8.11.1-linxu-x64.tar
    # 建立软连接
    ln -s /usr/local/node-v8.11.1-linxu-x64/bin/node /usr/local/bin/node
    ln -s /usr/local/node-v8.11.1-linxu-x64/bin/npm  /usr/local/bin/npm

## 二、 配置 Mongodb 数据库环境

安装 Mongodb 与 Node.js 步骤差不多，需要额外设置的是新建一个 mongodb 文件夹，用于存在数据库的数据 data、 配置 etc 和 logs 日志文件以及 Mongodb 程序文件。
    
### 安装 Mongodb 数据库

    # 安装 Mongodb 当前的稳定版本
    curl http://downloads.mongodb.org/linux/mongodb-linux-x86_64-3.6.4.tgz > mongo.tgz
    # 进行解压
    tar xzvf mongo.tgz
    # 当前 mongodb 下的文件
    data etc logs mongodb-linxu-x86_64-3.6.4

现在在 etc 文件下，新建配置文件 mongo.conf，配置如下：

    dbpath=/mongodb/data
	logpath=/mongodb/logs/mongo.log
	logappend=true
	journal=true
	quiet=true
	port=27017
	
### 保持数据库一直运行

保持程序在后台运行，可以通过系统服务 `systemctl start mongod -f /mongodb/etc/mongo.conf`  来实现，也可以通过把程序保存为守护(daemon)进程 `mongod --fork /mongodb/etc/mongo.conf`。数据库开启后，通过初始化设置后（可以看[这里](https://www.howtoforge.com/tutorial/how-to-install-and-configure-mongodb-on-centos-7/#step-add-the-mongodb-repository-in-centos)），可以在本地使用客户端如 [NoSQLBooster for MongoDB](https://nosqlbooster.com/)  来连接远程服务器的数据库,注意的是如果要使用客户端来连接，需要在 /etc/mongo.conf 更改 `bind_ip = 0.0.0.0`, 默认是 `	bind_ip = 127.0.0.1` 只允许在服务器主机进行数据库连接。

## 三、 上传网站项目文件至服务器

一般上传文件到服务器，可以通过 scp 协议上传，也可以通过支持 SFTP 协议的客户端图形化工具如 [FileZilla](https://filezilla-project.org/) 上传。

### 通过 scp 协议上传 

    # 把本地的文件上传到服务器，如果上传的是文件夹，则在路径前加 `-r`。
    scp -P port (-r) /path/to/local/file user@remote$_address:/path/to/remote/file

### 通过 SFTP 协议上传

使用 SFTP 可以通过用户名和密码连接服务器，也可以使用公钥无密码连接。
![FileZlip](http://p1s28g7i1.bkt.clouddn.com/fz3_win_main.png)

对于前后端分离的项目，前端项目文件建议放在 `/var/www/myapp.com` 下，而后端项目文件花森煜米放置在根目录 `/workspace/myapp-backend` 下，然后可以使用 [PM2](https://www.npmjs.com/package/pm2) 工具后台运行 Node.js 服务器。以上均是手动部署，一个项目会经过不断的版本迭代更新，为避免人工部署的出错，我们一般编写自动化部署脚本，从 github 上 pull 最新的项目文件，完成自动化部署，可参见 [https://github.com/visionmedia/deploy](https://github.com/visionmedia/deploy)。 

## 四、 使用 Nginx 配置域名和代理请求

使用 [Nginx](http://nginx.org/) 可以通过二级域名或二级目录（通过 node server 子目录来访问前端项目）来访问放置在 /var/www/myapp.com 的前端项目文件，本项目采用的是二级域名方法来访问（需要注意的是，二级域名要提前在服务器进行域名解析，否则访问不到。），因此会涉及到前后端交互跨域的问题，通过设置 Nginx 来代理转发请求。

![Reverse Proxy to Node.js Application](http://p1s28g7i1.bkt.clouddn.com/node_diagram.png)

### 安装 Nginx 服务器

首先添加 Nginx 存储库(资料库)，关于包管理器可以看[这篇文章](https://www.digitalocean.com/community/tutorials/package-management-basics-apt-yum-dnf-pkg)。创建一个名为 /etc/yum.repos.d/nginx.repo，然后粘贴下面的配置

    [nginx]
    name=nginx repo
    baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
    gpgcheck=0
    enabled=1

然后通过以下命令进行安装

    yum install nginx

命令语法： nginx -s signal

- stop--- 快速关机
- quit--- 优雅的关机
- reload--- 重新加载配置文件
- reopen--- 重新打开日志 log 文件

### 域名配置和转发请求

在 /etc/nginx 中，nginx.conf 为主配置文件。另外我们需要新建两个文件夹放置自定义的设置

- sites-available 用于保存所有的服务器配置文件。
- sites-enable 保存我们想要发布的服务器配置文件的软链接。

首先在打开 nginx.conf，在结尾处添加以下代码，确保在 sites-enable 中查找 server block {}：

    include /etc/nginx/sites-enabled/*.conf;
	server_names_hash_bucket_size 64;

在 sites-available 中，创建 myapp.com.conf，设置以下：

	server {
	    listen       80;
        # 二级域名配置
	    server_name  *.linjiyu.com;  
        # 访问的文件根目录
	    root   /var/www/linjiyu.com/ebio;
	    index index.html index.htm;
	
	    #charset koi8-r;
	    #access_log  /var/log/nginx/host.access.log  main;
	
	    location / {
	      # pass-on visitor real-IP to backend
	      proxy_set_header X-Real-IP $remote_addr;
	      # proxy_pass  http://127.0.0.1:8080/;
	    }
	    
        # 转发请求
	    location /users/ {
	      proxy_set_header Host    $host;
	      proxy_pass    http://127.0.0.1:3000/users/;
	    }
    }
    # ... 这里可以继续配置第二个域名
    # server {

    # }

## 五、 参考资料

- [Initial Server Setup with CentOS 7](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-centos-7)
- [Secure Shell](https://en.wikipedia.org/wiki/Secure_Shell)
- [SSH 基本用法](https://zhuanlan.zhihu.com/p/21999778)
- [Filesystem Hierarchy Standard](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)
- [How to Install and Configure MongoDB on CentOS 7](https://www.howtoforge.com/tutorial/how-to-install-and-configure-mongodb-on-centos-7/#step-add-the-mongodb-repository-in-centos)
- [How To Set Up Nginx Server Blocks on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-server-blocks-on-centos-7)

（完）
