---
title: NexT主题下一些样式调整的记录
date: 2021-04-16 21:59:45
tags: [博客搭建,Hexo]
categories: [笔记,博客搭建]
---
#### 前言

本文是对于NexT主题的样式的改动记录，以及遇到的一些问题的处理方法

> 版本支持如下：
>
> Hexo：5.4.0
>
> NexT：8.3.0（低几个版本应该也没问题）

#### 去掉博客顶部的黑条

`themes/next/layout/_layout.njk` 中，找到如下`div`并注释掉

```html
<body ...>
   <div class="headband"></div>
   <!--<div class="headband"></div>-->
... 
</body>
```

这个不影响你的进度条动画，如果你想把进度条动画也关了，可以在**主题配置文件**（next/_config.yml）中用 `nprogress` 调整

#### 修改文章内超链接样式

`themes/next/source/css/_common/components/post/post-body.styl` 中，结尾加上语句，颜色什么的可以自己调整

```stylus
// 文章超链接文本样式修改
.post-body {
  p,li{
    a{ 
      color: #0593d3; 
      border-bottom: none; 
      border-bottom: 1px solid #0593d3; 
        &:hover { 
          color: #fc6423; 
          border-bottom: none; 
          border-bottom: 1px solid #fc6423; 
        }
    } 
  }
}
```

注意，这个修改会同时影响正常模式和暗黑模式

#### 修改“阅读全文”按钮样式

这个我找了不少方案，但是都是像上面那个一样，用某个样式覆盖了原有的样式。它们都有一个问题：会影响暗黑模式下的样式，但是我想保留暗黑模式的样式。于是在next的源码里寻找修改它的基础样式，下面是设置方法

找到 `themes\next\source\css\_variables` 目录，下面有 `base.styl` 和 `Gemini.styl`，`Mist.styl`，`Muse.styl`，`Pisces.styl` 四个主题文件。找到我们博客使用的主题对应的文件（此处以 <u>Gemini</u> 为例），发现引入了 Pisces 的设置，说明这两种主题大部分样式是通用的（Mist和Muse同理）。我们在 `Pisces.styl` 中找到如下部分：

```stylus
// Button
// 按钮配色，这里的配色Gemini也通用（因为是import进去的）
// 此处均影响正常模式（除了圆角）
$btn-default-radius             = 2px;                // 按钮边框圆角半径
$btn-default-bg                 = white;              // 按钮背景色
$btn-default-color              = $text-color;        // 字体颜色
$btn-default-border-color       = $text-color;        // 按钮边框颜色
$btn-default-hover-bg           = $black-deep         // hover时按钮背景色
$btn-default-hover-color        = white;              // hover时字体颜色

// 在 base.styl 中，还有一个相关属性
$btn-default-hover-border-color = $black-deep         // hover时按钮边框颜色
```

按照自己想要的方式修改就可以了。另外，这里的值可以在 `base.styl` 中找到，你也可以自己定义一些值，或者直接用16进制数。

```stylus
// Color system
// --------------------------------------------------
$whitesmoke   = #f5f5f5;
$gainsboro    = #eee;
$grey-lighter = #ddd;
$grey-light   = #ccc;
$grey         = #bbb;
$grey-dark    = #999;
$grey-dim     = #666;
$black-light  = #555;
$black-dim    = #333;
$black-deep   = #222;
$red          = #ff2a2a;
$blue-bright  = #87daff;
$blue         = #0684bd;
$blue-deep    = #262a30;
$orange       = #fc6423;

// 自已定义颜色
$m-indigo       = #3F51B5
$m-teal         = #009688
$m-blue         = #448AFF
$m-light-blue   = #03A9F4
$m-blue-grey    = #607D8B
```

此外，如果要修改暗黑模式下的按钮样式，在 `base.styl` 中找到如下属性，对其中带有 `dark` 字样的属性修改即可。

