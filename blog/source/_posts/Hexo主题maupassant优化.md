---
title: Hexo主题maupassant优化却失败
date: 2020-05-26 14:44:06
toc: true
tags:
 - 技术
 - maupassant
categories:
 - hexo 
---

## 1. 自白

重开博客, 发现以前的博客样子不怎么好看，就在`hexo`的 theme 库里选了很久，结果都没有选到自己喜欢的主题

但在浏览博客时却发现一个喜欢的主题，找了很久终于知道叫 [maupassant](https://github.com/pagecho/maupassant/), 不过找到的是 jekyll 的，不过最下面有 hexo 下的主题

使用时才知道它的目录是放在 文章的右侧，而不是在挂件(widget)的区域, 然后就尝试修改,想改成widget下的一个子挂件


然后我就开始了自我尝试

## 2. 修改



然后说下修改部分, `maupassant` 本身是支持 toc 的，故此修改的部分不算多

1. 首先是在 _widget 下增加 `tag.pug`, 内容如下

  ```yaml
  .widget
    .widget-title
      i(class='fa fa-folder-o')= ' ' + __('contents')
    != toc(page.content, {list_number: theme.toc_number})
  ```

  > **提示**
  >
  > 有关 pug语法问题 [请点击这里](https://pugjs.org/api/getting-started.html)，里面详细解释了`!=` 这是什么东西，以及其他一些用法

2. 修改 `maupassant/_config.yml`

  ```yaml
  # 修改 toc_number
  toc_number: 3

  widgets: 
    - toc
    - search
    ...

  ```


3. 修改样式

    ```css
    .toc-article {
        border: 1px solid #bbb;
        border-radius: 7px;
        margin: 1.1em 0 0 2em;
        padding: 0.7em 0.7em 0 0.7em;
        max-width: 40%;
    }
    
    .toc-title {
        font-size: 120%;
    }
    
    
    .toc {
        padding: 0;
        margin: 0.5em;
        line-height: 1.8em;
        li {
            list-style-type: none;
        }
    }
    
    .toc-child {
        margin-left: 1em;
        padding-left: 0;
    }
    ```



然后它就长这样 ![](/images/hexo-toc-01.png)


## 3. 问题

但是最后发现一个严重问题 ———— 不在文章详情页面时，也会有这个目录，包括首页，归档，这个就不对了！！！

但是我探究了很久我都搞不定这个问题，探究的结果是  **widget里面的变量全都变成了全局变量，即使页面发生变化但里面的变量的值还是初次的赋值**

目前只能探究到这，写下此篇文章，等有机会再来搞定它。

最终我只能全部撤回，使用默认的目录





