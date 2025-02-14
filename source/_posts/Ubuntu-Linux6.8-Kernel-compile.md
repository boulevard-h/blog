---
layout: blog
title: Ubuntu编译Linux6.8内核
date: 2024-06-04 21:02:40
categories: 'Linux Kernel'
---

记录一下踩的坑

## 1. 编译前准备

### 1.1 下载源码

请在 kernel.org 下载 tar 包并解压到 `~/linux-x.y.z` 目录。

### 1.2 安装编译工具链

正常来说，需要安装的编译工具链为：

```shell
sudo apt install libncurses5-dev build-essential openssl flex bison libssl-dev libelf-dev
```

其次，6.8 版本还需要安装如下两个工具来支持对应特性：

```
sudo apt install dwarves # 支持 BTF 选项
sudo apt install zstd # 支持 zstd 压缩内核镜像
```

当然，这里我用的是 Clang-12 编译的，而不是 gcc。Linux 6.8 以后**不推荐**使用 Clang-12 而是 Clang-13 以上版本，但是用 Clang-12 也不会报错，Linux 6.9 就会直接报错了。

## 2. 编译

### 2.1 config 阶段

最重要的，把 Ubuntu 的 config 复制过来：

```
cp /usr/src/linux-headers-5.15.0-107-generic/.config ~/linux-x.y.z/.config
```

由于 5.15 和 6.8 版本内核的选项有很多变化，后面编译的时候会弹出很多问题要确认，建议一路回车，这么多也看不过来...

然后，`make menuconfig` 修改你自己想要的选项。

### 2.2 编译镜像

```
make -j$(nproc) CC=clang-12
```

报错过程中有很多坑，下面列出一些常见的：

1. BTF 与 zstd 报错，缺少第一章中提到的 dwarves 与 zstd 工具，或者关掉 BTF 与 zstd 选项也可以。

2. certs 有关报错：Debian 系都会验证签名，修改 `.config` 文件中的如下选项：

   ```
   CONFIG_SYSTEM_TRUSTED_KEYS=""
   CONFIG_SYSTEM_REVOVATION_KEYS=""
   ```

3. 栈溢出 `error: the frame size of xxxx bytes is larger than 1024 bytes`。在 `.config` 中调整栈大小即可。

   ```
   CONFIG_FRAME_WARN=XXXX # 4096 或者 8192
   ```

### 2.3 安装

```
make modules_install
make install
```

没报错的话直接重启即可选择系统
