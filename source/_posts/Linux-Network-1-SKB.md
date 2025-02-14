---
layout: blog
title: 'Linux Network 1: SKB'
categories: 'Linux Kernel'
date: 2024-07-01 19:40:21
---

## 参考链接

[Linux内核：sk_buff解析 - 唐稚骅 - 博客园 (cnblogs.com)](https://www.cnblogs.com/tzh36/p/5424564.html)

[skb结构和相关操作函数 - 简书 (jianshu.com)](https://www.jianshu.com/p/3c5d5fa339fc)

[Linux 内核网络协议栈 ------sk_buff 结构体 以及 完全解释 （2.6.16） - 明明是悟空 - 博客园 (cnblogs.com)](https://www.cnblogs.com/x_wukong/p/6650056.html)

## 1. SKB 结构概览

SKB 的结构是这样一个很复杂的图，可以看到 skb 分为三部分：控制（`struct sk_buff` 本身）、线性数据、非线性数据（skb_shared_info）

![img](/images/Linux-Network-1-SKB/SouthEast.png)

## 2. SKB 线性数据部分

SKB 的线性数据部分是用于存放数据包的**连续内存空间**，所以称为是线性的，主要有 head, end, data, tail 四个指针。

```C
struct sk_buff {
    
...

__u16 transport_header; //传输头相对于skb->head的偏移

__u16 network_header;//网络头相对于skb->head的偏移

__u16 mac_header;//以太网头相对于skb->head的偏移

/* These elements must be at the end, see alloc_skb() for details. */

sk_buff_data_t tail;

sk_buff_data_t end;

unsigned char *head, *data;

}
```

- head 和 end 指向数据趋于的头部和尾部，是分配的时候就固定的
- data 和 tail 是真正数据的开头和结尾
- head 与 data 之间的趋于被称为 headroom，data 和 tail 之间的区域被称为 tailroom。刚刚分配的时候，headroom 大小为 0，tailroom 大小为 size，后续的操作通过移动 data 和 tail 来完成。

四者的关系在这个图里面很清楚：

![img](/images/Linux-Network-1-SKB/941007-20160423142022179-2013851116.jpg)

## 3. SKB 非线性数据部分

对于大型或者是分片的数据包，其数据无法存储到连续的内存块也就是 data 中，所以需要非线性的数据存储。

skb 中有一个叫做 skb_shared_info 的结构来管理非线性部分：

```C
struct skb_shared_info {
    atomic_t dataref;
    unsigned short nr_frags;
    unsigned short gso_size;
    unsigned short gso_segs;
    unsigned short gso_type;
    __be32 ip6_frag_id;
    struct sk_buff *frag_list;
    skb_frag_t frags[MAX_SKB_FRAGS];
};
```

- nr_frags：非线性部分的片段数量
- frags：片段

每一个片段 skb_frag_t 如下：

```
typedef struct skb_frag_struct {
    struct page *page;
    __u32 page_offset;
    __u32 size;
} skb_frag_t;
```

这个结构体记录了片段在内存中的位置和大小。

### 3.1 非线性部分的用途

非线性部分的用途主要有：

- 数据包过大，难以分配连续内存
- IP 分片和重组

## 4. skb->len 与 skb->data_len

len 指的是数据包的总长度，也就是线性和非线性长度之和。

data_len 虽然带有 data 字样，但**不是**线性部分 data 的长度，**而是**非线性部分的长度。

而线性部分 skb_headlen(skb) = skb->len - skb->data_len。
