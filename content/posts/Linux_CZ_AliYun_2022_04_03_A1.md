---
title: "阿里云通过网络安装全新 Debian"
date: 2022-04-03T17:28:33+08:00
draft: false
ShowCodeCopyButtons: true
categories:
    - Linux
tags:
    - Debian
series:
    - Debian
---

> **转载自**：[https://sb.sb/blog/install-debian-alibabacloud/](https://sb.sb/blog/install-debian-alibabacloud/)

> 由于众多云主机商家的系统模板总是让人不满意，不是多装了软件就是瞎 JB 写了一些配置，而且 Debian 9.x 都发布一个多月了，国内云主机商能跟上步伐的很少，美团云甚至还停留在 Debian 7.0 时代，而且发了几次工单都没得到任何反应，这是绝对不能忍的
> 作为一只有洁癖的人，我决定亲自动手重新安装[阿里云](https://u.nu/aliyun)和[美团云](https://u.nu/mtyun)的系统，这里我们以阿里云为例，希望安装的操作系统是最新版本 Debian 9.x Stretch

## 1、准备工作和必要条件

准备工作

1、账号一枚
2、开通一个云主机并且选择操作系统选择 Debian 8
3、泡一杯茶静下心，按照我的教程一步一步来
必要条件

你得有后台登陆权限
1、云主机商得提供 Console （类似 VNC / KVM ，阿里云叫做远程连接）
2、云主机商使用的 KVM 构架，并且没有使用 LVM 分区
3、使用 Chrome 浏览器，Firefox 的某些快捷键会冲突

## 2、记录主机的 IP 信息

开通云主机之后，首先我们用 `ip route` 命令记录一下 IP 信息，阿里云的网络分两种，专用网络（VPC)和经典网络（Classic），其中经典网络和美团云一样，分两张网卡，网卡 0 是内网，网卡 1 是公网，专用网络只有一个网卡

这里我们以经典网络为例

```bash
root@alibabacloud:~# ip route
default via 47.88.88.247 dev eth1 
10.0.0.0/8 via 10.88.88.247 dev eth0 
10.88.88.0/22 dev eth0  proto kernel  scope link  src 10.88.88.88 
47.88.88.0/22 dev eth1  proto kernel  scope link  src 47.88.88.88 
100.64.0.0/10 via 10.88.88.247 dev eth0 
172.16.0.0/12 via 10.88.88.247 dev eth0
```

其中 `47.88.88.88` 为公网 IP， 网关是 `47.88.88.247`， CIDR 是 `47.88.88.88/22` 位于网卡 1，也就是第二张网卡

`10.88.88.88` 为内网 IP，网关是 ‘10.88.88.247’，CIDR 是 `10.88.88.88/22` 位于网卡 0，也就是第一张网卡

为何要记录这个信息呢？这个下面讲

## 3、下载 Debian 9.x 硬盘安装文件

于我们是全新的机器，不需要保留任何数据也不需要保留任何分区信息，所以我们直接按照 Debian 官方的[说法](https://www.debian.org/releases/stable/amd64/ch05s01.html.en#boot-initrd)，通过 GRUB2 启动进行抹盘安装

首先，新建立一个 `/boot/newinstall` 目录并且下载必要的两个文件 `linux` 和 `initrd.gz`

```bash
mkdir /boot/newinstall && cd /boot/newinstall
wget http://mirrors.ustc.edu.cn/debian/dists/stable/main/installer-amd64/current/images/netboot/debian-installer/amd64/initrd.gz
wget http://mirrors.ustc.edu.cn/debian/dists/stable/main/installer-amd64/current/images/netboot/debian-installer/amd64/linux
```

因为 Debian 8 使用的是 GRUB2，所以我们直接修改 `/boot/grub/grub.cfg` 文件，加入启动项

```bash
cat >> /boot/grub/grub.cfg << EOF
menuentry 'Debian Stable New Install' {
insmod part_msdos
insmod ext2
set root='(hd0,msdos1)'
linux /boot/newinstall/linux
initrd /boot/newinstall/initrd.gz
}
EOF
```

另外，你需要保证 `/etc/default/grub` 文件是可以让你选择进入界面的，举例如下

```bash
# If you change this file, run 'update-grub' afterwards to update
# /boot/grub/grub.cfg.
# For full documentation of the options in this file, see:
#   info -f grub -n 'Simple configuration'

GRUB_DEFAULT=0
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT="quiet"
GRUB_CMDLINE_LINUX="debian-installer=en_US net.ifnames=0 vga=792"

# Uncomment to enable BadRAM filtering, modify to suit your needs
# This works with Linux (no patch required) and with any kernel that obtains
# the memory map information from GRUB (GNU Mach, kernel of FreeBSD ...)
#GRUB_BADRAM="0x01234567,0xfefefefe,0x89abcdef,0xefefefef"

# Uncomment to disable graphical terminal (grub-pc only)
#GRUB_TERMINAL=console

# The resolution used on graphical terminal
# note that you can use only modes which your graphic card supports via VBE
# you can see them in real GRUB with the command `vbeinfo'
#GRUB_GFXMODE=640x480

# Uncomment if you don't want GRUB to pass "root=UUID=xxx" parameter to Linux
#GRUB_DISABLE_LINUX_UUID=true

# Uncomment to disable generation of recovery mode menu entries
#GRUB_DISABLE_RECOVERY="true"

# Uncomment to get a beep at grub start
#GRUB_INIT_TUNE="480 440 1"
```

如果这个文件更改过，那么还需要 `update-grub` 命令重新更新 grub 并且修改 `/boot/grub/grub.cfg` 文件

如果是更旧的系统，比如 Debian 7 或 Ubuntu 14.04 用的 GRUB1，那么你需要修改 `/boot/grub/menu.lst` 加入启动项

```bash
cat >> /boot/grub/menu.lst << EOF
title  Debian Stable New Install
root   (hd0,0)
kernel /boot/newinstall/vmlinuz
initrd /boot/newinstall/initrd.gz
EOF
```

## 4、安装全新的 Debian 9.x Stretch

首先，你需要开启阿里云后台的远程连接，进入后应该能看到黑色屏幕让你登陆的界面，不用去鸟他，我们直接 SSH 用 `reboot` 命令重启，然后我们可以看到阿里云的机器迅速重启并且到了让你选择启动项目的界面，选择 `Debian Stable New Install` 然后回车即可

**切记** 一定要手快按 ↓ 否则就又进入原来的系统了

![A1-1](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/grub.jpeg)

接着选择安装的语言，如无特别需求，请选择 `English` ，因为这会成为系统的默认语言，如果选择其他语言可能有些程序会出现各种奇怪错误

![A1-2](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/1.jpeg)

然后选择国家，这回影响到后续的服务器默认源和时区，建议选择服务器所在地，中国的话在 `Asia` > `China`，因为我们的服务器是香港的，所以我们选 `Hong Kong`

![A1-3](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/2.jpeg)

接着会让你选择主网卡，注意这里必须选公网的网卡，因为是网络安装，不联网是无法继续的

上文提到过，阿里云的经典网络以及美团云的网络都是有两个网卡，其中第二个网卡才是公网，所以我们得选 `ens4` 如果只有一个网卡那么就选默认的，具体还得看你的服务商的网络配置

![A1-4](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/3.jpeg)

默认 Debian 的安装会去尝试使用 DHCP 获取网络

![A1-5](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/4.jpeg)

在专用网络下，阿里云的机器获取 DHCP 是没问题的，而经典网络的阿里云则会失败，美团云也是如此，原因是他们 DHCP 获取的 DNS 地址是内网的，所以我们手工配置一下 IP 即可

![A1-6](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/5.jpeg)

这时候就需要我们在第二步里记录下的 IP 了，按照实际情况填写

我们选择手工配置网络

![A1-7](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/6.jpeg)

输入 IP 地址的 CIDR 形式，比如 `47.88.88.88/22`

![A1-8](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/7.jpeg)

输入网关地址，比如 `47.88.88.247`

![A1-9](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/8.jpeg)

输入 DNS 地址，比如 `8.8.8.8` 或 `223.5.5.5`

![B1-1](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/9.jpeg)

这时就会乖乖地去获取网络信息

![B1-2](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/10.jpeg)

接着输入 hostname 前缀

![B1-3](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/11.jpeg)

然后输入 hostname 的域名，没有的话可以不写，写了就会变成 DNS 搜索域名，一般不写也没问题

![B1-4](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/12.jpeg)

然后选择镜像地址

![B1-5](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/13.jpeg)

由于 Debian 在香港没有官方镜像，这里我们可以使用 [xTom](https://xtom.com.hk/) 提供的香港镜像 [mirror.xtom.com.hk](mirror.xtom.com.hk)

![B1-6](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/14.jpeg)

选择 Debian 镜像的目录

![B1-7](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/15.jpeg)

如果没有 HTTP 代理，就不需要理会，直接回车继续

![B1-8](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/16.jpeg)

接着就会开始检查需要下载的文件了

![B1-9](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/17.jpeg)

检查完毕后，会让你输入 root 密码并进行二次确认

![C1-1](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/18.jpeg)

![C1-2](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/19.jpeg)

注意 如果指定了 `root` 密码，那么默认是不装 `sudo` 的，普通用户切换 `root` 的时候需要用 `su` 命令，一般如果关闭了 `root` 密码登陆的话（默认都是关闭的），不指定 `root` 密码也无大碍

然后指定一个普通用户的名字并确认密码

![C1-3](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/20.jpeg)

![C1-4](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/21.jpeg)

![C1-5](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/22.jpeg)

如果网速不行，远程连接是很卡的，可以按 Tab 键到 `Show Password in Clear` 查看自己输入的密码是否正确，没问题再按回车确认

![C1-6](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/23.jpeg)

接下来会配置网络时间服务器（NTP）

![C1-7](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/24.jpeg)

然后就是等待下载安装各种必要的软件

![C1-8](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/25.jpeg)

接着会让你设置分区，如无特殊需求，都按照默认的确认即可

![C1-9](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/26.jpeg)

选择安装系统的硬盘

![D1-1](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/27.jpeg)

选择默认分区

![D1-2](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/28.jpeg)

确认分区无误

![D1-3](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/29.jpeg)

再次确认

![D1-4](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/30.jpeg)

好了，开始安装了

![D1-5](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/31.jpeg)

![D1-6](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/32.jpeg)

Debian 安装过程中会跳出是否愿意加入安装统计调查，自行选择即可

![D1-7](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/33.jpeg)

接下来会询问你需要装哪些软件

![D1-8](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/34.jpeg)

这里我们特别注意，因为是服务器，不需要桌面客户端，所以按空格把 `Debian desktop enviroment` 以及 `print server` 去掉，然后一定要勾选上 `SSH server` 否则没法用 SSH 连接管理服务器

![D1-9](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/35.jpeg)

继续安装软件

![E1-1](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/36.jpeg)

如果没有特殊需求，我们还是用 GRUB 来引导启动系统

![E1-2](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/37.jpeg)

选择安装 GRUB 引导程序的硬盘

![E1-3](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/38.jpeg)

开始安装 GRUB

![E1-4](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/39.jpeg)

安装完成，点继续

![E1-5](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/40.jpeg)

自动重启后，即可看到选择启动顺序里只有 Debian 一个选项了，继续后会出现登陆界面

![E1-6](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/41.jpeg)

![E1-7](https://cdn.loli.net/sb.sb.sb/post-images/Debian-Boot-Initrd/42.jpeg)

这样我们就已经安装好整个系统了

然后用你刚才创建的 username 进行 SSH 登陆操作，和常规的服务器并无区别，如需要内网则手工修改 `/etc/networ/interface` 增加 `ens3` 网卡
