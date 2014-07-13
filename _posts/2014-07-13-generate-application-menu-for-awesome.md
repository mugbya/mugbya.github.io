---
layout: post
title: awesome的美化
thread: 30
categories: 记录
tags: awesome
---

	   环境 ： arch-x64      awesome-v3.5.5

awesome 都安装大概有一个月之久了，但因为刚入职，才接触这个 WM ,不敢花太多事件在这上面折腾，所以一直没什么进度。趁这周不忙，把awesome调教出来。

至于为什么用这个 WM，套用句仙子的话来说：我想知道它做了什么，它要在我的掌控之内。

的确，awesome的可塑性很强。基本从安装到使用，有很多的配置要做，要自己动手才能让它方便你的使用。当我调教到现在，我真的越来越喜欢它了，真的很不错。

#### awesome 配置 

awesome本也可不需要配置，但是不配置的或许大多数人都不会喜欢。我们大多都会建立自己的 tag, 自己的的应用程序菜单。


#### awesome 美化

awesome 默认是黑色的主题，我们可以修改其themes ，下面是我修改后的thems 

	beautiful.init("/usr/share/awesome/themes/zenburn/theme.lua")

我用的zenburn。将大体主题确定后，我们在修改一些细小的部分，让它便成我们理想中的样子，我们喜欢的样子。

如： 我要换个背景，将状态条放大点，将应用程序菜单扩大些。

背景在你选择的themes里面去换掉。如我在我的 <code>/usr/share/awesome/themes/zenburn</code>文件下 进行了更改，置换了图片，当然还需要修改配置完文件，即修改 theme.lua 。


打开它,修改 theme.wallpaper = "path" 。 path 是你的背景图片路径。

接下来，我们让应用程序菜单扩大下, 在该配置文件中 ，找到 

        theme.menu_height = 15 
        theme.menu_width  = 100

调试它们，知道成为你满意的样子。


要美化的地方可能还有很多，这个文件也还有很多可折腾的，我就做了我想做的一些事情，你也可以做一些你喜欢的事情，让它令你满意。如果觉得你做的更好，你或许可以分享出来，帮助更多的人。


或许你可以参考一些链接 ：  

- <a href="http://lilydjwg.is-programmer.com/2011/9/7/generate-application-menu-for-awesome.29327.html" target="_blank">依云</a>

- <a href="http://awesome.naquadah.org/wiki/Main_Page/zh" target="_blank">awesome.naquadah.org</a>

下面是我配置部分的效果展示
 ![图11](/assets/arch/awesome/awesome_1.jpg)



- 首次编辑：2014-07-13


