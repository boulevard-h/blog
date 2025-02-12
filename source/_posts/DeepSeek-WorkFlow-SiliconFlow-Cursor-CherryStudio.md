---
layout: blog
title: 使用SiliconFlow API+Cursor+CherryStudio搭建DeepSeek AI工作流
date: 2025-02-12 11:06:43
categories: 杂项
---

## 1. API 的选择与获取

目前使用过的 API 平台有三种：

- ds 官方：服务器老寄、目前已经限制了 API 账户的充值
- 腾讯云：暂时免费（2.25前）、但是其 API 不知为何在 Cursor 中调用报错
- SiliconFlow：V3 8元/1M token，R1 16元/1M token

处于可用性的考虑，目前只能选择 SiliconFlow

https://cloud.siliconflow.cn/models 进入 SiliconFlow 网站，注册、实名、然后创建 API Key 即可，记得创建两个 API Key，分别给两个软件使用

![image-20250212111702905](/images/DeepSeek-WorkFlow-SiliconFlow-Cursor-CherryStudio/image-20250212111702905.png)

## 2. Cherry Studio 配置

Cherry Studio 是一个支持多模型的 AI 助手客户端，只需要添加 API Key 即可对话

在 https://cherry-ai.com/ 下载和安装即可，进入软件后，在设置-模型服务中找到硅基流动（SiliconFlow）的服务添加 API Key 即可，设置默认模型以后，就可以和 deepseek 对话

如果你想先白嫖一段免费的腾讯云，Cherry Studio 也支持添加自定义的 API，选择添加，提供商名字可以取名叫”腾讯云deepseek“，提供商类型选择 OpenAI，API 地址为：https://api.lkeap.cloud.tencent.com，然后添加 deepseek-r1 和 deepseek-v3 模型即可

![image-20250212112439082](/images/DeepSeek-WorkFlow-SiliconFlow-Cursor-CherryStudio/image-20250212112439082.png)

## 3. Cursor 配置

Cursor 是基于 vscode 开发的 AI IDE，其 AI 功能比 vscode 更加强大，例如报错以后一键对报错提问等。Cursor 支持 GPT-o1、Claude-3.5、deepseek 等模型，但是使用官方提供的模型服务需要 20 美刀一个月（没有 visa 或 PayPal 等的话即使愿意出钱也会被卡在支付方式上），所以选择不开通官方会员，而是使用自己的 API

在 Cursor-Model 中，首先取消勾选所有自带的模型（包括自带的 deepseek），添加两个模型：deepseek-ai/DeepSeek-R1和deepseek-ai/DeepSeek-V3

![image-20250212113359216](/images/DeepSeek-WorkFlow-SiliconFlow-Cursor-CherryStudio/image-20250212113359216.png)

然后在下方的 OpenAI API Key 中，修改 Override OpenAI Base URL 为 https://api.siliconflow.cn/v1，然后在上方填入自己的 API Key，如果能 Verify 通过的话就能正常使用了

![image-20250212113427747](/images/DeepSeek-WorkFlow-SiliconFlow-Cursor-CherryStudio/image-20250212113427747.png)
