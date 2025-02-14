---
layout: blog
title: WSL2配置SSH局域网访问
categories: 环境搭建
date: 2023-03-22 11:37:17
---

## 配置思路

- 和普通Linux一样，WSL也是通过OpenSSH配置SSH server
- 但是WSL是Windows的子系统，同一个局域网内的其他主机无法访问，所以要在Windows上面配置端口映射

参考：

[wsl2 远程登陆ssh__刘文凯_的博客-CSDN博客](https://blog.csdn.net/qq_24211837/article/details/117386077)

[Windows11，银河麒麟：如何打开端口_麒麟系统开放端口_全栈开发与测试的博客-CSDN博客](https://blog.csdn.net/weixin_42727710/article/details/122495314)

## 配置过程

### 配置OpenSSH

过程比较基础，不再赘述，/etc/ssh/sshd_config如下所示：

![image-20230322162125318](/images/wsl-ssh/image-20230322162125318.png)

打开以后在Windows powershell中输入：

``` shell
ssh username@localhost -p 2111
```

即可在本地Windows系统访问WSL的SSH服务

### 配置端口映射

在Windows中使用netsh配置：

``` shell
netsh interface portproxy set v4tov4 listenport=2233 listenaddress=0.0.0.0 connectport=2111 connectaddress={ip_wsl}
```

前面的port和ip表示Windows监听的ip和端口，后面的是wsl的ip和端口

此外，还要再Windows防火墙中打开2233端口，请参考第二篇博客
