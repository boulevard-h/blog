---
layout: blog
title: 家庭Mesh组网（一）—— Mesh 介绍
date: 2024-08-06 23:07:02
categories: 不务正业の小玩意
tags: [网络, Mesh, 无线]
---

家里的老华硕 [RT-AC66U](https://www.asus.com.cn/networking-iot-servers/wifi-routers/asus-wifi-routers/rt-ac66u-b1/) 在辛苦工作了五年后终于开始出问题了，WiFi 信号一天消失不见好几次，刚好电信的百兆宽带也要到期了，准备换成移动手机送的千兆。于是打算顺便把家里的路由器一换，组成现在流行的多路由器 Mesh 组网方案。

## 1. 什么是 Mesh？为什么要用 Mesh？

参考：[2024年新版 路由器Mesh组网全攻略（网络拓扑方案、装修网线预埋方案） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/343117525)

在家庭无线网络中，由于水泥墙等阻隔，一个路由器产生的 WiFi 信号很难覆盖全家，在一些死角处信号会很差。一个常见的误区是：购买价格昂贵的高端路由器来获得超强的穿墙能力，从而覆盖全家。因为相关标准对无线发射功率做了限制，最好的路由器也无法发出足够穿墙的信号，除非是类似华硕改地区为澳大利亚（澳洲对无线发射功率的限制较为宽松）等骚操作来进行。

与其花大几百乃至上千买一个不一定有用的高端路由器，不如多买两个普通路由器，放在不同的房间里面来覆盖信号死角。

而单纯放置多个路由器的坏处就是在一个家中有好几个不同的 WiFi，走到不同的地方要切换到信号好的一个。而 Mesh 组网可以让多个路由器共享一个 WiFi 名称（SSID），在用户的感知上，全屋只有一个 WiFi。

Mesh 分为有线 Mesh 和无线 Mesh 两种方式，有线 Mesh 的路由器之间采用有线进行连接，而无线 Mesh 的路由器之间使用无线信号传输通信。很明显，有线 Mesh 更加稳定高效，但布局也比无线困难。下面介绍有线 Mesh 的几种常见拓扑：

## 2. 传统有线 Mesh

![image-20240806234856308](/images/Home-Mesh-1/image-20240806234856308.png)

在最传统的 Mesh 拓扑结构中，主路由的 WAN 口连接光猫，而其他子路由的 WAN 口连接主路由的 LAN 口。

**注：**WAN 口是路由器网线入口，LAN 口是路由器网线出口。

在传统的家庭网线布局中，光猫往往位于**弱电箱**，也就是一个墙内的铁皮小箱子。箱子中有多条网线连接至不同房间的网口。因此，**主路由**往往需要**也放置在弱电箱**中，导致主路由的**信号被弱电箱削减**很多，也**不容易散热**，且很多弱电箱**放不下**较好的主路由。

另一种方式是把主路由放在弱电箱之外，在弱电箱中再放一个**交换机**，把主路由的 LAN 口和子路由的 WAN 口连接到交换机上。

交换机是很便宜的，做起来不难。这样做最大的难点是**主路由需要有两根网线连接到弱电箱**，一根负责连接光猫与 WAN 口，一根负责连接 LAN 口与交换机。而大多数家庭装修的时候，都只给一个房间配备了一根网线。

![img](/images/Home-Mesh-1/v2-f29c72cec154b87a4af078255ea82333_r.jpg)

但是这种在弱电箱中放置交换机，而把主路由放在外面的方法是好的，只不过需要一对 **VLAN 交换机**来进行**单线复用**。VLAN 虚拟局域网可以通过标签（tag）讲一个物理 LAN 隔离为多个虚拟 LAN。

位于弱电箱的 VLAN 交换机和主路由器旁的 VLAN 交换机只需要一根网线连接，但是却能够划分出两个 VLAN 来，图中的 VLAN2 用于连接光猫和主路由 WAN 口，VLAN3 用于连接主路由的 LAN 口和子路由的 WAN 口。

![img](/images/Home-Mesh-1/v2-02d1ba50ba96059fe345de8790661f25_r.jpg)

## 3. AP Mesh

上述三种方法，要么浪费一个主路由，要么需要两个网线，要么需要复杂的 VLAN，来进行路由之间的串联，那么如果把路由之间并联，也就是每一个路由器都直接连接到光猫，能不能行得通呢？

答案是可以的，现在很多新路由器支持 AP Mesh 方式，来避免复杂的布线。

但是这样，又有一个缺点：路由器之间的 Mesh 数据交换都来到了光猫上，运营商送的小光猫很可能受不住负载，导致网络出问题。一些万兆光猫如中兴 g7615 系列可以胜任 AP Mesh 的数据交换，但是换光猫又是一项成本和复杂度很高的任务。

![img](/images/Home-Mesh-1/v2-18116bfe8d841f4dd76702b5bb04553b_r.jpg)

另一个方式是在光猫后面接一个软路由来分担光猫**拨号上网**的任务，但是这玩意好像又是一个新的大坑了...

![img](/images/Home-Mesh-1/v2-b8618e750faf6b8d3482274e22cd9d44_720w.webp)