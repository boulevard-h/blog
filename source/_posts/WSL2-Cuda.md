---
layout: blog
title: WSL2配置Cuda
categories: 环境搭建
date: 2023-03-22 11:37:28
---

## 参考资料

[CUDA Toolkit 12.1 Downloads | NVIDIA Developer](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=WSL-Ubuntu&target_version=2.0&target_type=deb_local)

## 配置过程

### 在Windows中安装显卡驱动

注意是在Windows而不是WSL中安装驱动，安装的是游戏驱动即可

如果已经安装（毕竟打游戏的一般都提前安装好了），只需要在Windows powershell中输入`nvidia-smi`即可验证

### 在WSL2中配置

参考开头给出的链接，只需要输入图中的这些命令就可以配置完成

![image-20230322164339279](/images/wsl-cuda/image-20230322164339279.png)

然后到wsl里面再输入一遍`nvidia-smi`即可验证

### 注意事项

1. 显卡驱动是最新的Game Driver即可，安装在Windows而不是WSL
2. WSL版本需要为WSL2，WSL1不支持
3. 不支持Maxwell架构显卡（截至博客发布时间
4. 如果是SSH登录WSL，可能会提示没有`nvidia-smi`命令，可能是环境变量的问题，输入绝对路径`/usr/lib/wsl/lib/nvidia-smi`运行即可。方便起见，可以添加一个环境变量
