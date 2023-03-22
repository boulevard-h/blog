---
layout: blog
title: ssh环境变量缺失问题解决
categories: 不务正业の小玩意
date: 2023-03-22 17:13:26
tags: [wsl, ssh]
---

## 参考

[解决SSH远程执行命令找不到环境变量的问题 - 振宇要低调 - 博客园 (cnblogs.com)](https://www.cnblogs.com/zhenyuyaodidiao/p/9287497.html)

## 原因

在Linux Shell登录的时候会执行`/etc/profile`文件，但是ssh登入会执行`~/.bashrc`文件

## 解决

在`~./bashrc`中加入环境变量

方法为，添加一行：

```shell
export PATH="路径:$PATH"
```

保存后执行`source /etc/profile`即可。
