---
title: "挂载OSS"
date: 2022-04-03T15:16:17+08:00
draft: false
ShowCodeCopyButtons: true
categories:
    - Linux
tags:
    - OSS
series:
    - OSS
---

> OSS是阿里云存储服务，简单来说OSS就是一块服务器的移动硬盘。

> 使用OSS挂载到服务器可以用来备份、保存数据文件等，而且阿里云服务器和OSS之间走的内部网络，不需要流量（OSS流量要钱）。

## **1、安装 ossfs**

首先安装 ossfs 的安装工具：

```bash
apt install gdebi-core
```

接着下载 ossf s安装包并命名为 ossfs.deb ：

```bash
wget -O ossfs.deb http://gosspublic.alicdn.com/ossfs/ossfs_1.80.6_ubuntu18.04_amd64.deb
```

然后用安装工具安装ossfs：

```bash
gdebi ossfs.deb
```

## **2、配置账号访问信息**

首先在OSS控制台找到 AccessKey 的 id 和 secret ，以及需要挂载的 bucket ，接着把账号信息写入 passwd 文件：

```bash
echo bucket:id:secret > /etc/passwd-ossfs
```

然后配置文件权限：

```bash
chmod 640 /etc/passwd-ossfs
```

## **3、挂载 ossfs**

首先新建挂载文件夹：

```bash
mkdir /mnt/oss
```

然后把OSS挂载到 /mnt/oss ：

```bash
ossfs bucket /mnt/oss -ourl=http://oss-cn-hangzhou-internal.aliyuncs.com
```

## **4、最最重要的一步，就是开机自动挂载OSS**

原本是想通过修改 fstab 来实现的，但是考虑到如果出错容易导致无法启动，再参考网上的教程后，选择自定义服务来自动挂载OSS。

首先编辑启动脚本，新建 /usr/local/ossfs.sh 文件，输入：

```bash
#! /bin/bash
#
# ossfs      Automount Aliyun OSS Bucket in the specified direcotry.
#
# description: Activates/Deactivates ossfs configured to start at boot time.
ossfs bucket /mnt/oss -ourl=http://oss-cn-hangzhou-internal.aliyuncs.com -o allow_other
```

修改脚本文件为 755：

```bash
chmod 755 /usr/local/ossfs.sh
```

接下来就是自定义服务了。

新建服务文件：

```bash
vi /etc/systemd/system/ossfs.service
chmod 664 /etc/systemd/system/ossfs.service
```

在服务文件里写入：

```bash
[Unit]
Description=Auto OSS
[Service]
Type=forking
ExecStart=/usr/local/ossfs.sh
[Install]
WantedBy=multi-user.target
```

重载服务、启用服务：

```bash
systemctl daemon-reload
systemctl enable ossfs
```

大功告成，接下来重启，看看服务有没有启动：

```bash
systemctl status ossfs
```

> 参考链接：
[https://help.aliyun.com/document_detail/153892.html](https://help.aliyun.com/document_detail/153892.html)

[https://blog.csdn.net/abc_12366/article/details/87552848](https://blog.csdn.net/abc_12366/article/details/87552848)