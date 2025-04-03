---
title: SourceTree中使用SSH推送失败的问题分析与修正
date: 2024-02-05 23:14:43
tags:
    - SSH
    - Git
category: 个人随笔
---
## 问题描述
之前使用SSH密钥用Git命令行推送Github博客，后来突然发现Sourcetree推送时，推送页面内会有提示“是否信任公钥”。由于无法输入是和否，无法进行推送。最后导致推送失败。而Git命令行推送结果一切正常。
## 分析
首先怀疑的目标是Sourcetree并没有读取我Git使用的密钥导致的。点击Sourcetree的选项发现：![{A88A1362-99DC-49d9-9D96-FFDBF32910FD}](https://pic.firehomework.top/2025/03/b4503217a6e2ab5b16d46a44602ade63.webp)

我印象中从来没有见过这个东西。在之前的学习中，使用的都是OpenSSH。网上搜索了一下，确实可以通过直接修改这里的客户端为OpenSSH来解决问题。但既然遇到了不妨一起研究一下。（而且注意到Mercurial总是使用Plink on Windows，或许学习了也有好处）

### PuTTY

根据网上的各种资料：Putty 是一款骨灰级软件，历史可以追溯到上世纪 90 年代后期，目前已成为系统运维的必备工具之一。是一款集成[虚拟终端](https://zh.wikipedia.org/wiki/虚拟终端)、[系统控制台](https://zh.wikipedia.org/w/index.php?title=系统控制台&action=edit&redlink=1)和网络文件传输为一体的[自由及开放源代码](https://zh.wikipedia.org/wiki/自由及开放源代码软件)的程序。它支持多种网络协议，包括[SCP](https://zh.wikipedia.org/wiki/安全复制)，[SSH](https://zh.wikipedia.org/wiki/Secure_Shell)，[Telnet](https://zh.wikipedia.org/wiki/Telnet)，[rlogin](https://zh.wikipedia.org/w/index.php?title=Rlogin&action=edit&redlink=1)和原始的套接字连接。它也可以连接到[串行端口](https://zh.wikipedia.org/wiki/串行端口)。其软件名字“PuTTY”没有特殊含义。

它应该算是一个SSH的客户端，和OpenSSH的客户端不同，它是老东西了。不过作者现在还在开发。

### 解决方案

既然是另外的客户端，那么思路就是把我的私钥导入到客户端里。由于Github已经持有我的公钥，再让他信任Github的公钥，即可大功告成

选择：

![image-20240206093639525](https://pic.firehomework.top/2025/03/0ff9f391feaf5088f9653bfdfbb02d6f.webp)

然后选择：

![image-20240206093700595](https://pic.firehomework.top/2025/03/80f02c6f21fecf12604677702cbb0f99.webp)

导入我在SSH中的私钥

![image-20240206093725263](https://pic.firehomework.top/2025/03/7caf11714c5ab90ff4d047ec8425e6f2.webp)

此时便导入了私钥，接下来获取公钥。我没找到图形化的操作应该在哪儿设置公钥，根据

https://blog.51cto.com/u_16305940/9047912的第二种方案：

> 找到putty路径，默认在： SourceTree\app-版本号\tools\putty
>
> 进入这个路径在此处打开命令行，运行：`plink git@github.com`即可。

此时大功告成，以后也可以以图形化的界面来控制了。
