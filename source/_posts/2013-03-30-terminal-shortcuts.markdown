---
layout: post
title: "Terminal Shortcuts"
date: 2013-03-30 16:51
comments: true
categories: Utility
---
###mac/linux终端光标的快捷键操作

####常用的快捷键：
* Ctrl + d        删除一个字符，相当于通常的Delete键（命令行若无所有字符，则相当于exit；处理多行标准输入时也表示eof）
* Ctrl + h        退格删除一个字符，相当于通常的Backspace键
* Ctrl + u        删除光标之前到行首的字符
* Ctrl + k        删除光标之前到行尾的字符
* Ctrl + c        取消当前行输入的命令，相当于Ctrl + Break
* Ctrl + a        光标移动到行首（Ahead of line），相当于通常的Home键
* Ctrl + e        光标移动到行尾（End of line）
* Ctrl + f        光标向前(Forward)移动一个字符位置
* Ctrl + b        光标往回(Backward)移动一个字符位置
* Ctrl + l        清屏，相当于执行clear命令
* Ctrl + p        调出命令历史中的前一条（Previous）命令，相当于通常的上箭头
* Ctrl + n        调出命令历史中的下一条（Next）命令，相当于通常的上箭头
* Ctrl + r        显示：号提示，根据用户输入查找相关历史命令（reverse-i-search）

####次常用快捷键：
* Alt + f         光标向前（Forward）移动到下一个单词
* Alt + b         光标往回（Backward）移动到前一个单词
* Ctrl + w        删除从光标位置前到当前所处单词（Word）的开头
* Alt + d         删除从光标位置到当前所处单词的末尾
* Ctrl + y        粘贴最后一次被删除的单词

以上就是快捷键了，在这里我只想揭露一些事情，这些快捷键都是emacs的快捷键！熟悉终端的同学应该知道，终端里可以设置快捷键的类型是vim还是emacs。当然了大多数的终端都用的emacs，因为啥自己想去吧。

在这里之所以又将这些东西提出来的原因是：mac os x虽然是图形界面，但是chrome，xcode啥的也支持这些快捷键！

摘自：http://blog.cnrainbird.com/index.php/2012/03/23/mac_linux_zhong_duan_guang_biao_de_kuai_jie_jian_cao_zuo/