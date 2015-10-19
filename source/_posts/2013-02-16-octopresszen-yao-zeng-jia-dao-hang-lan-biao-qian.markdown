---
layout: post
title: "Octopress怎么增加导航栏标签"
date: 2013-02-16 13:24
comments: true
categories: octopress
---

多年前学习的html知识都又还回去了，再加上不懂ruby，对博客样式的修改觉得还很吃力。几经摸索，找到增加导航栏标签的方法如下：

###用命令行增加新页面
	tumatoMac-mini:octopress tuchangwei$ rake new_page[你要添加的标签]
	mkdir -p source/你要添加的标签
	Creating new page: source/你要添加的标签/index.markdown 
	
###在source/_includes/custom/navigation.html中添加如下代码

	<li><a href="{{ root_url }}/你要添加的标签">你要添加的标签名</a></li>
	
那么剩下的操作就是在新的标签页的index.markdown文件中写博文了。
	
