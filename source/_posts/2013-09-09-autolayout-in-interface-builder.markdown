---
layout: post
title: "[译]在IB中实现自动布局"
date: 2013-09-09 20:59
comments: true
categories: Autolayout
---
有关自动布局的其他文章：

* [Autolayout](http://commandshift.co.uk/blog/2013/01/13/autolayout/)
* [Visual format language for autolayout](http://commandshift.co.uk/blog/2013/01/31/visual-format-language-for-autolayout/)
* [Creating individual constraints](http://commandshift.co.uk/blog/2013/02/20/creating-individual-layout-constraints/)

可怜的界面编辑器。它只能服从命令。苹果告诉它，它必须创建最简单的约束集并且简明的传递给你通过拖拽和调整控件大小所形成的布局中。系统约束集留给你的可能是这样的：

<img src="http://static.flickr.com/7422/9710436440_4feab8c718.jpg" width="500" height="205" alt="autolayout_nightmare">


它也不一定非是上面的节奏。

在界面编辑器中快乐写约束的秘密其实很简单：告诉它你想要什么。只是拖拽控件是不够的。

创建一个包含空View的.xib文件(File —> New File —> User Interface —> View)，拖一个label放在View左上方的某个地方。IB将会为你创建一些约束：

<img src="http://static.flickr.com/7429/9711179916_c26ce7fea7.jpg" width="500" height="147" alt="label_top_left">

现在，把这个label移动到右下方：

<img src="http://static.flickr.com/7336/9707197253_d1a4987a28.jpg" width="500" height="430" alt="label_bottom_right">

IB已经丢掉了之前自动创建的约束，重新创建了一组新约束。这个label现在会有不同的表现——比如说，在3.4寸和4寸iPhone屏上，它将会和屏幕底部保持相同的距离，而不是和屏幕顶部。这可能不是你想要的。

再次将label移回到左上角，开始给IB一些帮助。选择这个label，然后点击右边的“size inspector”图标：

<img src="http://farm3.staticflickr.com/2890/9710436938_fe60560e80.jpg" width="500" height="350" alt="size_inspector">

这些影响label的约束出现了。点击这个“Top space to: Superview”约束上的设置图标，选择“Promote to user constraint”。它会改变颜色。兴奋之余，选择同一面板上的“Select and Edit”，把这个固定值改为比如400。这个label将会移动到屏幕底部，但是此时它的位置是相对于View顶部的，而不是底部。

然而，如果你再次拖拽这个label，IB认为你并不关心这个布局，它将会再次为你编造它自己的约束。

{% blockquote %}
	
	不要拖拽视图去调整布局，而是创建适当的约束。
	
{% endblockquote %}

被你提升或者说编辑的约束叫做用户约束，它在文档浏览器里会显示成蓝色，并且会比较粗。通过拖拽被IB自动增加的约束叫做系统约束，比较细一点。这里你能看出两种不同约束的区别：

<img src="http://static.flickr.com/2824/9707197725_e823c0353a.jpg" width="494" height="228" alt="user_and_system_constraints">

拖拽的方法是创建约束的一种方式，但是是最坏的一种方式。因为我们依赖IB为我们做出正确的猜想，什么才是我们想要的。你完全可以控制所有的约束的创建通过“Alignment and Pinning”面板：

{% img left http://static.flickr.com/7404/9710437404_b401a3e7ba.jpg %}

在这个Alignment面板中，如果你在编辑器中没有选择多个视图，那除了最下面的两个选项外，其他的选项都是不可选的。注意在每一种约束前面的小图标，这些小图标很好的描叙了对应约束的作用，它们在查看复杂布局的适合很有用。

{% img right http://static.flickr.com/5499/9710437638_02a608ab37.jpg %}

在这个“Pinning”面板中，这些“Horizontal”，“Vertical Spacing”，“Widths Equally“和“Heights Equally”选项仅仅在选择多个视图的情况下才可使用。再一次，请注意这些描叙性的图标。

“Pinning”和“Alignment”面板创建的约束相对于拖拽得到的约束，给了你更多的控制权。

对于复杂的布局，确保你在早期有一个清晰的想法将是成功的关键。不如说，你有一组label，你想让它们左边界对齐，并且距离父视图有十个像素点。拖拽将会给你正确的布局，但是不会创建对齐的约束：

<img src="http://static.flickr.com/3742/9707198407_af54f82b23.jpg" width="500" height="234" alt="aligned_labels_1">

所有的水平位置的约束都是相对于父视图的。如果你想一起移动这些label，不管是在IB中还是在程序运行中，你都需要改变每一个label的约束。当然，你也可以通过全选这些label，通过这个面板对齐它们的左边界。

{% img http://static.flickr.com/2865/9710438118_606077a97f.jpg"  %}

现在有一个单独的间隔约束指向父视图的左边界。编辑这个约束将会移动所有label的位置。

{% blockquote %}
	
	对于有清晰想法的布局，要通过“alignment and pinning”面板创建约束，而不是通过拖拽创建。
	
{% endblockquote %}

正如我们清楚我们想要什么，我们也必须清楚哪些是我们不想要的。在创建或者提升那些我们关心的约束之后，我们要迅速的移除那些剩余的约束。IB会帮你移除一些，但是你的目的是全部使用你自己创建或提升的用户约束，而不留下系统约束。如果你不这样做，当视图在程序运行中改变，将会因为布局管理器使用那些多余的约束而产生不必要的影响。

{% blockquote %}
	
	为了达到100%的用户约束——在你布局中用到的每一个约束，都要保证是你想要的且理解的那个。
	
{% endblockquote %}

选择，编辑和删除约束可以在三个不同的地方操作：

1. 在文档导航里

	在这里，将你的视图命名是非常有帮助的。约束可以在它应用的视图下面找到。注意，一个视图的位置是一个在它父视图上面约束（因为这个约束影响frame），视图的大小则是一个在它自身上面的约束（因为这个约束影响bounds）。
	
	{% blockquote %}
		
		在IB中将视图命名对于约束很有帮助。也可以在“identity inspector”中的“Label”处命名。
		
	{% endblockquote %}

2. 在布局中

	这需要技巧，依赖于展示约束的数量。首先，点击你感兴趣的视图，然后点击那一两个像素宽的指示器。

3. 在“size inspector”中指定的部分

	这一列包含大小约束和位置约束。“Select and edit”选项和在其他地方选择约束，打开其“attributes inspector”一样。
	
	在约束的“attributes inspector”选项中，你能够编辑它的优先级。对于大小约束和间隔约束，有小于等于，等于，大于等于和常量等关系。
	
通过按照我在文章中突出强调的关键点，你现在应该能够通过IB管理你的约束了。作为总结：

* 不要通过拖拽视图去调整布局，而是编辑合适的约束。
* 有清晰的目的，使用“alignment and pinning”面板去创建约束而不是通过拖拽。
* 为了达到100%的用户约束——在你布局中用到的每一个约束，都要保证是你想要的且理解的那个。
* 在IB中将视图命名对于约束很有帮助。也可以在“identity inspector”中的“Label”处命名。


翻译出处：[Autolayout in Interface Builder](http://commandshift.co.uk/blog/2013/01/18/autolayout-in-interface-builder/)






