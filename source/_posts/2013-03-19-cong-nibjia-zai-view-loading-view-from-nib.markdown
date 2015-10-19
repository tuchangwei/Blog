---
layout: post
title: "从Nib加载View(loading view from nib)"
date: 2013-03-19 22:15
comments: true
categories: nib
---
话说，有些View界面控件比较多，还是用nib拖拖拉拉方便。但是麻烦的一点是，我们要将nib与View对应的类进行连接，然后使用这个View的地方通过nib进行对象反序列化。我们一般用下面的语句进行反对象序列化。
	
	self.aboutView = [[[NSBundle mainBundle]loadNibNamed:@"AboutView" owner:self options:nil] objectAtIndex:0];
    [self.view addSubview:self.aboutView];
    
    
每次都写如上的代码有些麻烦，其实我们可以为UIView创建一个category:

	@implementation UIView (MHExtensions)
 
	+ (id)mh_viewFromNibNamed:(NSString *)nibName owner:(id)ownerOrNil
	{
		NSArray *nib = [[NSBundle mainBundle] loadNibNamed:nibName owner:ownerOrNil options:nil];
		for (id object in nib)
		{
			if ([object isKindOfClass:[self class]])
				return object;
		}
		return nil;
	}
 
	@end
	
这样每次反序列化就只需要调用 `+ (id)mh_viewFromNibNamed:(NSString *)nibName owner:(id)ownerOrNil` 这个方法即可。