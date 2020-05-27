---
title: Hexo主题maupassant优化
date: 2020-05-26 14:44:06
tags:
 - 技术
 - 来吧
categories:
 - hexo 
---

## 1. 效果

重开博客, 发现以前的博客样子不怎么好看，就在`hexo`的 theme 库里选了很久，结果都没有选到自己喜欢的主题

但在浏览博客时却发现一个喜欢的主题，找了很久终于知道叫 `maupassant`, 不过找到的是 jekyll 的，不过最下面有 hexo 下的主题

然后我就制作了自己的主题：[maupassant-hexo](https://github.com/mugbya/maupassant-hexo)


拿到后主要修改了 toc 的样式。现在样子是这样 ![](/images/hexo-toc-01.png)


## 2. 修改

然后说下修改部分, `maupassant` 本身是支持 toc 的，故此很多修改的部分不算多

首先是在 _widget 下增加 tag.pug

```css


```


`maupassant/_config.yml`

```yaml

# 修改 toc_number

toc_number: 3



```
