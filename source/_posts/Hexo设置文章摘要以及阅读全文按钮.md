---
title: Hexo设置文章摘要以及阅读全文按钮
date: 2021-04-19 11:07:18
tags: [教程,Hexo]
categories: 博客搭建
---
#### 前言

看了不少教程，发现都是老版本的教程，新版本Hexo的阅读全文功能已经单独分离成一个插件，没法直接配置

> 版本说明：
>
> Hexo：5.4.0
>
> 此处我是用的是NexT主题：8.3.0

#### Step1

安装相关插件。博客根目录下运行 `npm install hexo-excerpt --save` 

#### Step2

`站点配置文件` 中配置

```yaml
# 文章摘要
# Excerpt
excerpt:
  depth: 2
  excerpt_excludes: []
  more_excludes: []
  hideWholePostExcerpts: true
```

depth表示深度，也就是说当从文章（不包括标题）开始算起depth个代码块后就会开始显示阅读全文按钮

#### Step3

`主题配置文件` 中配置

```yaml
# Automatically excerpt description in homepage as preamble text.
excerpt_description: true

# Read more button
# If true, the read more button will be displayed in excerpt section.
read_more_btn: true
```

