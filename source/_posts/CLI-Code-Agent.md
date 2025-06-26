---
layout: blog
title: CLI-Code-Agent与Gemini CLI配置
date: 2025-06-26 15:41:12
categories: 环境搭建
---

最近 Cursor 取消了快速请求，即使是 Pro 会员的请求也变得非常慢，属实是有点玩不起了...而同类型的原生 AI IDE 例如 VSCode Copilot、Trae、Windsurf 用起来总感觉还是没 Cursor 爽（还是放不下 Cursor 的一年会员^_^）

于是尝试了一下最近很火的 CLI Code Agent，最知名的 Claude Code 和 Google 刚出的 Gemini CLI：

- Claude Code 定价还是经典的 20 刀一个月，但是 Claude 众所周知的政治站位和小心眼让人不是很敢往里边充钱
- Gemini CLI 是开源的，再加上之前已经白嫖过一年的 Gemini Pro 了，所以选择这个来尝试

## 1. 安装和启用

这个很简单，使用 npm 安装即可（没有 npm 命令的话需要先安装 Node.js）

```shell
npm install -g @google/gemini-cli
```

安装好以后，使用前建议现在命令行中设置代理的环境变量，以我的 Clash-Verge-Rev 使用 7897 端口为例：

``` shell
export https_proxy=http://127.0.0.1:7897 http_proxy=http://127.0.0.1:7897 all_proxy=socks5://127.0.0.1:7897
```

然后再运行 `gemini` 命令进入，初次进入会要求设置主题、登录账号等，一步一步来即可

## 2. Google Cloud 启用

按理来说，登录账号以后就可以开用了，但是不知道为什么，我的账号被识别为了 Workspace account（似乎是所有启用了 Google Cloud 的账户都被视为 Workspace account），一定需要配置 Google Cloud Project 才可以使用，否则会出现如下界面：

<img src="/images/CLI-Code-Agent/image-20250626155910225.png" alt="image-20250626155910225" style="zoom:50%;" />

那就按照要求去启用，首先访问 https://cloud.google.com/gemini/docs/discover/set-up-gemini#enable-api：

<img src="/images/CLI-Code-Agent/image-20250626160204043.png" alt="image-20250626160204043" style="zoom:50%;" />

点击启用，然后去找你的 Project ID，进入 https://console.cloud.google.com/，如果你此前没有使用过 Google Cloud 的话，可以使用默认创建的 My First Project，如图左上角所示：

![image-20250626160357013](/images/CLI-Code-Agent/image-20250626160357013.png)

点击这个 Project，即可获取其 ID：

<img src="/images/CLI-Code-Agent/image-20250626160423376.png" alt="image-20250626160423376" style="zoom:50%;" />

然后把这个 ID 导入到你的环境变量中：

``` shell
export GOOGLE_CLOUD_PROJECT="上面的这个ID"
```

再次启动 gemini 就可以用了。

当然 export 的方式是临时生效的，可以把这个环境变量放大你的 shell 配置文件中，这里以 zsh 为例：

``` shell
echo 'export GOOGLE_CLOUD_PROJECT="上面的这个ID"' >> ~/.zshrc
source ~/.zshrc
```





