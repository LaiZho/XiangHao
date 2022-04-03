---
title: "通过 iso 重装阿里云 ECS"
date: 2022-04-03T20:13:00+08:00
draft: false
ShowCodeCopyButtons: true
categories:
    - Linux
tags:
    - Debian
series:
    - Debian
---
> **原文地址**：[https://blog.kuretru.com/posts/59955d97/](https://blog.kuretru.com/posts/59955d97/)

> 阿里云 ECS 内置了多种系统镜像，但是每个镜像均内置了阿里云云盾等后门程序，这篇文章教你使用自定义 iso 文件重新安装一个纯净的系统。这篇文章以在阿里云 ECS 上安装 `CentOS 8.1` 为例，其他 IDC 或系统可以参照修改。

### 1. 在阿里云 ECS 控制台重置系统为 `CentOS 8`

### 2. 购买一块按量付费的最小容量的云盘，每小时 1 分钱 (购买前先充值 100 块押金，释放 24 小时后可以提现，若金额小于 1 分钱，实测不收费)

### 3. 在目标 ECS 上挂载该云盘，在系统中该云盘被识别为 `/dev/vdb`

### 4. 格式化并挂载至 ECS

```bash
mkfs -t ext4 /dev/vdb
mkdir /mnt/iso
mount /dev/vdb /mnt/iso
cd /mnt/iso
```

### 5. 下载要安装的自定义 iso，可以通过阿里云的开源镜像站下载，并验证文件的完整性

```bash
wget https://mirrors.aliyun.com/centos/8.1.1911/isos/x86_64/CentOS-8.1.1911-x86_64-dvd1.iso
sha256sum CentOS-8.1.1911-x86_64-dvd1.iso
```

### 6. 记录新增云盘的 UUID

```bash
ls -l /dev/disk/by-uuid/ | grep vdb
```

### 7. 配置 grub，在 `/etc/grub.d/40_custom` 文件的最下方新增，并替换 UUID

```bash
vim /etc/grub.d/40_custom

menuentry "CentOS-8.1.1911-x86_64-dvd1.iso" {
    set iso_path="/CentOS-8.1.1911-x86_64-dvd1.iso"
    loopback loop (hd1)/$iso_path
    linux (loop)/isolinux/vmlinuz inst.stage2=hd:UUID="此处为上一步所记录的UUID" noeject iso-scan/filename=$iso_path
    initrd (loop)/isolinux/initrd.img
}
```

### 8. 配置默认启动项为光盘启动，编辑 `/etc/default/grub` 文件

```bash
vim /etc/default/grub

GRUB_DEFAULT="CentOS-8.1.1911-x86_64-dvd1.iso"
```

### 9. 生成 grub 配置

```bash
grub2-mkconfig --output=/boot/grub2/grub.cfg
```

### 10. 重新启动，通过 VNC 连接 ECS，即可看到已进入光盘安装画面

### 11. 正常安装完成后，释放新增的云盘

## **注意事项**

- `CentOS 8.1.1911` 无法在 512MB 内存下安装
- 阿里云国际版新手套餐不支持该方法，请使用 `dd` 命令的方法重装