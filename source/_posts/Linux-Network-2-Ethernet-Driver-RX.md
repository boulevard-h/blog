---
layout: blog
title: 'Linux Network 2: 以太网驱动收包'
categories: 'Linux Kernel'
date: 2024-07-01 20:35:10
---

## 参考链接

[linux网络报文接收发送浅析_netif tx netif rx-CSDN博客](https://blog.csdn.net/wangquan1992/article/details/112129162)

[链路层和网络层的接口 （linux网络子系统学习 第五节 ） _51CTO博客_linux链路层编程](https://blog.51cto.com/yaoyang/1269713)

## 1. 以太网驱动收包处理流程

以 ne2k_pci 驱动为例，介绍 linux 中以太网设备驱动是怎么从设备读取数据、构造 skb、并上传到上层的。

```c
else if ((pkt_stat & 0x0F) == ENRSR_RXOK) {
    struct sk_buff *skb;

    skb = netdev_alloc_skb(dev, pkt_len + 2);
    if (skb == NULL) {
        netif_err(ei_local, rx_err, dev,
                  "Couldn't allocate a sk_buff of size %d\n",
                  pkt_len);
        dev->stats.rx_dropped++;
        break;
    } else {
        skb_reserve(skb, 2);    /* IP headers on 16 byte boundaries */
        skb_put(skb, pkt_len);  /* Make room */
        ei_block_input(dev, pkt_len, skb, current_offset + sizeof(rx_frame));
        skb->protocol = eth_type_trans(skb, dev);
        if (!skb_defer_rx_timestamp(skb))
            netif_rx(skb);
        dev->stats.rx_packets++;
        dev->stats.rx_bytes += pkt_len;
        if (pkt_stat & ENRSR_PHY)
            dev->stats.multicast++;
    }
}
```

### 1.1 分配 skb 和预留空间

如果要从设备接收 `pkt_len` 大小的数据，首先会从设备中分配 `pkt_len + 2` 大小的内存。

然后 `skb_reverse(skb, 2)`，这个函数很简单：

```c
static inline void skb_reserve(struct sk_buff *skb, int len)  
{  
         skb->data += len;  
         skb->tail += len;  
 } 
```

也就是把 data 和 tail 都向后移动 2 位。

而 `skb_put(skb, pkt_len)` 如下：

```c
static inline unsigned char *skb_put(struct sk_buff *skb, unsigned int len)  
{  
         unsigned char *tmp = skb->tail;  
         SKB_LINEAR_ASSERT(skb);            
         skb->tail += len;                 // 移动指针  
         skb->len  += len;                 // 数据空间增大len  
         if (unlikely(skb->tail>skb->end)) // 空间不够
                 skb_over_panic(skb, len, current_text_addr());  
         return tmp;  
} 
```

则是把 tail 向后移动 `pkt_len` 位，让线性数据的真实长度变为 `pkt_len`。

### 1.2 读取数据

`ei_block_input(dev, pkt_len, skb, current_offset + sizeof(rx_frame));` 则是令 buf = skb->data，然后从硬件中 copy `pkt_len` 长度的数据到 buf 中。

### 1.3 eth_type_trans

阅读有关源码，这个函数做了两件事情：

- 将 mac_header 指向当前的 data 位置
- `skb_pull(ETH_HLEN)` 将 data 向后移动以太网帧头部长度，可以理解为丢弃了以太网帧头部
- 从 mac_header 读取网络层协议类型字段，并返回

这个函数运行完成以后，skb 的以太网头部被丢弃了，并将返回的网络层协议类型赋值给了 protocol。

### 1.4 上传 skb

调用 `netif_rx(skb);` 最终会调用到 `netif_receive_skb`，后者通过检查 skb->protocol 通过 ptype_base hash 表来上传给对应的上层协议栈处理。

## 2. 题外话

在第一章中说到，skb 除了线性部分，还有非线性部分，而从以太网驱动的处理流程中来看，数据全部都被放到了线性部分中。

而以太网头部在驱动就被处理掉了，传递给上层的包就只剩下 IP 往上层的包了。

