---
layout: blog
title: SKY-UPSI
date: 2023-03-01 20:58:58
categories: 隐私计算
tags: [PSI, UPSI, C++]
---

本文主要是对基于[OPRF-PSI](https://github.com/peihanmiao/OPRF-PSI)的SKY-PSI进行了UPSI框架适配，以及更换[BinaryFuseFilter]([GitHub - FastFilter/xor_singleheader: Header-only binary fuse and xor filter library](https://github.com/FastFilter/xor_singleheader/))进行OPRF值传递。

## 修改BinaryFuseFilter

在OPRF-PSI中，计算出来的OPRF值是直接发送过去的，我们的优化思路是把值放到过滤器中存储起来，然后发送过滤器参数给对方，让对方重构过滤器然后查询OPRF值。

## TBC

后面的忘了...妈妈生的^ ^
