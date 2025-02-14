---
title: 安卓卸载自带应用
date: 2022-12-25 10:41:06
categories: 搞机
---

## 前置准备

电脑：ADB工具

手机：ES文件浏览器、打开USB调试

## 查找包名

在`ES文件浏览器 -> 应用 -> 系统应用`中找到想要卸载的应用，点开的属性中回显示包名

比如这里我想卸载的是Oxygen13自带的Netflix，其包名为`com.netflix.mediaclient`

## 删除

在输入`adb devices`可以找到设备以后，输入`adb shell`

然后输入`pm uninstall --user 0 包名`即可卸载