```stylus
// 全局默认按钮样式
// Buttons
// --------------------------------------------------
$btn-default-radius                    = 0;
$btn-default-bg                        = $black-deep;
$btn-default-bg-dark                   = $black-deep;
$btn-default-color                     = white;
$btn-default-color-dark                = $text-color-dark;
$btn-default-border-color              = $black-deep;
$btn-default-border-color-dark         = $black-light;
$btn-default-hover-bg                  = white;
$btn-default-hover-bg-dark             = $grey-dim;
$btn-default-hover-color               = $black-deep;
$btn-default-hover-color-dark          = $text-color-dark;
$btn-default-hover-border-color        = $black-deep
$btn-default-hover-border-color-dark   = $grey-dim;
```

效果图（orange）：

![image-20210416210010804](https://cdn.jsdelivr.net/gh/kudryavka1013/note-pic@master/note/20210416210053.png)

![image-20210416210120579](https://cdn.jsdelivr.net/gh/kudryavka1013/note-pic@master/note/20210416210120.png)

#### 修改左上角站点标题块背景颜色

![image-20210416210852030](https://cdn.jsdelivr.net/gh/kudryavka1013/note-pic@master/note/20210416210852.png)

`themes\next\layout\_partials\header\brand.njk` 文件中，找到如下语句（就在第一行），加上内联样式`style="background: ..."`

```html
<div class="site-brand-container">
<!-- 颜色自己设置 -->
<div class="site-brand-container" style="background: #607D8B"> 
```

这会对正常模式和暗黑模式都生效。

#### 修改首页文章标题样式

`themes/next/source/css/_common/components/post/post-header.styl` 中，找到如下语句，颜色自己定义（16进制数）

```stylus
.posts-expand .post-title-link {
  border-bottom: 0;
  color: var(--link-color);   //此处为标题颜色
  display: inline-block;
  position: relative;

  &::before {
    background: var(--link-color);     //此处为下划线颜色
    bottom: 0;
    content: '';
    height: 2px;
    // Fix issue #75
    left: 0;
    position: absolute;
    transform: scaleX(0);
    transition: transform $transition-ease;
    width: 100%;
  }
  
  // 加上下方语句，为hover时标题的颜色
  &:hover{
    color: $m-green;
  }
  // 加上上方语句

  &:hover::before {
    transform: scaleX(1);
  }

  .fa-external-link-alt {
    font-size: $font-size-small;
    margin-left: 5px;
  }
}
```
#### 修改文章内h1、h2、h3等标签的样式

> 参考文章：
>
> [Hexo Next 主题字体相关配置 | 禾七博客 (leay.net)](https://leay.net/2020/02/14/hexo-next-font/)
>
> [blog/styles.styl at master · muzi502/blog (github.com)](https://github.com/muzi502/blog/blob/master/source/_data/styles.styl)

`themes/next/source/css/_common/components/post/post-body.styl` 文件中，添加如下配置，其中颜色自己修改

```stylus
// 文章内标题样式（左边的竖线和下划线）
.post-body {
  h2, h3{
    color: $m-green;
    border-left: 6px solid $m-green;
    margin-left: 0px;
    padding-left: 24px;
    border-bottom: 1px solid $m-green;
    padding-bottom: 0.3em;
    position: relative;
  }

  h4 {
    color: $m-green;
    border-left: 5px solid $m-green;
    margin-left: 0px;
    padding-left: 24px;
    border-bottom: 1px solid #cfd8dc;
    padding-bottom: 0.3em;
    position: relative;
  }

  h5 {
    color $m-green;
    border-left: 4px solid $m-green;
    margin-left: 0px;
    padding-left: 16px;
    position: relative;
  }

  h6 {
    color: $m-green;
  }
}
```

效果图：

![image-20210420154731795](https://cdn.jsdelivr.net/gh/kudryavka1013/note-pic@master/note/image-20210420154731795.png)

![image-20210420154704110](https://cdn.jsdelivr.net/gh/kudryavka1013/note-pic@master/note/image-20210420154704110.png)

---

补充：`themes/next/source/css/_schemes/Gemini/index.styl` 下修改h1，h2，h3的`border-bottom`样式