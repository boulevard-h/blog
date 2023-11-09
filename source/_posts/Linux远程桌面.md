---
layout: blog
title: Linux远程桌面
date: 2023-11-09 16:31:43
tags: [Linux]
---

最近因为跑蓝牙 Fuzz 的原因，想要不在实验室的时候也能够远程看一下代码的运行情况，做出修改。所以需要配置一个远程环境。

单纯的配一个 Linux 远程环境是很简单的，ToDesk 软件就官方支持 Linux；但还有一个需求——重启，重启以后需要自动进行：

- 选择规定的内核
- 自动进入桌面
- 自动运行 ToDesk

那么设置步骤如下：

## 一、选择默认内核

编辑 grub 的默认选项：

例如，当我进入系统的时候，第一个界面是：

> Ubuntu
>
> Advanced Options for Ubuntu
>
> Windows Boot Manager

在选择第二个选项以后，再次选择：

> Ubuntu, with Linux 6.5.0
>
> Ubuntu, with Linux 6.5.0 (recovery mode)
>
> Ubuntu, with Linux 6.2.0
>
> Ubuntu, with Linux 6.2.0 (recovery mode)

这里我想选择 6.2 的内核，那么选择的顺序是：第2个->第3个

就需要在 /etc/default/grub 中设置 `GRUB_DEFAULT="1> 2"`

注意这里的选项编号是从 0 开始的，所以应该是 "1> 2" 而不是 "2> 3"

然后 `sudo update-grub` 以后重启就可以默认进入 6.2 内核了。

## 二、自动进入桌面

打开设置，找到 "Users -> Automatic Login" 这样开机以后就可以自己进入桌面。否则的话，即使 ToDesk 连上也无法显示任何东西。

## 三、自动启动ToDesk

在 ToDesk 中设置开机启动，并且设置安全密码。因为重启以后临时密码会变（即使设置成manual重置密码也会变）。

同时，如果设置了代理的话，代理软件最好也提前打开防止上不了网，从而导致连不上。
