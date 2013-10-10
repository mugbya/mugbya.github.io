---
layout: post
title: cairo-dock 黑边框的问题 
thread: 4
categories: archlinux
tags: archlinux xfce4 
---

> 环境：archlinux xfce4    

在某一天，终于忍受了docky，便试着做些尝试，做些改变。  

以前在gnome下用过cairo-dock ,一直很喜欢，最是中意的属它的chrome主题，这次决定就换成它。安装很简单:

$ sudo pacman -S cairo-dock 

然后启用它，在启用的时候我没注意选择，最后导致出现黑边框的问题（第一次好好配置，可以避免的）。然后我去archwiki找解决方法,在最后确实找到的（<a href="https://wiki.archlinux.org/index.php/Cairo-Dock" targen="_blank"> 点击这里 </a> ），但是却不能用上。 

 在我这里，System 下没有任何东西，就别说 Composition 这个了。最后在网上找到解决方法，现在记录下了。 
 
 En:Settings -> Settings Manager -> Window Manager Tweaks ->Compositor-> Enable Display Compositing
 
 中文：设置 -> 设置管理器 -> 窗口管理器微调 -> 合成器 -> 启用显示合成 
 
