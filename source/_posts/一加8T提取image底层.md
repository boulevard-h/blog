---
title: 一加8T提取image底包 & namelssOS刷氧OS13踩坑
date: 2022-12-24 08:50:26
categories: 搞机
tag: [安卓刷机]
---

## 前言

之前因为不满ColorOS的臃肿刷入了原生的nameless A13：[一加8T刷入namelessOS | Boulevard's Blog (blog-boulevard.top)](https://blog-boulevard.top/2022/12/19/一加8t刷入namelessos/)

但是nameless的功能实在太少，bug实在太多，堪比从MI10搬运过来的MIUI13包。所以继续折腾一下，从nameless A13刷入到大氢的亲兄弟——国外测试已经更新到了安卓13的OxygenOS

**附：一加8系，9R，9RT，10R的OxygenOS13 Full OTA包地址**[Stable Android 13 update rolls out to the OnePlus 8 series and more (xda-developers.com)](https://www.xda-developers.com/oneplus-8-series-9r-9rt-10r-oxygenos-13-stable/)

## 提取images

一加8T的底层可能比较多样化（低情商：混乱）。刷系统ROM包之前，如果你搞不清现在的系统底层和目标系统的底层是否是一致的，最好先刷入目标系统的四个img文件`boot.img,vbmeta.img,vbmeta_system.img,recovery.img`

良心的安装包提供者会把这四个文件的下载连接也给你，当然如果他没给的话，也可以从Full OTA包里面提取出来，只需要**payload dumper**这个工具。

首先下载payload dumper，这个工具有Go和Py的源码可以自己build，但是懒人还是选择大佬做好了的exe包，点击即可使用：[tool/payload.bin解包工具/payload dumper_go_大侠阿木重制 - 其他文件 - 一加手机官方ROM下载 (daxiaamu.com)](https://yun.daxiaamu.com/files/tool/payload.bin解包工具/payload dumper_go_大侠阿木重制/)

下载好以后，解压Full OTA的zip包，可以看到一个payload.bin文件，把这个文件图标拖动到payload dumper图标上就会自动出现一个终端黑框，开始提取。提取完毕以后，会在payload.bin同目录下出现一个extraced_xxx文件夹，点进去就可以看到很多img文件，从中找到我们需要的四个image即可。

![image-20221224092710542](/images/8T/image-20221224092710542.png)

## 补充

虽然提取img是成功的，但是从一个第三方刷到OOS13对我来说还是有点困难了，在刷入四个image以后再从recovery sideload失败变砖了

然后就只好开始9008救砖会H2OS，再从H2OS卡刷回OOS
