---
title: NexT主题下的Canvas_Nest使用教程
date: 2021-04-14 23:52:08
tags: [教程,Hexo,NexT]
categories: 博客搭建
---
#### 前言

最近用Hexo搭了个博客，用的是NexT主题。看到了别人用Canvas_Nest做背景，就是我们很熟悉，但是又叫不出名字的那种背景，就是[这个](https://git.hust.cc/canvas-nest.js/)样子的。

看了别人的教程，自己实践的时候却又不行（老经典了）。最后翻到源码仓库，发现开发者用了别的语法。

#### 实现方法

> 首先说明这里使用的版本
>
> Hexo：5.4.0
>
> NexT：8.3.0（v7.3.0+即可）

这基本是从[开发者仓库](https://github.com/theme-next/theme-next-canvas-nest)提供的教程翻译过来的

##### Step1

确保你的hexo文件夹里有`scaffolds`，`source`，`themes`和其它配置文件。说白了就是你得先把你的blog搭起来才能做这件事

##### Step2

创建一个 `footer.swig` 文件，把它放到 `hexo/source/_data` 目录下，如果没有 `_data` 文件夹就创建一个。该文件里写入下面的内容：

```html
<script color="0,0,255" opacity="0.5" zIndex="-1" count="99" src="https://cdn.jsdelivr.net/npm/canvas-nest.js@1/dist/canvas-nest.js"></script>
```

`color` 顾名思义就是颜色，`opacity` 是透明度，`count` 是生成点的个数

##### Step3

在NexT下的 `_config.yml` 配置里（一般都把这个叫做<u>主题配置文件</u>），找到 `custom_file_path` ，把里面的 `footer` 写成如下所示：

```yaml
# Define custom file paths.
# Create your custom files in site directory `source/_data` and uncomment needed files below.
custom_file_path:
  #head: source/_data/head.swig
  #header: source/_data/header.swig
  #sidebar: source/_data/sidebar.swig
  #postMeta: source/_data/post-meta.swig
  #postBodyEnd: source/_data/post-body-end.swig
  footer: source/_data/footer.swig
  #bodyEnd: source/_data/body-end.swig
  #variable: source/_data/variables.styl
  #mixin: source/_data/mixins.styl
  #style: source/_data/styles.styl
```

然后你就可以用 `hexo server` 看下效果如何了

