---
layout: blog
title: wsl配置zsh
categories: 不务正业の小玩意
date: 2023-03-22 18:23:11
tags: [wsl, zsh, Linux美化]
---

## 前提条件

请在科学上网环境执行

参考：[Oh My Zsh - a delightful & open source framework for Zsh](https://ohmyz.sh/#install)

## 安装zsh

执行`sudo apt install zsh`即可

## 自动配置

输入

```shell
sh -c "$(wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```

自动执行，中途会提示是否设置zsh为默认shell，输入yes回车即可

## 更换主题

在[Themes · ohmyzsh/ohmyzsh Wiki · GitHub](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes)中选择心仪的主题，在`~/.zshrc`中设置`ZSH_THEME="主题名"`然后执行`source ~/.zshrc`即可
