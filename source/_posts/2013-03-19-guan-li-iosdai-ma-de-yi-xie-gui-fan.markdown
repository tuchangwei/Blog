---
layout: post
title: "管理iOS代码的一些规范"
date: 2013-03-19 20:08
comments: true
categories: Code-Standard
---
1.	做任何事前先`git init`。
2.	使用[CocoaPods](http://cocoapods.org/)管理第三方库。
3.	使用ARC。如果第三方库没有使用ARC，可以在他们的.m文件中加上`-fno-objc-arc`关键字。
4.	清除`@synthesize`。当你为成员变量增加`@property`时，Xcode会在编译时候默认为你增加`@synthesize`关键字。
5.	移除注释掉的代码或工程中未使用的文件。
6.	不要在AppDelete中写代码。
7.	增加`#pragram mark –`在你需要注释的地方
8.	管理好Xcode工程中的文件夹结构。一般使用`Model`,`View`,`Controller`,`Helper/Manager`和`Images`文件夹。如果有第三方库，则用`External`文件夹。
9.	代码格式，空格。
10.	将`__mycompany`改成有意义的。
11.	时常NSLog，形成日志系统。

-----------------神奇的分割线-------------------

又找到了一些规范：

Under the Warnings header, also set the Other Linker Flags option to -Wall. The compiler will now check for all possible situations that can cause problems. By default many of these warnings are turned off but I find it useful to always have all warnings on and to treat them as fatal errors. In other words, if the compiler gives a warning I will first fix it before continuing. Whether that is something you may want to do on your own projects is up to you, but during the conversion to ARC I recommend that you take a good look at any issues the compiler may complain about.
For the exact same reason, also enable the Run Static Analyzer option under the Build Options header, Xcode will now run the analyzer every time we build the app. That makes the builds a bit slower but for an app of this size that’s barely noticeable.

如果大家有什么好的代码规范，欢迎补充。O(∩_∩)O~。