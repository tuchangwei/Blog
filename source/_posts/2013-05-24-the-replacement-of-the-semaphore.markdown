---
layout: post
title: "The replacement of the semaphore"
date: 2013-05-24 22:49
comments: true
categories: GCD iOS
---
In order to synchronize between threads, i used semaphore in [here](http://tuchangwei.github.io/blog/2013/03/05/gcdhe-xin-hao-liang/). Today, I find a replacement of the semaphore. I think it is more efficient and cool. Now, i note it as follow:

		- (id)initWithLibraryChangedHandler:(void (^)(void))libraryChangedHandler
	{
	    self = [super init];
	    if (self) 
		{
			_assetItems = [NSMutableArray array];
	        //Creates a new dispatch queue to which blocks can be submitted.  创建一个新的分发队列用于提交blocks
			_assetItemsQueue = dispatch_queue_create("com.apple.avmovieexporter.assetItemLibraryQueue", DISPATCH_QUEUE_SERIAL);
			
			_libraryGroup = dispatch_group_create();
	        //Returns a well-known global concurrent queue of a given priority level.
	        //The well-known global concurrent queues cannot be modified
	        //Blocks submitted to these global concurrent queues may be executed concurrently with respect to each other.
			_libraryQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0);
			
			// Update the table view whenever the library changes
			[[NSNotificationCenter defaultCenter] addObserverForName:ALAssetsLibraryChangedNotification 
															  object:nil 
															   queue:[NSOperationQueue mainQueue] 
														  usingBlock:^(NSNotification *block){
																   libraryChangedHandler();
															   }];
	    }
	    
	    return self;
	}
	
	- (void)dealloc
	{
		dispatch_release(_assetItemsQueue);
		
		dispatch_release(_libraryQueue);
		dispatch_release(_libraryGroup);
		
		[[NSNotificationCenter defaultCenter] removeObserver:self name:ALAssetsLibraryChangedNotification object:nil];
	}
	
	- (void)loadLibraryWithCompletionBlock:(void (^)(void))completionHandler
	{
		// Load content using the Media Library and AssetLibrary APIs, also check for content included in the application bundle
		[self.assetItems removeAllObjects];
		
		[self buildMediaLibrary];
		[self buildAssetLibrary];
		[self buildApplicationBundleLibrary];
		
	    
	    //Schedules a block object to be submitted to a queue when a group of previously submitted block objects have completed.
		dispatch_group_notify(self.libraryGroup, self.libraryQueue, ^{
			dispatch_async(dispatch_get_main_queue(), ^{
				completionHandler();
			});
		});
	}
	
	- (void)addURL:(NSURL *)url
	{
		__unsafe_unretained __block VideoLibrary *weakSelf = (VideoLibrary *)self;
		
		if (url == nil)
			return;
		
		dispatch_async(self.assetItemsQueue, ^{
			[weakSelf.assetItems addObject:[[AssetItem alloc] initWithURL:url]];
		});
	}
	
	#pragma mark - iPod Library
	
	- (void)buildMediaLibrary
	{
		__unsafe_unretained __block VideoLibrary *weakSelf = (VideoLibrary *)self;
	    
	    //将blocks绑定到libraryGroup，然后放在libraryQueue并发队列中
		dispatch_group_async(self.libraryGroup, self.libraryQueue, ^{
			NSLog(@"started building media library...");
			
			// Search for video content in the Media Library
	#if  __IPHONE_OS_VERSION_MAX_ALLOWED >= 50000
			NSNumber *videoTypeNum = [NSNumber numberWithInteger:MPMediaTypeAnyVideo];
	#else
			NSNumber *videoTypeNum = [NSNumber numberWithInteger:(MPMediaTypeAny ^ MPMediaTypeAnyAudio)];
	#endif
			MPMediaPropertyPredicate *videoPredicate = [MPMediaPropertyPredicate predicateWithValue:videoTypeNum forProperty:MPMediaItemPropertyMediaType];
			MPMediaQuery *videoQuery = [[MPMediaQuery alloc] init];
			[videoQuery addFilterPredicate: videoPredicate];
			NSArray *items = [videoQuery items];
			
			for (MPMediaItem *mediaItem in items) 
				[weakSelf addURL:[mediaItem valueForProperty:MPMediaItemPropertyAssetURL]];
			
			NSLog(@"done building media library...");
		});
	}
	
	- (void)buildAssetLibrary
	{
		NSLog(@"started building asset library...");
		
		__unsafe_unretained __block VideoLibrary *weakSelf = (VideoLibrary *)self;
		dispatch_group_enter(weakSelf.libraryGroup);
		
		ALAssetsLibrary *assetLibrary = [[ALAssetsLibrary alloc] init];
		
		// Enumerate through all the groups in the Asset Library
		[assetLibrary enumerateGroupsWithTypes:ALAssetsGroupAll 
									usingBlock:
		 ^(ALAssetsGroup *group, BOOL *stop)
		 {
			 if (group != nil)
			 {
				 // Filter by groups that contain video
				 [group setAssetsFilter:[ALAssetsFilter allVideos]];
				 [group enumerateAssetsUsingBlock:
				  ^(ALAsset *asset, NSUInteger index, BOOL *stop)
				  {
					  if (asset)
						  [weakSelf addURL:[[asset defaultRepresentation] url]];
				  }];
			 }
			 else
			 {
				 dispatch_group_leave(weakSelf.libraryGroup);
				 NSLog(@"done building asset library...");
			 }
		 }
								  failureBlock:^(NSError *error)
		 {
			 dispatch_group_leave(weakSelf.libraryGroup);
			 NSLog(@"error enumerating AssetLibrary groups %@\n", error);
		 }];
		
	}
	
The above is used for understanding the whole plot, now you can see the detail:

	dispatch_group_notify(self.libraryGroup, self.libraryQueue, ^{
			dispatch_async(dispatch_get_main_queue(), ^{
				completionHandler();
			});
		});
	}
	
The explain of the [document](https://developer.apple.com/library/ios/#documentation/Performance/Reference/GCD_libdispatch_Ref/Reference/reference.html) for the selector is that "Schedules a block object to be submitted to a queue when a group of previously submitted block objects have completed." So, we can wait the tasks to complete, then to do something we want. The `dispatch_group_enter` is cool as well. it can let the block enter the queue and execute as a task associated group.

So, that is all. See you.
