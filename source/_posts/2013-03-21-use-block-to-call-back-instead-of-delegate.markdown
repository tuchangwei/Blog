---
layout: post
title: "Use Block to call back instead of Delegate"
date: 2013-03-21 22:25
comments: true
categories: GCD
---
Some days ago, i posted the [blog](http://tuchangwei.github.com/blog/2013/03/19/dui-xiang-chuan-zhi-de-ji-chong-fang-shi-some-methods-to-make-talk-to-each-other-between-objects-a-and-b/) to introduce some ways to talk to each oher between Object A and B. One of the points is Blocks. Now I will show how to call back use Block instead of Delegate.

Some codes posted as follow:

	A：SWDPromptViewController
	
	.h
	//declare a block named "dismissPromptViewBlock".
	@property (copy, nonatomic) void(^dismissPromptViewBlock)	(SWDPromptViewController *prompt);

	.m
	//after some operations, we will dismiss A and release it.
	-(void)dismissPrompt
	{
    	if (self.dismissPromptViewBlock)
    	{
       	 self.dismissPromptViewBlock(self);
       	 self.dismissPromptViewBlock = nil;
    	}
	}
	
	
	B：
	.m
	//declare A object and init it,then assign a block to the property of A.
	//so when A executes the method "dismissPrompt", the block plays a part of
	//Delegate.
	SWDPromptViewController *promptViewController = [[SWDPromptViewController 	alloc] initWithNibName:@"SWDPromptViewController" bundle:nil type:type];
    UIWindow *promptWindow = [[UIWindow alloc] initWithFrame:[[UIScreen 	mainScreen] bounds]];
    promptWindow.windowLevel = UIWindowLevelAlert;
    promptWindow.backgroundColor = [UIColor clearColor];
    promptWindow.rootViewController = promptViewController;
    [promptViewController release];
    [promptWindow makeKeyAndVisible];
    
    promptViewController.dismissPromptViewBlock = ^(SWDPromptViewController 	*prompt){
        
        [promptWindow release];
        };
   	 
   	 
   That is all. Welcome to communicate with me.O(∩_∩)O~.