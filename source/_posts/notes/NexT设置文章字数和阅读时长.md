---
title: NexT设置文章字数和阅读时长
tags:
  - 博客搭建
  - Hexo
categories:
  - 笔记
  - 博客搭建
abbrlink: 6aa33862
date: 2021-04-19 13:54:51
---
#### 前言

新版本Hexo把这些小功能基本都做成插件分离出去了，没法像以前版本一样直接配置

> 版本说明：
>
> Hexo：5.4.0
>
> 此处我是用的是NexT主题：8.3.0

#### Step1

安装插件，`npm install hexo-word-counter`

还有另一个插件，叫 hexo-wordcount，可以在各种主题下用。不过比较一下，NexT还是用这个方便一些

#### Step2

`站点配置文件` 中配置

```yaml
# 文章字数统计和阅读时长估计
symbols_count_time:
  symbols: true                # 文章字数统计
  time: true                   # 文章阅读时长
  total_symbols: true          # 站点总字数统计
  total_time: true             # 站点总阅读时长
  exclude_codeblock: true      # 排除代码字数统计
  awl: 2                       # Average Word Length（平均词长）
  wpm: 200                     # Words Per Minute（每分钟阅读词数）
```

其中 awl 和 wpm 是用来计算阅读时长的

#### Step3

`主题配置文件` 中配置

```yaml
# 字数统计，阅读时长
# Post wordcount display settings
# Dependencies: https://github.com/next-theme/hexo-word-counter
symbols_count_time:
  separated_meta: true     # 是否另起一行（true的话不和发表时间等同一行）
  item_text_post: true     # 首页文章统计数量前是否显示文字描述（本文字数、阅读时长）
  item_text_total: true    # 页面底部统计数量前是否显示文字描述（站点总字数、站点阅读时长）
```

#### 参考

[next-theme/hexo-word-counter: ⏰ Symbols count and time to read of articles for Hexo. (github.com)](https://github.com/next-theme/hexo-word-counter)
