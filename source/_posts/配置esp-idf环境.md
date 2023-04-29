---
layout: blog
title: 配置esp-idf环境
date: 2023-04-25 21:07:17
tags:
---

## 前言

esp32开发环境主流有三种：esp-idf、micropython、arduino：

- esp-idf：官方出品，使用C语言编辑，环境比较难装，不过集成了很多库还是不错。
- micropython：没了解太多，不过python语法比C还是友好很多。
- arduino：arduino的类C开发语言，比直接用嵌入式C开发简单，而且有巨大的arduino生态支持。

虽然怎么看micropython和arduino都很香，但是无奈这次需要用到SM2椭圆曲线，看到的几个esp32上跑ecc的代码主要都在esp-idf上。本来就是刚入门，要是开发环境还和网上资料不一样的话感觉要寄...所以**最终选择了ESP-IDF**。

## （已放弃）Windows配置ESP-IDF

### 直接安装本地ESP-IDF + VSCode

[vscode-esp-idf-extension/install.md at master · espressif/vscode-esp-idf-extension · GitHub](https://github.com/espressif/vscode-esp-idf-extension/blob/master/docs/tutorial/install.md)

看似很简单，直接在VSCode中下载插件，插件可以自动安装好环境（当然也可以先安装ESP-IDF，然后到VSCode中识别已有环境），然后就可以在VSCode里面进行编译/调试。

但是实际上ESP-IDF本并不适合在Windows本地装，编译速度比Linux慢5倍左右且不说。在Windows本地安装了好几遍不同版本的IDF，到最后编译HelloWorld工程的时候都会编译失败。

上网一查，很多人都遇到这个问题，可能是因为IDF的主程序是IDF.py，而Windows上面之前配过的python环境产生了一些冲突。在全新的Windows或者Linux系统下配置IDF编译就都没问题。

故放弃Windows本地的ESP-IDF

### Docker+WSL2+VSCode Remote+WSL2串口工具

然后就看到有人说WSL2+Docker+VSCode Remote+WSL2串口工具可以很方便的配置环境。

看了一下例子，不是那么方便...不过本地实在是配不好了，就考虑去试试。

然后最脑瘫的来了^ ^：

一打开我多年未动的WSL2和Docker Desktop居然发现他们寄了。仔细一想好像是这学期初装VBox的时候把Hyper-V虚拟化关掉了，然后一顿操作打开Hyper-V复活WSL2和Docker，我的VBox与VMWare又寄了...没办法这学期要做实验，只能老老实实删了WSL2和Docker，关闭Hyper-V，配了半天恢复原样

只能说和Hyper-V沾点的东西还是少去碰...

## Ubuntu虚拟机配置ESP-IDF+VSCode

### 安装ESP-IDF

之前不是有人说在没配过Python环境的系统上随便配吗，既然到Windows上面跑是彻底泡汤了，那就直接开一个Ubuntu虚拟开装吧：

[Linux 和 macOS 平台工具链的标准设置 - ESP32 - — ESP-IDF 编程指南 v5.0.1 文档 (espressif.com)](https://docs.espressif.com/projects/esp-idf/zh_CN/v5.0.1/esp32/get-started/linux-macos-setup.html#get-started-prerequisites)

由于是全新的虚拟机环境，装的过程还是很顺利地，开个连GitHub很稳定的加速器，跟着上文一顿输入脚本就行。

需要注意的是，ESP-IDF有很多环境变量，需要运行`$HOME/esp/esp-idf/export.sh`来导入配置。由于这些环境变量实在是太多了，所以写入到`~/.bashrc`里面又不是很合适，于是可以到`~/.bashrc`中加入一个运行`export.sh`的快捷命令：

``` shell
alias get_idf='. $HOME/esp/esp-idf/export.sh'
```

这样每次打开终端，只需要输入一句`get_idf`就激活idf环境了。

### 简单使用IDF

随便搞一个工程，比如`hello_world`，进入到终端激活idf环境，首先可以设置目标芯片：

``` bash
cd ~/esp/hello_world
idf.py set-target esp32
idf.py menuconfig
```

menuconfig中会弹出来一个和电脑BIOS一样简陋的界面，可以到里面配置ESP32的一些设置，比如WIFI配置、处理器速度等等。

配置好以后，运行`idf.py build`即可编译出bin文件，总之，如果没有VSCode等IDE，就需要在终端中使用`idf.py`对工程操作。

### 安装VSCode

这个和在Windows上大差不差，安装插件，配置插件的时候选择之前安装好的IDF即可。

### 烧录ESP32

虽然是在虚拟机上面，但是实际上和直接烧录大差不差（夸一夸VMWare做的还是挺好的，不像垃圾VBox^ ^）

注：如下设置方法仅支持ESP32 DevKitC v4通过USB烧录

1. 不插入ESP32，首先在终端输入`ls /dev/tty*`
2. 插入ESP32，VMWare会提示要把USB设备放在主机还是虚拟机使用，选择虚拟机
3. 再次输入`ls /dev/tty*`，比第一次多出来的的就是你选的设备（我这里叫ttyUSB0）
4. 使用`sudo code .`打开VSCode；或者`sudo chmod 777 /dev/ttyUSB0`（这一步是为了防止没有权限访问串口）
5. 在VSCode左下角修改串口，选择步骤3中确定的串口
6. VSCode左下角第二个esp32，选择型号，我这里选择`esp32 > esp32 with USB bridge`，也就是通过USB通信的esp32开发板
7. 点击`编译-烧录-监视`（也就是一个火苗的图标）。第一次执行编译完毕以后，会让你选择烧录方式，选`UART`，然后开始烧录，按一下ESP32上面的`BOOT按键`，出现烧录进度。烧录完毕以后会自动打开监视界面。

如果你没有开发板，而想要验证程序，可以在[esp32-bin-file.ino - Wokwi Arduino and ESP32 Simulator](https://wokwi.com/projects/305457271083631168)上传bin文件仿真。



