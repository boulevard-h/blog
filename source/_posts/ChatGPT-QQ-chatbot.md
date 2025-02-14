---
layout: blog
title: ChatGPT QQ机器人快速配置
categories: 瞎折腾
date: 2023-03-21 14:56:51
---

## 准备工具

1. ChatGPT-mirai-QQ-bot

   [GitHub - lss233/chatgpt-mirai-qq-bot: 🚀 一键部署！真正的 ChatGPT QQ 聊天机器人！支持ChatGPT API、 ChatGPT Plus、新版 Bing，多账号负载均衡，人设调教，敏感词检测，虚拟女仆、对话上下文，图片渲染，代理加速 (内有视频教程）](https://github.com/lss233/chatgpt-mirai-qq-bot)

   在GitHub下载最新Release压缩包即可

2. 手机滑块验证码工具

   https://pan.baidu.com/s/1Ij-CCHgY7S16cZo584RvsA 

   提取码：7qjk

## 配置思路

本项目由mirai和chatgpt两个模块组成，mirai负责与QQ通信，将信息转发到chatpgt，然后把chatgpt的回复发送到QQ

只要在两个模块上分别登录好QQ和ChatGPT账户即可

## 配置过程

### 初始化&配置ChatGPT账户

解压缩安装包，会有一个`初始化.bat脚本`，直接运行，完毕以后会弹出一个cfg配置文件，在其中配置ChatGPT账户：

- 配置QQ号

![image-20230322163044228](/images/ChatGPT-Mirai/image-20230322163044228.png)

- 配置梯子![image-20230322163126537](/images/ChatGPT-Mirai/image-20230322163126537.png)
- 配置OpenAI API密钥![image-20230322163141324](/images/ChatGPT-Mirai/image-20230322163141324.png)

当然，提供了很多的OpenAI登陆账号，这里选择除了贵没有缺点的API方式（反正有免费额度

### 配置Mirai

这玩意配置bug很多，出问题了建议多上GitHub看看issue（毕竟是“非法”登陆腾讯的东西^ ^

点击`启动Mirai.bat`，然后等待Mirai初始化完成，输入

```c
Login qq号 qq密码 设备类型
```

设备类型等于是模拟在什么设备上面登录，一开始选择的是ANDROID_WATCH也就是安卓手表，出现了error，查询了[Mirai官方的解决方案](https://mirai.mamoe.net/topic/223/无法登录的临时处理方案?lang=zh-CN)，把设备改为了MACOS就行了

然后，如果是第一次登陆的话，会要进行验证，如下图所示：

![image-20230322163737380](/images/ChatGPT-Mirai/image-20230322163737380.png)

复制这个captcha链接，在滑动验证码app中输入并完成验证：

![image-20230322163809239](/images/ChatGPT-Mirai/image-20230322163809239.png)

会得到一串TX值，赋值TX值，直接输入到Mirai终端就可以完成登录

### 启动ChatGPT

然后点击`启动ChatGPT.bat`，如下图就是登陆成功：

![image-20230322163918590](/images/ChatGPT-Mirai/image-20230322163918590.png)

到私聊或者到群里@进行对话测试
