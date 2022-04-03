---
title: "告别百度网盘，安装自己的专属网盘——Cloudreve，不限制下载速度！"
date: 2022-04-03T20:51:58+08:00
draft: false
ShowCodeCopyButtons: true
categories:
    - Linux
tags:
    - Cloudreve
series:
    - Cloudreve
---

> Cloudreve 是一个用[Go](https://so.csdn.net/so/search?q=Go%E8%AF%AD%E8%A8%80&spm=1001.2101.3001.7020)语言写的公有网盘程序，我们可以用它来快速搭建起自己的网盘服务，公有云/私有云都可。

![A1-1](/images/Cloudreve_install/A1-1.webp)

## ✨ 特性

- ☁️ 支持本机、从机、七牛、阿里云 OSS、腾讯云 COS、又拍云、OneDrive (包括世纪互联版) 作为存储端
- 📤 上传/下载 支持客户端直传，支持下载限速
- 💾 可对接 Aria2 离线下载
- 📚 在线 压缩/解压缩、多文件打包下载
- 💻 覆盖全部存储策略的 WebDAV 协议支持
- ⚡️ 拖拽上传、目录上传、流式上传处理
- 🗃 文件拖拽管理
- 👩‍👧‍👦 多用户、用户组
- 🔗 创建文件、目录的分享链接，可设定自动过期
- 👁‍🗨 视频、图像、音频、文本、Office 文档在线预览
- 🎨 自定义配色、黑暗模式、PWA 应用、全站单页应用
- 🚀 All-In-One 打包，开箱即用
- 🌈 … …

## 安装前的准备

安装之前我们需要准备好环境：

- 一台服务器（VPS）
- 安装宝塔面板
- 安装nginx
- 安装mysql
- 准备一个域名

## 🛠 部署

### Go语言环境

- 安装环境：CentOS Linux release 7.8.2003 (Core)

- 宝塔面板7.4.5

- golang：go1.15.1.linux-amd64.tar.gz

## Linux 常用命令

### 查看系统版本

```bash
ll /etc/*centos*
cat /etc/centos-release
```

### 解压命令

`tar –xvf file.tar` //解压 tar包

`tar -xzvf file.tar.gz` //解压tar.gz

`tar -xjvf file.tar.bz2` //解压 tar.bz2tar –xZvf file.tar.Z //解压tar.Z

## Go简介

下载之前先去官网溜达下，点击【download go】就可进入下载页面：

![A1-2](/images/Cloudreve_install/A1-2.webp)

![A1-3](/images/Cloudreve_install/A1-3.webp)

官网：[https://golang.google.cn/](https://golang.google.cn/)

根据自己的系统环境下载相应的版本，这里选择的是go1.15.1.linux-amd64.tar.gz。

## 下载安装

宝塔面板可以直接在面板里面下载安装，这里为了方便，直接就在命令环境下面下载安装配置了。

### 下载

SSH工具连接服务器开始操作：

```bash

cd /www/server && wget -O golang.tar.gz https://dl.google.com/go/go1.15.1.linux-amd64.tar.gz
```

这些可以直接在面板环境里操作，也很方便。

### 解压

下载好之后解压：

```bash
tar -xzvf golang.tar.gz
```

### 添加环境变量

添加环境变量，使用vim 打开`/etc/profile` 文件。

```bash
vim /etc/profile
```

在profile 最底部添加：

```bash
export GOROOT=/www/server/go
export GOBIN=$GOROOT/bin
export GOPKG=$GOROOT/pkg/tool/linux_amd64
export GOARCH=amd64
export GOOS=linux
export GOPATH=/www/wwwroot/Golang
export PATH=$PATH:$GOBIN:$GOPKG:$GOPATH/bin
```

丢一张图：

![A1-4](/images/Cloudreve_install/A1-4.webp)

添加好之后，保存退出，然后执行如下命令使其生效：

```bash
source /etc/profile
```

### 测试是否生效

使用如下命令来测试Go语言环境是否安装成功。

```go
go version
```

![A1-5](/images/Cloudreve_install/A1-5.webp)

如果出现上图，表示我们已经安装成功了。

### 创建GOROOT目录

使用命令来创建，不过也可以用面板环境来进行可视化操作：

```bash
mkdir /www/wwwroot/Golang
```

通过上面的步骤，我们就完全安装好Go了。

### Go run

我们可以用一段代码来验证一下go语言的运行。

我们到Golang里面新建一个文件命名为test.go

```bash
touch test.go
```

之后用vim简单编辑下，也可以直接到宝塔面板里新建编辑：

```bash
vim test.go
```

比如复制这段代码进去：

```go
package main 
import "fmt" 
func main() {   
    /* 这是我的第一个简单的程序 */   
    fmt.Println("Hello, Roy's Blog!")
}
```

![A1-6](/images/Cloudreve_install/A1-6.webp)

代码保存好之后，我们要开始执行 Go 程序了。

如何执行呢？

打开命令行，并进入程序文件保存的目录中。

```go
go run test.go
```

出现下图：

成功执行了这一段代码，输出了“Hello, Roy’s Blog!”

![A1-7](/images/Cloudreve_install/A1-7.webp)

### 总结

整个环境的安装和简单的测试运行代码就说完了，希望对想学习Go语言的同学能有一点帮助。

Go 是一个开源的编程语言，它能让构造简单、可靠且高效的软件变得容易。

Go是从2007年末由Robert Griesemer, Rob Pike, Ken Thompson主持开发，后来还加入了Ian Lance Taylor, Russ Cox等人，并最终于2009年11月开源，在2012年早些时候发布了Go 1稳定版本。现在Go的开发已经是完全开放的，并且拥有一个活跃的社区。

### Go 语言特色

- 简洁、快速、安全
- 并行、有趣、开源
- 内存管理、数组安全、编译迅速

### Go 语言用途

1. Go 语言被设计成一门应用于搭载 Web 服务器，存储集群或类似用途的巨型中央服务器的系统编程语言。

2. 对于高性能分布式系统领域而言，Go 语言无疑比大多数其它语言有着更高的开发效率。它提供了海量并行的支持，这对于游戏服务端的开发而言是再好不过了。

## 安装Cloudreve

```bash
cd /opt
wget https://github.com/cloudreve/Cloudreve/releases/download/3.1.1/cloudreve_3.1.1_linux_amd64.tar.gz
tar -zxvf cloudreve_3.1.1_linux_amd64.tar.gz   #解压获取到的主程序
chmod +x ./cloudreve  #赋予执行权限
./cloudreve   #启动 Cloudreve
```

分别复制命令回车执行，安装成功截图如下：

![A1-8](/images/Cloudreve_install/A1-8.webp)

Cloudreve 在首次启动时，会创建初始管理员账号，请注意保管管理员密码，此密码只会在首次启动时出现。(比如我这里的’admin@cloudreve.org’,‘rgFzEBLc’)

如果您忘记初始管理员密码，需要删除同级目录下的“cloudreve.db”，重新启动主程序以初始化新的管理员账户。

Cloudreve 默认会监听“5212”端口。你可以在浏览器中访问’http://服务器IP:5212’进入 Cloudreve。如果宝塔面板需要在安全中放行“5212”端口。注意用默认的管理账号和密码登录。

## 放行端口

### 服务器端

![A1-9](/images/Cloudreve_install/A1-9.webp)

![B1-1](/images/Cloudreve_install/B1-1.webp)

### 宝塔端

![B1-2](/images/Cloudreve_install/B1-2.webp)

## 进程守护

以上步骤操作完后，最简单的部署就完成了。

我们需要一些更为具体的配置，才能让Cloudreve更好的工作，宝塔面板我们可以使用Supervisor管理器来设置进程守护，具体流程请参考下面的配置流程。

## 安装Supervisor管理器

在宝塔面板里头，软件商店→系统工具 ，找到Supervisor管理器安装即可。

![B1-3](/images/Cloudreve_install/B1-3.webp)

## 添加守护进程

打开Supervisor管理器添加守护进程：

![B1-4](/images/Cloudreve_install/B1-4.webp)

![B1-5](/images/Cloudreve_install/B1-5.webp)

`注意：路径修改为自己的。添加完成后，守护进程就会启动成功。`

![B1-6](/images/Cloudreve_install/B1-6.webp)

![B1-7](/images/Cloudreve_install/B1-7.webp)

## 域名访问

新建网站，之后在网站设置中，配置反向代理。

## 添加域名解析

先在域名注册商那边把域名解析到服务器的IP上，我这边是腾讯的：

![B1-8](/images/Cloudreve_install/B1-8.webp)

关于域名解析方面的知识，可以看这篇文章[【服务器、域名购买】Namesilo优惠码和域名解析教程（附带服务器购买推荐和注意事项）](https://yirenliu.cn/archives/namesilo)

## 添加网站

![B1-9](/images/Cloudreve_install/B1-9.webp)

## 设置HTTPS

给网站加上SSL，https访问（小绿锁）：

![C1-1](/images/Cloudreve_install/C1-1.webp)

![C1-2](/images/Cloudreve_install/C1-2.webp)

## 配置Nginx反向代理

宝塔自带可以反向代理，但是不好用，而且貌似只能设置一个：

![C1-3](/images/Cloudreve_install/C1-3.webp)

这里我建议直接在网站的Nginx配置文件里修改：

![C1-4](/images/Cloudreve_install/C1-4.webp)

选中这部分，把他们变成注释`（前面加#号）`：

![C1-5](/images/Cloudreve_install/C1-5.webp)

然后在这之后加上下面的代码：

```nginx

        location / {
    proxy_pass http://127.0.0.1:5212/;
    rewrite ^/(.*)$ /$1 break;
    proxy_redirect off;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Upgrade-Insecure-Requests 1;
    proxy_set_header X-Forwarded-Proto https;
  }
```

![C1-6](/images/Cloudreve_install/C1-6.webp)

## 大功告成！

现在就可以用域名打开Cloudreve 访问了。

![C1-7](/images/Cloudreve_install/C1-7.webp)

![C1-8](/images/Cloudreve_install/C1-8.webp)

> 支持的存储策略：

![C1-9](/images/Cloudreve_install/C1-9.webp)

> 添加oneindex存储策略时详细的引导：

![D1-1](/images/Cloudreve_install/D1-1.webp)

> 离线下载：

![D1-2](/images/Cloudreve_install/D1-2.webp)

还有其他很多功能，大家自行尝试摸索~

## 进阶

首次启动时，Cloudreve 会在同级目录下创建名为`“conf.ini”`的配置文件，我们可以修改此文件进行一些参数的配置，保存后需要重新启动 Cloudreve 生效。

![D1-3](/images/Cloudreve_install/D1-3.webp)

![D1-4](/images/Cloudreve_install/D1-4.webp)

默认情况下，Cloudreve 会使用内置的 SQLite 数据库，并在同级目录创建数据库文件“cloudreve.db”，如果我们想要使用 MySQL，可以在配置文件中加入以下内容，并重启 Cloudreve。（非必须）

![D1-5](/images/Cloudreve_install/D1-5.webp)

```conf
[Database]
#数据库类型，目前支持 sqlite | mysql
Type = mysql
#用户名，以你数据库的为准
User = cloudreve
#密码，以你数据库的为准
Password = root
#数据库地址
Host = 127.0.0.1
#数据库名称，以你数据库的为准
Name = v3
#数据表前缀
TablePrefix = cd
```

![D1-6](/images/Cloudreve_install/D1-6.webp)

注意：更换数据库配置后，Cloudreve 会重新初始化数据库，原有的数据将会丢失。

![D1-7](/images/Cloudreve_install/D1-7.webp)

记下新的账户和密码。

注意这边因为我们已经启动了一次cloudreve，所以重启的时候会提示端口被占用。

![D1-8](/images/Cloudreve_install/D1-8.webp)

我们可以重启一下服务器，就能用新的账户密码登录了。

![D1-9](/images/Cloudreve_install/D1-9.webp)

登录后，记得改一下默认的账户和密码：

![E1-1](/images/Cloudreve_install/E1-1.webp)

![E1-2](/images/Cloudreve_install/E1-2.webp)

![E1-3](/images/Cloudreve_install/E1-3.webp)

默认用户只有1GB的空间，我们可以根据自己的vps大小更改空间大小。

![E1-4](/images/Cloudreve_install/E1-4.webp)

其实小文件（100M以内）完全可以用[蓝奏云](http://www.lanzou.com/account.php?action=register&USBeNVZgBTQFNgBhDmdXNVM2ADtZOQ==)来储存，速度快。

> 点击注册：[蓝奏云](http://www.lanzou.com/account.php?action=register&USBeNVZgBTQFNgBhDmdXNVM2ADtZOQ==)

我们这边的网盘一般可能会放一些大一点的文件，现在如果传会发现传不上去，原因是Nginx默认最大上传文件是50M，我们需要修改一下Nginx默认的上传文件大小：

![E1-5](/images/Cloudreve_install/E1-5.webp)

可以改成5120（5G）大小，然后记得重载一下配置。

如果是国内的服务器，备案完成之后，可以去白嫖七牛云和又拍云的每月10个G的空间，一般就挺够用了。我之前腾讯云新客户买的服务器88月一年，备案了之后白嫖了他们的空间，还是挺香的。

![E1-6](/images/Cloudreve_install/E1-6.webp)

现在可以正常上传了，我这边上传速度在3M左右。

![E1-7](/images/Cloudreve_install/E1-7.webp)

可以正常播放视频：

![E1-8](/images/Cloudreve_install/E1-8.webp)

分享的链接可以带密码保护、设置过期时间、允许预览等等：

![E1-9](/images/Cloudreve_install/E1-9.webp)

如果分享的密码忘了，或者想要修改分享的信息，在后台-我的分享里头也可以查看分享链接的信息:

![F1-1](/images/Cloudreve_install/F1-1.webp)

被分享者打开链接看到的界面：

![F1-2](/images/Cloudreve_install/F1-2.webp)

我这边是移动的网络，下载速度大概在3.5M/s左右：

![F1-3](/images/Cloudreve_install/F1-3.webp)

## 最后

从使用体验来看，cloudreve效果很不错，功能强大，支持存储种类也多，唯一不足的地方竟然不支持Google Drive 。作者更是说目前不支持，未来也不会支持。

场景使用：可以使用 Cloudreve 搭建个人用网盘、文件分享系统，亦或是针对大小团体的公有云系统。

## 参考资料

[https://www.daniao.org/8544.html](https://www.daniao.org/8544.html)

[https://www.daniao.org/5094.html](https://www.daniao.org/5094.html)

[https://www.gitiu.com/reprinted_articles/](https://www.gitiu.com/reprinted_articles/cloudreve-v3-and-go-install/)