---
layout: post
title: "GCD和信号量"
date: 2013-03-05 21:50
comments: true
categories: iOS GCD
---
 
#GCD
概念不多说，直接上代码。话说也不是什么高深的东东，不过极大简化了代码，一目了然。后面对信号量的记录也采用了相同的原理。
	
	//抛出线程
	dispatch_async(dispatch_get_global_queue(0, 0), ^{
	    
	    NSURL * url = [NSURL URLWithString:@"http://www.google.com"];
	    NSString * data = [NSString stringWithContentsOfURL:url encoding:NSUTF8StringEncoding error:nil];
	    if (data != nil) {
	        //抛出的线程执行完后，回到主线程处理界面逻辑。
	        dispatch_async(dispatch_get_main_queue(), ^{
	            
	      	});
	    } else {
	       	
	    }
	});
#semaphore（信号量）
信号量主要是用在传说中的生产者消费者模式里面，当信号量为0时线程挂起，当信号量大于0时继续向下执行。以前研究过一次，不过没怎么搞懂，反而把界面给搞死了。这次，取相册里的图片，GCD加Semaphore，又折腾了一次，还好。不多说，继续上代码。

	//这几段代码的大概意思是：抛一个线程出来，取相册里的group，再在取group中的线程中抛出线程，去取group中的result，最终到界面线程中去操作。
	//这里需要两个信号量，一个用来阻塞取group的线程，避免界面线程先于取group的线程运行完毕。另外一个用来阻塞取result中的线程，避免取result的
	//线程先于取group执行完毕。
	
	//抛出线程
	dispatch_async(dispatch_get_global_queue(0, 0), ^{
	
		//创建信号量
        dispatch_semaphore_t semaphore1 = dispatch_semaphore_create(0);
        dispatch_semaphore_t semaphore2 = dispatch_semaphore_create(0);
        ALAssetsGroupEnumerationResultsBlock groupEnumerAtion = ^(ALAsset *result, NSUInteger index, BOOL *stop){
        
            if (result == nil) {
                //当某个group取完毕后，信号量加1，dispatch_semaphore_wait方法执行，信号量为0，程序循环，去取下一个group中的result
                dispatch_semaphore_signal(semaphore2);
                
            }else if ([[result valueForProperty:ALAssetPropertyType]isEqualToString:ALAssetTypePhoto]||[[result valueForProperty:ALAssetPropertyType]
	 isEqualToString:ALAssetTypeVideo]) {
            
                FileNode *node = [[FileNode alloc] init];
                node.m_name =  result.defaultRepresentation.filename;
                node.m_path = [result.defaultRepresentation.url absoluteString];
                [self.picList addObject:node];
                NSLog(@"%@",node.m_name);
                NSLog(@"%@",node.m_path);
                [node release];
                
            }
        };
        
        
        ALAssetsLibraryGroupsEnumerationResultsBlock libraryGroupsEnumeration = ^(ALAssetsGroup* group, BOOL* stop){
            
            if (group!=nil)
            {
                
                [group enumerateAssetsUsingBlock:groupEnumerAtion];
                dispatch_semaphore_wait(semaphore2, DISPATCH_TIME_FOREVER);
                
            }else{
                //当所有group取完后，信号量加1，程序不再阻塞，进入界面线程。
                dispatch_semaphore_signal(semaphore1);
            }
            
        };
        
        ALAssetsLibraryAccessFailureBlock failureblock = ^(NSError *error){
            
            
            NSLog(@"failureblock:%@",error);
        };
        
        
        ALAssetsLibrary* library = [[ALAssetsLibrary alloc] init];
        [library enumerateGroupsWithTypes:ALAssetsGroupAll
                               usingBlock:libraryGroupsEnumeration
                             failureBlock:failureblock];
        
        dispatch_semaphore_wait(semaphore1, DISPATCH_TIME_FOREVER);
        dispatch_release(semaphore1);
        dispatch_release(semaphore2);
        dispatch_async(dispatch_get_main_queue(), ^{
            
            [self loadPicAtPage:self.currentPage];
            [self loadPicAtPage:self.currentPage + 1];
        });
    });
    
   
