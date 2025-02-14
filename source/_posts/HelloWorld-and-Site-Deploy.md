---

title: HelloWorld & 建站教程
categories: 博客搭建
date: 2024-10-12 11:39:16
---

## 1. 简要介绍

[hexo](https://hexo.nodejs.cn/) 是一个强大的博客框架项目，将 Markdown 文档转化为博客网站，并且拥有大量自定义主题模板可以使用。在安装了 hexo 以后，只需要在 hexo 博客目录下简单的执行一行 `hexo server` 命令即可生成网站，并且在本地的 4000 端口访问它。

因此，本教程的重点不是如何安装和使用 hexo，而是如何把你的 hexo 页面部署在网站上。

GitHub Pages 就是一个不错的部署平台，hexo 官方支持 GitHub 一键部署，可以直接将生成的网页文件推送到 GitHub Pages 对应的仓库中去。

但是 GitHub Pages 也有一些缺点：

1. 国内访问速度慢（emmm
2. 源码和静态网站分离，也就是说，被我们 push 到 GitHub 仓库中的只有静态网站，而没有你的 hexo 目录

第二个问题可以使用 GitHub Actions 解决。使用两个仓库（或者一个仓库的两个分支也可），一个存储 hexo 源码，一个存储静态网站。然后在 hexo 源码仓库中写一个 actions 脚本，当推送新内容的时候，自动运行 hexo 生成静态网站并且推送到另一个仓库。

而更好的方法是使用 Netlify，其在国内的访问速度比 GitHub Pages 快，而且非常方便：与你的 GitHub hexo 源码仓库关联以后，会自动拉取其源码，然后自动部署生成网站。

**唯一美中不足的是，在我使用了两年的 Netlify 后，我的账号莫名其妙被 ban 了**，所以本网站目前又迁移回了 GitHub Pages。

## 2. GitHub Pages 配置教程

follow 这篇博客即可，实测没有问题：

[HEXO系列教程 | 使用GitHub部署静态博客HEXO | 小白向教程 – 夜梦星尘の折腾日记 (yemengstar.com)](https://tech.yemengstar.com/hexo-tutorial-deploy-githubpages-beginner/)

[HEXO系列教程 | 为HEXO绑定自己的域名 | 小白向教程 – 夜梦星尘の折腾日记 (yemengstar.com)](https://tech.yemengstar.com/hexo-tutorial-use-your-domain-beginner/)

## 3. Netlify 配置教程

如果有幸能够注册 Netlify 账号而且不被 ban 的话，可以按照这个教程配置：

[个人博客搭建教程 | 爱扑bug的熊 (cuijiacai.com)](https://blog.cuijiacai.com/blog-building/)

但是没有配置其中的CloudWare CDN加速，Netlify自带的已经够用
