---
title: "MBR&GPT硬盘分区类型&属性详解（Win下更改/设置OEM/恢复分区方法）"
date: 2022-04-02T17:01:53+08:00
draft: false
ShowCodeCopyButtons: true
categories:
    - Windows
tags:
    - Diskpart
series:
    - Diskpart
---

> 原文地址： [https://www.iruanmi.com/mbr-and-gpt-partition-type-and-attributes/](https://www.iruanmi.com/mbr-and-gpt-partition-type-and-attributes/)

打开Windows系统的磁盘管理，你可能会在硬盘上发现一个或若干个特殊分区，他们一般都带有特殊的标记，并且通常都具有隐藏属性，比如，OEM分区、恢复分区皆如此。那么这些分区是靠什么表现出特殊属性的呢？而我们又能不能改变这些属性呢？本文就来详细解释这个问题。

> 为方便理解本文内容，建议先阅读[《UEFI+GPT引导基础篇（一）：什么是GPT，什么是UEFI？》](https://www.iruanmi.com/what-is-gpt-and-what-is-uefi/) 一文，这篇文章简单介绍了MBR和GPT硬盘分区原理，告诉我们硬盘上各分区的相关信息都存储在各自的分区表中：MBR硬盘分区信息存储在MBR分区表中；而GPT硬盘分区信息则存储在GPT分区表中，除此之外，GPT硬盘还包含一个PMBR分区表。本文下面要讲到的东西可以看作是对这些内容的一点扩展，它捎带回答了这两个问题：
> 
> 1、GPT硬盘中存在PMBR分区表和GPT分区表，那么系统凭借什么将其识别为GPT硬盘？
>
> 2、我们还知道，MBR硬盘也可以实现UEFI引导，那么其用于实现引导的分区（FAT32分区）是不是也有像GPT硬盘中的“EFI系统分区”这样鲜明的标志？

## **MBR硬盘和GPT硬盘使用不同的分区规则，我们先来看MBR硬盘。**

MBR硬盘的MBR分区表中包含了硬盘上各主分区的分区信息，每个分区信息中都有一段内容（1字节，即8位）用来表示分区类型。[可以在这里查看分区类型列表](http://www.win.tue.nl/~aeb/partitions/partition_types-1.html)（十六进制表示）。Windows下可识别的分区类型主要有：

0x42 表示LDM数据分区

0x27 表示恢复分区（WinRE分区、Acer等系统备份分区）。

0x07 表示普通分区（Windows分区、数据分区。默认分区类型。）

0x12 表示OEM分区（康柏、IBM Thinkpad）。

0x84 表示OEM分区（Intel Rapid Start technology）。

0xDE 表示OEM分区（戴尔）。

0xFE 表示OEM分区

0xA0 表示OEM分区（Laptop hibernation partition）

0xEE 表示该分区表是PMBR，紧随其后的应该是GPT分区表头和GPT分区表，因此这是一块GPT硬盘。

0xEF 表示EFI系统分区


Windows正是根据分区表中设定的分区类型决定分区的用途（OEM或其他）和属性（是否隐藏等）。其他大多数分区类型Windows无法识别。

## Windows下更改分区类型的方法

自Vista开始，系统自带的diskpart分区管理工具已具备更改分区类型的功能。更改分区类型，只需在具有管理员身份的CMD中依次执行以下几个命令即可（括号内为注释内容）：

```
Diskpart（打开diskpart工具）

List disk（可选。帮助查看连接到电脑的所有存储器及其编号）

Select disk N（选择地N个硬盘，N为硬盘编号）

List part（可选。帮助查看选定硬盘上的所有分区及其编号）

Select part N（选定第N个分区，N代表分区编号）

Set id = xx（设定分区类型，xx代表十六进制分区类型ID，省略0x）
```

**举两个我们可能需要用到的例子：**

① 改变隐藏的OEM分区类型，从而能够查看OEM分区中的内容。

**注意：** 如果还想更改回去，请在select part之后运行detail part记下分区默认的分区类型，方便事后还原。

![分区1](https://js.tinqin881.top/uploads%2F2022%2F04%2F02%2FA1-1.gif?Expires=1648891470)

完成图中的操作后，如果没有自动分配盘符，可以尝试重启**或**在磁盘管理中手动添加“驱动器号”**或**紧接着图中最后一步执行以下命令添加盘符（e为盘符）。
```
assign letter=e
```
同理，如果要将某一个分区设置为OEM分区，只需将其分区类型设置为出厂默认的OEM分区类型ID或12或DE即可。

② 作为博客[Win8/8.1备份教程](https://www.iruanmi.com/win8-or-win8-1-system-backup/)的补充。我们将系统备份映像存放到单独的隐藏分区中，以保护备份映像不受到损坏。

首先，准备一个可容纳备份映像文件的空分区（主分区、逻辑分区都可以），将备份映像按下图所示的路径存放（\sources\install.wim）

![分区2](https://js.tinqin881.top/uploads%2F2022%2F04%2F02%2FA1-2.gif?Expires=1648891487)

然后，配置恢复映像，将分区类型设置为 **“恢复分区”** 。如下图所示。

![分区3](https://js.tinqin881.top/uploads%2F2022%2F04%2F02%2FA1-3.gif?Expires=1648891501)

完成图中的步骤，用于恢复系统的系统备份分区就被隐藏掉了。如果计算机中还能够看到该分区（有盘符），紧接着图中最后一步运行下面的命令删除盘符：
```
remove
```

---

## **那么，GPT硬盘上又是怎么样的情况呢？**

在GPT分区表中的分区信息中同样有一段用于表示分区类型的内容（16字节，即128位）。可以[在这里查看分区类型](http://zh.wikipedia.org/wiki/GUID%E7%A3%81%E7%A2%9F%E5%88%86%E5%89%B2%E8%A1%A8#.E5.88.86.E5.8C.BA.E7.B1.BB.E5.9E.8BGUID)列表（十六进制的GUID表示）。**Windows下常见的GUID分区类型主要有：**

```
C12A7328-F81F-11D2-BA4B-00A0C93EC93B            EFI系统分区
DE94BBA4-06D1-4D40-A16A-BFD50179D6AC         WinRE恢复环境分区、系统备份分区
E3C9E316-0B5C-4DB8-817D-F92DF00215AE            微软保留（MSR）分区
EBD0A0A2-B9E5-4433-87C0-68B6B72699C7            基本数据分区
5808C8AA-7E8F-42E0-85D2-E1E90434CFB3            逻辑软盘管理工具元数据分区
AF9B60A0-1431-4F62-BC68-3311714A69AD            逻辑软盘管理工具数据分区
37AFFC90-EF7D-4e96-91C3-2D7AE055B174          IBM通用并行文件系统(GPFS)分区
E75CAF8F-F680-4CEE-AFA3-B001E56EFC2D          存储空间（Storage Spaces）分区

BFBFAFE7-A34F-448A-9A5B-6213EB736C22           Lenovo OEM分区（一键还原启动分区）
F4019732-066E-4E12-8273-346C5641494F               Sony OEM分区（一键还原启动分区）

GPT分区类型用于区别分区的用途，GPT分区表中的分区信息中除了分区类型外，还用了另一段区域（8字节，即64位）来表示分区属性，各位作用如下：

0x0000000000000001（0位）  将分区表示为必需分区，不允许用户更改数据（Windows下将标记为OEM分区）
0x8000000000000000（63位）   当硬盘被挂载到另一台电脑时默认不分配盘符。
0x4000000000000000（62位）  表示该分区不可被检测到。
0x2000000000000000（61位）  表述该分区为另一个分区的卷影拷贝。
0x1000000000000000（60位）  为分区设置只读属性。
```

关于分区属性，更详细的介绍参考[《CREATE_PARTITION_PARAMETERS structure》](http://msdn.microsoft.com/en-us/library/aa381635(VS.85).aspx)

## **Windows下通常采用以下分区类型和分区属性组合：**

普通数据分区——EBD0A0A2-B9E5-4433-87C0-68B6B72699C7——0x0000000000000000

OEM分区——无特定GUID值，OEM决定——0x8000000000000001

WinRE分区——DE94BBA4-06D1-4D40-A16A-BFD50179D6AC——0x8000000000000001

EFI系统分区——C12A7328-F81F-11D2-BA4B-00A0C93EC93B——0x8000000000000001

MSR保留分区——E3C9E316-0B5C-4DB8-817D-F92DF00215AE——0x8000000000000000

恢复/备份分区——DE94BBA4-06D1-4D40-A16A-BFD50179D6AC——0x8000000000000001


## **更改GPT分区类型和分区属性的方法：**
在管理员身份的CMD中（Vista以上版本系统）依次执行以下命令即可（括号内为注释内容）：

```
Diskpart    （打开diskpart工具）
List disk    （可选。帮助查看连接到电脑的所有存储器及其编号）
Select disk N    （选择地N个硬盘，N为硬盘编号）
List part    （可选。帮助查看选定硬盘上的所有分区及其编号）
Select part N    （选定第N个分区，N代表分区编号）
Set id = xx    （设定分区类型，xx代表十六进制GUID分区类型ID）
gpt attributes = 0xXXXXXXXXXXXXXXXX    （设置分区属性，XXXXXXXXXXXXXXXX代表分区属性）
```

## **同样采用上文MBR硬盘中的两个例子，其在GPT硬盘中的操作方法如下：**

① 改变隐藏的OEM分区类型，从而能够查看OEM分区中的内容。

**注意：** 如果还想更改回去，请在select part之后运行detail part记下分区默认的分区类型和属性，方便事后还原。

![](https://js.tinqin881.top/uploads%2F2022%2F04%2F02%2FA1-4.gif?Expires=1648891514)

完成图中的操作后，如果没有自动分配盘符，可以尝试重启**或**在磁盘管理中手动添加“驱动器号”**或**紧接着图中最后一步执行以下命令添加盘符（e为盘符）。
```
assign letter=e
```

同理，如果要将某一个分区设置为OEM分区，只需将其分区类型设置为出厂默认或{EBD0A0A2-B9E5-4433-87C0-68B6B72699C7}或其他非特殊（即上文列表中之外）的GUID，再将其属性设置为0x8000000000000001（隐藏）或0x0000000000000001即可。

② 作为博客[Win8/8.1备份教程](https://www.iruanmi.com/win8-or-win8-1-system-backup/)的补充。我们将系统备份映像存放到单独的隐藏分区中，以保护备份映像不受到损坏。

首先，准备一个可容纳备份映像文件的空分区，将备份映像按下图所示的路径存放（\sources\install.wim）

![](https://js.tinqin881.top/uploads%2F2022%2F04%2F02%2FA1-5.gif?Expires=1648891526)

然后，配置恢复映像，将分区类型设置为 **“恢复分区”** 。如下图所示。

![](https://js.tinqin881.top/uploads%2F2022%2F04%2F02%2FA1-6.gif?Expires=1648891534)

完成图中的步骤，用于恢复系统的系统备份分区就被隐藏掉了。如果计算机中还能够看到该分区（有盘符），紧接着图中最后一步运行下面的命令删除盘符即可：

```
remove
```