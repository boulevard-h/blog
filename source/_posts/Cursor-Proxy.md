---
layout: blog
title: Cursor-Proxy
date: 2025-08-28 16:53:04
categories: 环境搭建
---

# Cursor绕过地区限制

~~最近~~（一两个月前），Cursor限制了某些地区的模型使用，Claude-4-sonnet等模型都会显示当前地区不可用，找到了三种解决方法。

## 1. HTTP1.1 + Cursor内设置Proxy

进入Cursor设置-Network，把HTTP Compatibility Mode改为HTTP/1.1

然后进入General-Editor Settings，把Http: Proxy设置为你的Http代理地址

这样的缺点是HTTP/1.1模式下很容易出现卡顿断流的情况

## 2. 全局代理

最简单粗暴的方式，直接在Clash中设置全局代理（TUN模式）

缺点也显而易见，全局代理可能会导致访问墙内软件/网站较慢

## 3. Proxifier

为软件/目标域名指定代理的插件，这类软件很多，类似的还有FreeCap等，但是只有Proxifier近期还在更新，且支持Mac和Win~~（而且网上一搜一大把激活码~~

理论上很简单，只需要添加你的代理地址，然后在Rules中选择Cursor APP即可：

<img src="/images/Cursor-Proxy/image-20250828170026517.png" alt="image-20250828170026517" style="zoom:50%;" />

<img src="/images/Cursor-Proxy/image-20250828170055502.png" alt="image-20250828170055502" style="zoom:50%;" />

但是不知道为什么，这样设置以后，还是代理不到Cursor。

于是尝试把Default代理规则也切换成走Clash，结果就代理上了：

<img src="/images/Cursor-Proxy/image-20250828170233334.png" alt="image-20250828170233334" style="zoom:50%;" />

但是这样就和梯子开全局一样，一不小心把微信也给代理了；但是从开了Default以后的代理信息，可以看出问题来：之前软件自动识别的Cursor信息有误，真正的App名称应该是：Cursor Helper (Plugin)，代理的网址是 `*.cursor.sh`

在Rules中加上正确的规则，在把Default规则设置会Direct，就能成功代理了：

<img src="/images/Cursor-Proxy/image-20250828170452741.png" alt="image-20250828170452741" style="zoom:50%;" />
