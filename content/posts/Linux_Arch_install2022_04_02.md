---
title: "2021 Archlinux双系统安装教程（超详细）"
date: 2022-04-02T20:37:35+08:00
draft: false
ShowCodeCopyButtons: true
categories:
    - Linux
tags:
    - Arch
series:
    - Arch
---
> 原文地址： [https://zhuanlan.zhihu.com/p/138951848](https://zhuanlan.zhihu.com/p/138951848)

如果你不想折腾，直接上Manjaro，如果你喜欢折腾、有时间并且有一定的能力，Arch欢迎你

这篇教程来帮助那些想用Arch但是又害怕命令行的同学（Arch Wiki写得很详细，但其实对小白来说还是有难度）

因为截图很重要这里我按照YouTube上一个可靠教程来贴截图讲解

有条件的也可以直接去看 [YouTube原视频](https://www.youtube.com/watch?v=QU0KRTgRJZU&t=14s)

我的CPU是Intel i5-8250U（x86_64）

**启动方式是UEFI而非BIOS（重要）**

显卡是UHD 620+NVIDIA MX150

安装Arch的硬盘是Samsang 970 EVO（NVME）

硬盘的类型最好确认一下，如果是NVMe的，先进BIOS修改 从硬盘的启动方式 为AHCI，否则你进入安装界面不会看到你的NVMe硬盘

**为确保一次成功，以下步骤（包括输入的命令）如果你不是很懂不要颠倒顺序，此外请保证安装时有顺畅的网络连接**

**因为EFI分区用的是windows的 而这个区还只有100MB 所以建议借助一些分区工具比如傲梅分区进入PE把这个区扩大 具体操作可以直接百度**

## 0.下载ISO

[Arch Linux - Downloads](www.archlinux.org/download/)

建议去下面找中国的镜像下

分区使用Windows的磁盘管理就行，没必要用DiskGenius

这里我使用的分区方案是 只额外分一个区来挂载 / 目录 EFI利用Windows的EFI分区

不使用swap分区 而是swap文件

![这里分了250G](https://pic2.zhimg.com/80/v2-54e0ac602cbc3fca1859bc1f5ea6132d_1440w.png)

## 2.制作启动U盘

制作工具建议使用 [Rufus](https://rufus.ie/)，写入方式为DD而非ISO，选项那选择GPT而非默认的MBR

## 3.BIOS的设置

保持上一步制作好的启动U盘一直插着

开机出现品牌logo时狂按对应键进入BIOS设置比如我的Dell Inspiron就是F12

进去之后

1.禁用safeboot 2.如果你的硬盘是NVMe的，把 从硬盘的启动方式 改成 AHCI

3.修改启动顺序，把U盘的启动顺序放到最上面（此处小心，不要delete任何东西）

完成之后退出重启

重启之后就是选择，回车进入arch iso

## 4.检查网络

输入下面指令检查：

```bash
ip a
```

![](https://pic3.zhimg.com/80/v2-2cce585de83d1c35149b117af9558de2_1440w.jpg)

这里用的是有线连接，如果你用的是无线连接需要按照下面的步骤连接到无线网：

输入

```bash
iwctl
```

进入iwd模式，输入

```bash
device list
```

查看你的网卡名字，这里假设是wlan0，输入

```bash
station wlan0 scan
```

检查扫描网络，输入

```bash
station wlan0 get-networks
```

查看网络名字，假设名字叫BUPT-portal，输入

```bash
station wlan0 connect BUPT-portal
```

接着输入密码（如果有密码的话），输入

```bash
exit
```

退出iwd模式

连接成功之后，检查可以连接到pacman源

```bash
pacman -Syyy
```

![如图说明一切正常](https://pic3.zhimg.com/80/v2-da5d5bc2996a7b6990673c4c4700d77a_1440w.png)

重新设置mirrorlist（可选，建议）：

使用reflector来获取速度最快的6个镜像，并将地址保存至/etc/pacman.d/mirrorlist

```bash
reflector -c China -a 6 --sort rate --save /etc/pacman.d/mirrorlist
```

## 5.硬盘

### 1.检查硬盘

```bash
lsblk
```

![](https://pic3.zhimg.com/80/v2-202971d782206ee00df4894348acf2b6_1440w.jpg)

这里没有看到之前划分好的空间，不要慌，那是因为之前只是划了空间，并没有建立分区

### 2.建立分区

因为之前划好的空间在nvme0n1上，所以执行

```bash
cfdisk /dev/nvme0n1
```

这里以你个人情况而定 sda sdb 或是其他的

![这里我们就能看到最后有250G的Free Space](https://pic2.zhimg.com/80/v2-49855cb185cf06755485c570310f0af1_1440w.jpg)

![](https://pic2.zhimg.com/80/v2-d254c8449f527d10e724fe7b0c5cd465_1440w.jpg)

选择New 回车

![](https://pic2.zhimg.com/80/v2-beb16fd18d81dc69d910bc0f9578cacd_1440w.png)

这里就输入250G 回车

![](https://pic1.zhimg.com/80/v2-4c370c9718b302ed0774c8f2a9faad44_1440w.jpg)

选择Write 回车 输入 yes 回车

写入完成 选择Quit 回车退出

检查分区情况

```bash
lsblk
```

![建立分区之后就可以看到分好的250G区](https://pic1.zhimg.com/80/v2-553670cd4ee4b584a616078bc750c948_1440w.jpg)

### 3.分区格式化

将刚刚分好的区格式化为ext4格式，这里认准分区号是nvme0n1p5

```bash
mkfs.ext4 /dev/nvme0n1p5
```

![](https://pic4.zhimg.com/80/v2-311d75a2b56ac97898484f449341009b_1440w.jpg)

### 4.挂载分区

先挂载/分区，同样，这里分区号也是nvme0n1p5

```bash
mount /dev/nvme0n1p5 /mnt
```

这里利用Windows的EFI分区，检查EFI分区号

```bash
lsblk
```

![这里可以看到是nvme0n1p2](https://pic2.zhimg.com/80/v2-dcb17539e1c3938c74b377a65526001d_1440w.jpg)

建立boot文件夹

```bash
mkdir /mnt/boot
```

挂载EFI分区

```bash
mount /dev/nvme0n1p2 /mnt/boot
```

![](https://pic3.zhimg.com/80/v2-265d33a99301d176fae47b6343311c1e_1440w.jpg)

## **6.安装基本系统**

执行

```bash
pacstrap /mnt base linux linux-firmware nano
```

等待安装完毕

（如果你不想用默认的内核，也可以使用linux-lts, linux-zen, linux-hardened，具体介绍请看[Wiki](https://wiki.archlinux.org/index.php/kernel)）

## **7.生成fstab文件**

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

检查生成的fstab文件

```bash
cat /mnt/etc/fstab
```

![如图说明一切正确](https://pic4.zhimg.com/80/v2-bb67b920ff61eb7ea40d8c073bd15857_1440w.jpg)

如图说明一切正确

## **8.正式配置新系统**

### 1.切换到装好的系统

```bash
arch-chroot /mnt
```

### 2.建立swapfile（建议，没有swap空间无法休眠）

> 在 ext4 上使用 swapfile 的用户请注意，升级到 5.7.x 内核后可能出现诸如「kernel: swapon: swapfile has holes」这样的报错而无法启用 swapfile 。使用 dd 命令创建 swapfile （而非 fallocate） 可能可以解决问题，也可以回退 5.6 系列内核等待上游修复。
> 
> Arch Linux 错误跟踪：[https://bugs.archlinux.org/task/66921](https://bugs.archlinux.org/task/66921)
> 
> 内核错误跟踪：[https://bugzilla.kernel.org/show_bug.cgi?id=207585](https://bugzilla.kernel.org/show_bug.cgi?id=207585)

如果之前安装的内核是linux-lts：

```bash
fallocate -l 2GB /swapfile
```

注意：命令中是 **小写字母l** 而非 数字1 也非 字母i的大写

如果之前安装的内核不是linux-lts，这里创建swapfile需要使用dd命令：

```bash
dd if=/dev/zero of=/swapfile bs=2048 count=1048576 status=progress
```

这里分了2G作为swapfile

改权限

```bash
chmod 600 /swapfile
```

建立swap空间

```bash
mkswap /swapfile
```

激活swap空间

```bash
swapon /swapfile
```

修改/etc/fstab文件

```bash
nano /etc/fstab
```

到文件末尾输入

```bash
/swapfile none swap defaults 0 0
```

![保存退出](https://pic4.zhimg.com/80/v2-32f9a86a9806695b1a4c6d94c79d25e3_1440w.jpg)

### 3.设置时区

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

或

```bash
timedatectl set-timezone Asia/Shanghai
```

同步硬件时钟

```bash
hwclock --systohc
```

### 4.设置locale

```bash
nano /etc/locale.gen
```

Ctrl+W 输入 #en_US 回车 找到UTF-8那一行 删掉前面的#（取消注释）

Ctrl+W 输入 #zh_CN 回车 找到UTF-8那一行 删掉前面的#（取消注释）

保存退出

生成locale

```bash
locale-gen
```

创建并写入/etc/locale.conf文件

```bash
nano /etc/locale.conf 
```

填入内容，注意这里只能填这个

```bash
LANG=en_US.UTF-8
```

5.创建并写入hostname

```bash
nano /etc/hostname
```

这里我写入的是 arch 作为hostname，你也可以输别的

![保存退出](https://pic2.zhimg.com/80/v2-c6f62d0bf545ebaa952a1746cd7668ad_1440w.png)

### 6.修改hosts

```bash
nano /etc/hosts
```

写入内容如图（中间的空白用tab而非空格），arch替换为你之前在hostname里写入的内容，其他都按照图里面的写（注意最后一行的ip是127.0.1.1）

![](https://pic3.zhimg.com/80/v2-54fc38004dc15b76d68983a78e59928a_1440w.jpg)

保存退出

建议上述编辑的内容都用cat输出检查一下

### 7.为root用户创建密码

```bash
passwd
```

然后输入并确认密码（linux终端的密码没有回显，输完直接回车就好）

### 8.创建启动器

安装基本的包，这里使用grub为启动器

```bash
pacman -S grub efibootmgr networkmanager network-manager-applet dialog wireless_tools wpa_supplicant os-prober mtools dosfstools ntfs-3g base-devel linux-headers reflector git sudo
```

**如果你不知道这些包的作用，请务必确保输入的指令与上面的一致**

检查完毕回车，需要选择直接回车就好，等待安装结束

如果你是intel的cpu，需要安装intel的微码文件

```bash
pacman -S intel-ucode
```

如果是amd

```bash
pacman -S amd-ucode
```

2021.06.16更新：

> Grub 2.06 更新 os-prober 用户需要手动干预
> 
> grub 2.06 更新已经进入官方源，本次更新有以下两个需要注意的变化：
>
> 1. 如果您正在使用 os-prober 生成其他系统的引导项，grub 2.06 不再自动启用 os-prober，您需要添加 GRUB_DISABLE_OS_PROBER=false 至 /etc/default/grub 配置文件中并且重新运行 grub-mkconfig
>
> 2. grub 2.06 现在会自动添加 固件设置菜单 引导项目，无需手动创建

鉴于此需要手动启用os-prober来确保Windows能被正确识别：

输入

```bash
nano /etc/default/grub
```

在里面找一条空行输入

```bash
GRUB_DISABLE_OS_PROBER=false
```

之后Ctrl-X 加Y回车保存退出

![完成之后输入](https://pic4.zhimg.com/80/v2-405f1485261b0213aceef88ca833909b_1440w.jpg)

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Arch
```

确保输入指令完全正确回车

![生成grub.cfg](https://pic3.zhimg.com/80/v2-9a43ea86f2ab93b70b53549d3ee84c52_1440w.png)

生成grub.cfg

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

完成之后如图

![](https://pic3.zhimg.com/80/v2-de2e93e62d337ba61654e02001a1f226_1440w.jpg)

## 9.退出新系统并取消挂载

```bash
exit
```

![A1](https://pic2.zhimg.com/80/v2-240ca9989b005714cdeaf20e85c7340d_1440w.jpg)

```bash
umount -a
```

![A1-1](https://pic1.zhimg.com/80/v2-e87ae5d5e79da0247e93cf2e19e3bc50_1440w.jpg)

重启

```bash
reboot
```

启动时请拔出u盘

## 10.进入装好的Arch系统并激活网络

进去之后 先输入 root 回车 输入密码 回车

启动网络服务

```bash
systemctl enable --now NetworkManager
```

设置WiFi

```bash
nmtui
```

![A1-2](https://pic4.zhimg.com/80/v2-3be8707847310be9f8f8d86de0799c8f_1440w.jpg)

回车

![A1-3](https://pic1.zhimg.com/80/v2-8e0546e1ea798ec7f146f4a05924e28c_1440w.jpg)

选择你要连接到的WiFi 输入密码 回车 然后退出

## 11.新建用户并授权

```bash
useradd -m -G wheel mir
```

wheel后面是你的用户名，这里输入的是mir

为用户创建密码

```bash
passwd mir
```

输入并确认密码

授权

```bash
EDITOR=nano visudo
```

Ctrl+W 输入 # %wheel 回车 找到这行 删除前面的 #（取消注释）

![A1-4](https://pic1.zhimg.com/80/v2-8aaf436214afc9aaefbee330f96f4690_1440w.jpg)

保存退出

## 12.安装显卡驱动

安装AMD集显驱动

```bash
pacman -S xf86-video-amdgpu
```

安装NVIDIA独显驱动

```bash
pacman -S nvidia nvidia-utils
```

## 13.安装Display Server

这里用的是开源世界最为流行的xorg

```bash
pacman -S xorg
```

出现选择直接回车即可

## 14.安装Display Manager

这里需要按你要安装的桌面环境而定，这里没有列出的可以自己去ArchWiki查

Gnome：

```bash
pacman -S gdm
```

KDE：

```bash
pacman -S sddm
```

```bash
Xfce || DDE：
```

```bash
pacman -S lightdm lightdm-gtk-greeter
```

设置开机自动启动，以gdm为例：

systemctl enable gdm
如果是别的请将这里的gdm替换为你安装的那个dm

## 15.安装Desktop Environment

Gnome：

```bash
pacman -S gnome
```

KDE：

```bash
pacman -S plasma kde-applications packagekit-qt5
```

Xfce：

```bash
pacman -S xfce4 xfce4-goodies
```

DDE：

```bash
pacman -S deepin deepin-extra
```

同样 需要选择时直接回车

## 16.添加archlinuxcn源

```bash
nano /etc/pacman.conf
```

在最后加上下面两行（我这里使用了北外的镜像站）

```bash
[archlinuxcn]
Server = https://mirrors.bfsu.edu.cn/archlinuxcn/$arch
```

同时取消对multilib源的注释

![A1-5](https://pic3.zhimg.com/80/v2-5519f934341533ce81fdff7259b0e9d6_1440w.jpg)

保存退出之后同步并安装archlinuxcn-keyring

```bash
pacman -Syu && pacman -S archlinuxcn-keyring
```

最后不要忘记安装中文的字体，如果这一步不装进去图形界面之后还是要装：

```bash
pacman -S ttf-sarasa-gothic noto-fonts-cjk
```

我这里安装的是更纱黑体和noto cjk，包比较大，耐心等待安装完毕。

最后重启

```bash
reboot
```

在grub界面选择archlinux回车

当你看到登录界面时，恭喜你，一个相对完整的Arch安装完毕，Enjoy it！

---

进一步配置可以看看专栏前面那篇Manjaro-KDE的配置，装点常用的软件，大体上就能用了，然后根据自己的情况配配显卡驱动，就差不多能玩游戏了

关于N卡的启用与切换，建议使用[optimus-manager](https://github.com/Askannz/optimus-manager)（其他方案我都试过不好使

```bash
sudo pacman -S optimus-manager
```

没有DE只有一个Bspwm 这是我的dotfiles：

[Dracula&Nord Dotfiles](github.com/MiraculousMoon/bspwm-dotfiles.git)

此外还有i3wm的：

[https://github.com/ayamir/i3-dotfiles](github.com/ayamir/i3-dotfiles)

dwm的：

[https://github.com/ayamir/dwm-dotfiles](github.com/ayamir/dwm-dotfiles)

spectrwm，xmonad，sway的：

[https://github.com/ayamir/nord-and-light](github.com/ayamir/nord-and-light)

最后多说一句，如果你装完了，就要意识到自己拥有了Arch WiKi（世界上最好的WiKi之一），这也是Arch用户令人羡慕的一点。遇到问题先找Arch WiKi，再去找别的资料