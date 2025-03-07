---
title: amx40新固件刷机教程
date: 2022-09-23 16:04:14
categories: 客制化键盘
---

## 更新说明

固件蓝牙发射功率增加了4dBm，减少断连可能性（但是介于使用环境不同，不保证完全不断连）

## 更新固件教程

### 下载群文件

下载群文件中的**amx40刷机文件.zip**，解压，有如下两个文件：

![image-20220923152342798](/images/amx40firmware/image-20220923152342798.png)

其中**烧录工具箱-1.2.2.0.7z**中是烧录固件的工具，**nrf52_kbd_with_sd.hex**是新固件。

### 安装烧录工具箱

解压**烧录工具箱-1.2.2.0.7z**，进入解压后的文件夹是这样的：

![image-20220923152530993](/images/amx40firmware/image-20220923152530993.png)

首先点击**首次使用点我安装驱动.EXE**，进入这个界面：

![image-20220923152608293](/images/amx40firmware/image-20220923152608293.png)

点击安装，过几秒出现：

![image-20220923152633738](/images/amx40firmware/image-20220923152633738.png)

然后关掉窗口即可。

### 刷写固件

点击烧录工具箱中的**wch_nrf_burner.exe**

![image-20220923152727227](/images/amx40firmware/image-20220923152727227.png)

**有线**连接amx40键盘，在当前设备一行的最右边点击刷新（最右边那个小圈圈），过几秒钟后，就可以在当前设备中看到这两个了（点击最右边那个倒三角形）

![image-20220923152951204](/images/amx40firmware/image-20220923152951204.png)

然后选择其中的一个（CMSIS-DAP和Lotlab Configurator都可）

再在蓝牙固件中，选择amx40刷机文件中的**nrf52_kbd_with_sd.hex**

![image-20220923153250363](/images/amx40firmware/image-20220923153250363.png)

USB固件和下面的两个选项都不要管，选择好设备和蓝牙固件以后直接点击**烧录**

然后稍微等待几十秒，下面出现这一堆英文提示，左下角显示准备就绪以后就刷好了：

![image-20220923153426919](/images/amx40firmware/image-20220923153426919.png)

然后断开USB，重新插入即可

## （特别注意）关于刷机以后键位的问题

刷机以后**键位有可能被重置**了，导致出现一些问题，比如你原来设置的蓝牙设备连接键位被改变了。

参考**群文件>爱丽丝40>AMX40_配置网页改建服务教程.pdf**改成你习惯的键位。

