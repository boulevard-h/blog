---
title: 一加8T刷入namelessOS
date: 2022-12-19 11:04:55
categories: 搞机
---

## 准备工作

- 一台一加8/9手机（废话）

  - 本教程使用一加8T、ColorOS13

- [Download | Nameless AOSP](https://nameless.wiki/category/download)下载如下软件：

  - 最新刷机包：

    ![image-20221219110803199](/images/oneplusnameless/image-20221219110803199.png)

  - 四个image镜像：

    ![image-20221219110826171](/images/oneplusnameless/image-20221219110826171.png)

- [For OnePlus 8/8 Pro & 8T/9R | Nameless AOSP](https://nameless.wiki/getting-started/install/for_8_9R)下载工具：

  - platform-tools：

    ![image-20221219111008416](/images/oneplusnameless/image-20221219111008416.png)

  - Google USB driver：

    ![image-20221219111028900](/images/oneplusnameless/image-20221219111028900.png)

## 系统内解锁

下面的操作在手机系统内完成：

### 解锁开发者模式

设置-关于本机-版本信息，快速连续点击十次版本号，然后就会提示输入锁屏密码进入开发者模式

![50e68c266b5449ff106a54535127cf3](/images/oneplusnameless/50e68c266b5449ff106a54535127cf3.jpg)

### 开启USB调试与OEM解锁

打开了开发者模式中，可以在设置-系统设置中找到开发者选项，开启其中的OEM解锁和USB调试：

![7cda331d6b562b1b9ad338d2ff14b6c](/images/oneplusnameless/7cda331d6b562b1b9ad338d2ff14b6c.jpg)

## 解锁Bootloader

**注意：解锁BL会格式化手机、做好备份**

手机开机状态下连接电脑，会弹出USB调试信息，同意即可。

在之前下载的platform-tools文件夹打开命令行，输入：

``` powershell
./adb.exe devices
```

就可以看到设备了：

![image-20221219112346386](/images/oneplusnameless/image-20221219112346386.png)

然后输入：

``` powershell
./adb.exe reboot bootloader
```

这时手机会自动重启到这个界面（图是网上随便找的）：

![image-20221219112200025](/images/oneplusnameless/image-20221219112200025.png)

然后输入：

``` powershell
./fastboot.exe flashing unlock
```

你的手机会提示你是否要解锁bl：

![image-20221219112628704](/images/oneplusnameless/image-20221219112628704.png)

按音量上下可以选择，选第二个，然后按电源键确认。

然后手机格式化并解锁BL，等待一会以后，重新进入系统。

由于系统被格式化了，所以需要重复第一步打开USB调试和OEM解锁。

## 刷入四个image

在plateform-tools目录下使用cmd（注意是cmd命令提示符，而不是windows powershell）

输入：

``` shell
adb.exe reboot fastboot
```

手机会进入fastboot模式：

![image-20221219115434920](/images/oneplusnameless/image-20221219115434920.png)

把四个img文件放入到platform-tools/images文件夹，然后依次执行：

``` shell
fastboot flash boot images/boot.img
```

``` shell
fastboot flash --disable-verity --disable-verification vbmeta images/vbmeta.img
```

``` shell
fastboot flash --disable-verity --disable-verification vbmeta_system images/vbmeta_system.img
```

``` shell
fastboot flash recovery images/recovery.img
```

![image-20221219113849951](/images/oneplusnameless/image-20221219113849951.png)

## 刷入系统

上一步四个image都刷入完毕以后，在手机上选择：高级-重启到恢复模式：

然后如图所示手机会进入恢复模式（注意恢复模式是可以直接触屏操作的哦）：

![image-20221219115529504](/images/oneplusnameless/image-20221219115529504.png)

选择Install update - ADB Sideload

然后把刷机包的zip文件也放到platform-tools文件夹，cmd输入：

``` shell
adb sideload 刷机包名字.zip
```

然后你的手机就会开始刷入新系统了...

![image-20221219114930221](/images/oneplusnameless/image-20221219114930221.png)

## 刷入完毕

上面的windows进度条会有一些bug，刷入完成后进度条大概只在50%左右

所以以手机显示的进度为准，完成时如下图，手机最下面会有一个Step2/2：

![image-20221219115635311](/images/oneplusnameless/image-20221219115635311.png)

然后点击左上角的返回键，回到recovery首页，选择"Factory reset"->"Format data/factory reset"->"Format data"

然后Format data结束以后，返回到recovery首页点击reboot to system就可以进入新系统了。

![image-20221219115943950](/images/oneplusnameless/image-20221219115943950.png)
