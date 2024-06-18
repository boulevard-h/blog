---

title: LaTeX语法支持教程
categories: 博客搭建
tags: 简易教程
date: 2022-12-7 12:07:00
mathjax: true
---

## 教程来源

主要参考[(98条消息) hexo下LaTeX无法显示的解决方案_zealscott的博客-CSDN博客](https://blog.csdn.net/crazy_scott/article/details/79293576)

略有不同，主要在主题的_config.yml

## 安装插件

首先需要安装mathjax：

``` shell
npm install hexo-math -save
```

然后需要把hexo默认的md渲染引擎[hexo-renderer-marked](https://github.com/hexojs/hexo-renderer-marked)换成[hexo-renderer-kramed](https://github.com/sun11/hexo-renderer-kramed)：

``` shell 
npm uinstall hexo-renderer-marked -save
npm install hexo-renderer-kramed -save
```

## 修改转义

LaTeX与markdown语法有语义冲突，在markdown中，斜体和加粗可以用\*或者\_表示，在这里我们修改变量，将\_用于LaTeX，而使用\*表示markdown中的斜体和加粗

找到博客下`node_modules\kramed\lib\rules\inline.js`修改如下

``` js
  //escape: /^\\([\\`*{}\[\]()#$+\-.!_>])/,
  escape: /^\\([`*\[\]()#$+\-.!_>])/,
```

``` js
  //  em: /^\b_((?:__|[\s\S])+?)_\b|^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
  em: /^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
```

## 修改配置

原文中说需要找到主题下的`_config.yml`修改如下：

``` yaml
# MathJax Support
mathjax:
  enable: true
  per_page: true
  #cdn: //cdn.bootcss.com/mathjax/2.7.1/latest.js?config=TeX-AMS-MML_HTMLorMML
  cdn: //cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-AMS-MML_HTMLorMML
```

但是我用的`Chic`主题，进去以后发现`config.yml`有关部分是这样的：

``` yaml
# plugin functions
## Mathjax: Math Formula Support
## https://www.mathjax.org
mathjax:
  enable: true
  import: demand # global or demand
  ## global: all pages will load mathjax,this will degrade performance and some grammers may be parsed wrong.
  ## demand: Recommend option,if your post need fomula, you can declare 'mathjax: true' in Front-matter
```

就没修改了

## 测试

在博客开头加上 `mathjax: true`

然后测试效果如下：

$\Sigma _0 ^n \frac{1}{n}$
