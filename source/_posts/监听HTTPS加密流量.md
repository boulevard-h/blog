---
layout: blog
title: 监听HTTPS加密流量
categories: 不务正业の小玩意
date: 2023-04-23 20:15:05
tags: [抓包, HTTPS]
---

## 安装Charles抓包软件

[网络抓包工具Charles 4.5.6 中文版(Windows便携免安装) - 『逆向资源区』 - 吾爱破解 - LCG - LSG |安卓破解|病毒分析|www.52pojie.cn](https://www.52pojie.cn/thread-1600964-1-1.html)

安装破解即可

## 启用SSL代理

### 安装根证书

`帮助 > SSL代理 > 安装Charles根证书`：

点击安装证书（I）：

![image-20230423201826079](/images/charlesHTTPS/image-20230423201826079.png)

选择当前计算机，下一页，将证书存储到受信任的根证书颁发机构：

![image-20230423201921773](/images/charlesHTTPS/image-20230423201921773.png)

安装完成

### 安装浏览器证书

`帮助 > SSL代理 > 在移动设备或远程浏览器上安装Charles根证书`，会弹出一个保存页面，随便找个位置保存。

浏览器设置中搜索“证书”，进入证书添加页面加入保存的证书即可。

## 测试HTTPS抓包

`代理 > SSL代理设置 > 包括`加入一条`*:*`

然后开启SSL，访问一个HTTPS网页，发现可以抓包并解密：

![image-20230423202307163](/images/charlesHTTPS/image-20230423202307163.png)

