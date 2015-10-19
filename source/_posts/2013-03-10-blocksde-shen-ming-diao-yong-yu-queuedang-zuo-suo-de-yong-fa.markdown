---
layout: post
title: "Blocks的申明调用与Queue当做锁的用法"
date: 2013-03-10 11:40
comments: true
categories: GCD
---

###Blocks的申明与调用
话说Blocks在方法内使用还是挺方便的，之前都是把相同的代码封装成外部函数，然后在一个方法里需要的时候调用，这样挺麻烦的。使用Blocks之后，我们可以把相同代码在这个方法里封装起来，然后再在这个方法中需要的地方直接调用，逻辑清晰，操作也不会那么繁琐。

上代码：
	
	//在方法体内
	//block申明与初始化
	 void(^removePicViewBlock)(int,int) = ^(int start,int stop){
        
        while (start <= stop)
        {
            NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
            SWDPictureView *view = (SWDPictureView *)[self.scroller viewWithTag:start + 1];
            view.imageView.image = nil;
            view.imageView = nil;
            view.delegate2 = nil;
            [view removeFromSuperview];
            start ++;
            [pool drain];
        }
        
    
    };
    if (page < self.currentPage)
    {
    	//block的调用
        removePicViewBlock(0,page);
    }
    else if(page > self.currentPage)
    {
    	//block的调用
        removePicViewBlock(page,[self.picList count]-1);
    }
    
    
###Queue作为锁的用法

说到多线程就必须要提到锁了，话说用锁也蛮麻烦，需要在初始化的时候申明一个锁，然后每个方法的开头加一把锁，结尾再把锁去掉。如果用queue就不必那么麻烦了。根据task在queue中FIFO的特性就可以防止多线程中资源被同时访问。

上代码。

	//初始化时初始化一个queue
	UserQueue = dispatch_queue_create("use for storing image to disk", nil);
	
	//多线程调用时，将任务放在queue中执行。此时queue充当锁的角色。
	//存储图片缓冲到硬盘
	-(void)storeImage:(UIImage *)image key:(NSString *)key
	{
	    dispatch_async(UserQueue, ^{
	        
	        NSFileManager *fileManager = [[NSFileManager alloc] init];
	        BOOL succ = [fileManager createFileAtPath:[self cachePathForKey:key] contents:UIImageJPEGRepresentation(image, (CGFloat)0.6) attributes:nil];
	        NSLog(@"store successfully!!! %d",succ);
	        [fileManager release];
	        fileManager = nil;
	    });
	     
	}
	
好了，就这些。