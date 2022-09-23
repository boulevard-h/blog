---
title: LotLab固件编译教程
date: 2022-09-23 16:04:14
catagories: 客制化键盘
tags: [客制化键盘, 简易教程]
---

之前在Win和Linux上尝试配置了好多次LotLab蓝牙键盘的固件编译环，每次都寄了，后来发现有Docker环境...

直接docker run lotlab/nrf52-keyboard即可

把你的项目文件夹（包含config.h，keymap_common.h等）放入容器的/work/keyboard目录即可，然后进入项目文件夹输入` make -j`即可，注意可用的蓝牙文件是nrf52_keyboard_with_sd.hex
