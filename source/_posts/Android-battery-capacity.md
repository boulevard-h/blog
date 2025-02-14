---
title: 安卓查看电池寿命
date: 2022-12-26 17:22:16
categories: 搞机
---

## 原理

查看系统内的uevent文件可以查找到电池容量

这个文件，按照网上大多数的说法，在：`/sys/class/power_supply/bms/uevent`，不过我的路径略有出入，建议进入到`/sys/class/power_supply`以后自己找一找

## 读取uevent文件

那么难点就在怎么获得这个文件了

### 方法一：termux（需要root）

#### 安装termux

在此下载：[Releases · termux/termux-app (github.com)](https://github.com/termux/termux-app/releases)

里面有很多版本，需要下载和你的手机CPU架构一致的。所以需要查看CPU架构，方法可以选择：

- 搜索你的CPU型号是什么架构
- 使用ADB命令：`adb shell getprop ro.product.cpu.abi`

我的手机CPU是865，ADB返回的架构是arm64-v8a

所以下载后面标注arm64-v8a的apk包

#### 获取文件

安装好以后，termux就等于在你的手机里面运行了一个Linux Shell，直接cat就好

不过我cat不到，估计是没有root导致的

### 方法二：adb（还是需要root）

同样的，adb其实也是Linux shell，能访问的话cat就好，但是普通的adb是没有root权限的。

adb root的方法有两种，第一个是直接 adb root访问，第二个是装adbd insecure
