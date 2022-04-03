---
title: "给 PowerShell 带来 Zsh 的体验"
date: 2022-04-03T16:48:32+08:00
draft: false
ShowCodeCopyButtons: true
categories:
    - Windows
tags:
    - Windows
series:
    - PowerShell
---

## 前言

>众所周知 cmd.exe 真的是难用的一批，但是我们有 PowerShell 啊，现在 PowerShell 也出了跨平台版本，目前最新版是 PowerShell 7。
>
>然而相对于 Linux 下配置好的 zsh 还是没有那么顺手，那么怎么样才能把 zsh 的那种体验搬到 PowerShell 里面呢？

安装 PowerShell

本文需要用到 PowerShell 7 或以上版本，可以从 GitHub 下载安装 PowerShell 7，在 release 页找一个正式版下载安装即可（预览版也行），例如 PowerShell-7.0.1-win-x64.msi。

[PowerShell Releases](https://link.zhihu.com/?target=https%3A//github.com/PowerShell/PowerShell/releases)

PowerShell 7 带来了一系列的新功能、语法和改进，如 null 传播符 `?.`、`??`，并发的 `ForEach-Object -Parallel`，`&&`和`||`操作符，自动更新检测，简短的错误提示，等等

为什么不用 Windows 自带的 PowerShell 5.1 呢？因为

慢
难用
配色丑
语法落后
不能跨平台
报错信息太长
......
当然，虽然没试过，但是本文对于 5.1 应该大部分也是可用的。

## PSReadLine
PSReadLine 是一个由微软发布的用于 PowerShell 的行读取实现，提供了以下功能：

语法着色
简单语法错误通知
良好的多行体验
可自定义的键绑定
Cmd和Emacs模式
许多配置选项
Bash 样式的补全
Bash/zsh 样式的交互式历史记录搜索
Emacs yank/kill ring
基于 PowerShell Token 的单词移动和删除
撤销/重做
自动保存历史记录，包括在实时会话中共享历史记录
菜单补全、Intellisense
目前发布了 2.1.0-beta1 版本，为了能够达到本文中的体验，需要使用 2.1.0-beta1 或者以上版本。

[PSReadLine](https://link.zhihu.com/?target=https%3A//github.com/PowerShell/PSReadLine)

# oh-my-posh & posh-git

类似于 oh-my-zsh，oh-my-posh 为 PowerShell 提供了很多自定义主题和配色，而 posh-git 为 PowerShell 提供了 git 状态显示和命令补全等。

[oh-my-posh](https://link.zhihu.com/?target=https%3A//github.com/JanDeDobbeleer/oh-my-posh)

oh-my-posh 建议用 2.x 版本，2.x 版本是 PowerShell 写的，但是 3.0 开始该插件换成用 Golang 实现了，不仅体积大了十几 mb，样式还变得更难看，速度也更慢了。（其实这里想吐槽一下，用 Go 去写 PowerShell 脚本的插件怕不是作者脑子进水了。作者觉得用 Go 写的话 oh-my-posh 就可以给其他 shell 用了，但问题在于 bash 和 zsh 用户都有自己的 oh-my-* 实现，犯不着去用 oh-my-posh）

Get Started
1.安装 PowerShell Core
2.运行 `pwsh`启动 PowerShell Core，然后安装以下模块：

```
Install-Module -Name PSReadLine -AllowPrerelease -Force # PSReadLine
Install-Module posh-git -Scope CurrentUser # posh-git
Install-Module oh-my-posh -Scope CurrentUser # oh-my-posh
```

3. 编辑文件 $Profile，这个文件类似于 ~/.zshrc，会在 PowerShell 启动的时候自动执行，因此我们在这个文件中加载我们所需的模块，设置相关主题：

在 Windows 上：

    notepad.exe $Profile

在 *inx 上：

    nano $Profile

在里面添加以下内容：

```powershell
Import-Module posh-git # 引入 posh-git
Import-Module oh-my-posh # 引入 oh-my-posh

Set-Theme Paradox # 设置主题为 Paradox

Set-PSReadLineOption -PredictionSource History # 设置预测文本来源为历史记录
 
Set-PSReadlineKeyHandler -Key Tab -Function Complete # 设置 Tab 键补全
Set-PSReadLineKeyHandler -Key "Ctrl+d" -Function MenuComplete # 设置 Ctrl+d 为菜单补全和 Intellisense
Set-PSReadLineKeyHandler -Key "Ctrl+z" -Function Undo # 设置 Ctrl+z 为撤销
Set-PSReadLineKeyHandler -Key UpArrow -Function HistorySearchBackward # 设置向上键为后向搜索历史记录
Set-PSReadLineKeyHandler -Key DownArrow -Function HistorySearchForward # 设置向下键为前向搜索历史纪录
```

当然，上述主题、快捷键和对应的功能是完全可以自己修改的，设置好后保存，重启 PowerShell 加载配置。

4. 安装一个 PL 字体，这里推荐 [CascadiaPL](https://link.zhihu.com/?target=https%3A//github.com/microsoft/cascadia-code/releases)。

## 看看效果

## 默认状态

![A1-1](/images/Windows_Powershell_2022_04_03_A1/A1-1.jpg)

## 根据历史记录补全

![A1-2](/images/Windows_Powershell_2022_04_03_A1/A1-2.jpg)

## Tab 补全

![A1-3](/images/Windows_Powershell_2022_04_03_A1/A1-3.jpg)

## 菜单补全 & Intellisense：

![A1-4](/images/Windows_Powershell_2022_04_03_A1/A1-4.jpg)

![A1-5](/images/Windows_Powershell_2022_04_03_A1/A1-5.jpg)

当然，对于 `Get-ChildItem`的 **alias** `ls`也没问题：

![A1-6](/images/Windows_Powershell_2022_04_03_A1/A1-6.jpg)

## 历史记录搜索：

![A1-7](/images/Windows_Powershell_2022_04_03_A1/A1-7.jpg)

![A1-8](/images/Windows_Powershell_2022_04_03_A1/A1-8.jpg)

## Git 状态指示

![A1-9](/images/Windows_Powershell_2022_04_03_A1/A1-9.jpg)

## 结语

舒服！
