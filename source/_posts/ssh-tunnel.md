---
layout: blog
title: 使用SSH反向隧道实现公网IP连接内网SSH服务器
date: 2025-02-15 14:48:38
categories: 环境搭建
---

## 0. 前言

在使用 SSH 连接内网服务器的时候，由于缺乏公网 IP，只有在同一个内网的服务器可以访问，而且我们的内网 IP 是动态变化的，即使关闭 DHIP 设置固定 IP 也无济于事，很麻烦，所以用一台白嫖的阿里云 2C2G 服务器做中转

**注：把 SSH 端口暴露在公网一定要注意安全，记得使用仅公钥认证登陆等手段**

## 1. 配置过程

首先要在公网和内网服务器都配置好 SSH，保证自己的电脑可以连接

在公网 SSH 服务器上配置转发：

- 在 `/etc/ssh/sshd_config` 中添加：

  ```bash
  GatewayPorts yes    # 允许外部访问转发端口
  TCPKeepAlive yes    # 保持TCP连接
  ClientAliveInterval 60
  ```

- 重启 SSH 服务：

  ```bash
  sudo systemctl restart sshd
  ```

- 在阿里云防火墙上打开 2222 端口

在内网 SSH 服务器上打开反向隧道：

- `sudo apt install autossh` 安装 autossh

- 建立隧道

  ```bash
  autossh -M 0 -N -o "ServerAliveInterval 30" -o "ServerAliveCountMax 3" -R 2222:localhost:22 user@云服务器IP
  ```

然后就可以通过转发连接了：

``` bash
ssh -p 2222 内网user@云服务器IP
```

autossh 的窗口需要一直挂着很烦，也可以用 nohup 等在后台执行
