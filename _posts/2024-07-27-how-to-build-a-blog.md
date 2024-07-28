---
layout:     post
title:      "如何快速搭建一个博客"
comment:    true
subtitle:   ""
date:       2024-07-27 16:00:00
author:     "Yi"
header-img: "img/post-bg-2015.jpg"
tags:
    - Life
---

如题，这篇文章记录一下搭建一个简易博客的步骤。<del>竟然快大四才搭自己的博客。</del>

## 选择一个博客模板

为了方便，我们可以先去github上找一个自己喜欢的博客模板，然后fork到自己的仓库中

这里仓库一定要取名为：`username.github.io`，这样我们就可以通过`username.github.io`来访问我们的博客了。`username`是你的github用户名。

这里我选择了[Huxpro](https://huangxuan.me/)大佬的模板，[仓库在这儿](https://github.com/Huxpro/huxpro.github.io)。
这里还有一个备用的模板，[huxblog-boilerplate](https://github.com/huxpro/huxblog-boilerplate)，但是我用这个搭建过程中出现了问题。

## 修改仓库配置

![Github设置](/img/in-post/how-to-build-a-blog/github.png)

在仓库的`Settings`中，找到`Pages`，然后选择`Branch`为`master`，点击Save，这里其实就配置好了，稍等一段时间就可以根据`username.github.io`访问到你的博客了。

## 发布自己的博客

仓库建立之后，`clone`到本地，在`_posts`目录中添加`markdown`文件，并按照`_doc`文件夹下的两个手册的说明进行编写，然后`push`到仓库中，稍等一会就可以在博客中看到你的文章了。

## 结语

之前一直有搭建博客的想法，但是一直太忙了（<del>太懒了</del>），之后会坚持写一点文章分享学习的心得和笔记。
