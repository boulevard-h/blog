---
layout: blog
title: QMK 固件降低延迟
date: 2024-10-11 17:08:47
categories: 客制化键盘
---

平常打游戏的键盘用的是 kbdfans 的 kbd8x_mk2，一测延迟有 10ms，刚好又看到 qmk 仓库中有这把键盘的源码，于是想着修改一下参数，降低延迟。

在 config.h 中这些参数可能影响延迟：

- USB_POLLING_INTERVAL_MS 轮询率，默认 1 ms，所以也不是瓶颈所在
- MATRIX_IO_DELAY 矩阵扫描率，默认 30 us，更加没问题了
- DEBOUNCE 消抖时间，默认 5 ms，关系很大，根据轴体素质可以适当降低。我用的力驰的轴，素质比较高，测试了设置为 0 小键都不会有问题，但是空格会双击，最后调节到了 2 ms

此外，还尝试了编译时加上 OPT=2 等，最后发现还是消抖最有用，可能是这把键盘使用的是垃圾 Atmega32U4，性能太差了，导致有些优化看不出来。

对于 STM32F401 等 MCU，可以参考这个开源工程做到 8K：

- [QMK键盘延迟测试 以及1k与8khz回报率延迟对比_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1nA411d77x/?spm_id_from=333.1007.top_right_bar_window_custom_collection.content.click)
- [qmk_firmware/keyboards/akeypad at qmk_master_build_2022q4 · luantty2/qmk_firmware (github.com)](https://github.com/luantty2/qmk_firmware/tree/qmk_master_build_2022q4/keyboards/akeypad)
- [luantty2/akeypad (github.com)](https://github.com/luantty2/akeypad)

