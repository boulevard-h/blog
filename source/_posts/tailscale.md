---
layout: blog
title: 'tailscale: 在你的远程设备之间搭建局域网'
categories: 瞎折腾
date: 2024-06-18 10:48:47
---

## 前言

设想如何在外部远程连接你在校园网或实验室内网中的设备：

- 采用 ToDesk 等远程桌面软件，缺点是卡、清晰度低、而且在 Linux 系统中的兼容性做的不是很好，容易崩溃。
- 采用系统自带的远程软件如：SSH、RDP、VNC。

后者的性能和稳定性更好，但是一般都是直接通过 `ip:port` 连接。而对于局域网（如校园网、实验室内网）内的设备，是没有公网 IP 的，只能在局域网的出口路由处做端口映射，比较麻烦。

而已知解决办法是通过 FTP 进行内网穿透，将你的内网设备映射到一个外部 IP 例如 `your-id.ftp.com`，但是在很多学校中，这种内网直接映射公网 IP 的方法是非法的，有潜在的威胁。

而本文介绍的 tailscale 则会在你的设备之间组件个人虚拟局域网，而不是直接把你的设备暴露在公网上，更加安全，且可以达到同样的效果。

而要配置 tailscale，其实流程非常简单，在你的不同设备上安装 tailscale 软件，然后登陆同一个账号就可以互相访问了。但比较恶心的是这个软件非常难装，尤其是在 windows 上面。

## Windows 安装

当你访问 tailscale 官方下载页下载 Windows 安装包的时候，会下载一个几百 kb 的 exe 包来安装，这个安装过程非常缓慢，而且大概率进行到一半报错失败了。下面是一个成功率更高的解决办法：

1. 关闭防火墙&杀毒软件，**这个很重要！！！**
2. 在官方下载页下载 msi 安装包，大概有二十多 mb，可以有效防止 exe 包安装的时候下载到一半失败了。

​		[pkgs.tailscale.com/stable/#windows](https://pkgs.tailscale.com/stable/#windows)

​		一般的 64 位 PC 下载 `tailscale-setup-x.y.z-amd64.msi` 即可。

然后，从 msi 安装，打开软件登录即可。

## Ubuntu 安装

官方给的安装方法也是运行一个脚本，然后和 windows 一样，脚本运行到一半容易因为网络问题报错。所以，直接下载其 deb 包安装：

```
wget https://pkgs.tailscale.com/stable/ubuntu/pool/tailscale_x.y.z_amd64.deb
sudo dpkt -i tailscale_x.y.z_amd64.deb
```

## 使用

都安装和登陆好以后，就可以互相 ping 同对方的虚拟地址了，然后就可以直接通过 IP 进行 SSH、VNC 等连接了。
