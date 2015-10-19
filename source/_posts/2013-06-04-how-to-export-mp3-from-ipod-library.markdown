---
layout: post
title: "How to export mp3 from ipod-library"
date: 2013-06-04 22:33
comments: true
categories: iOS ipod-library
---

Someday, our leader let me export videos, music and photos from Video, Music and Photos in iPad or iPhone. After looked up some articles, I found it was difficult to export mp3.

But, Nowadays, it came true to export mp3, so every format of ipod-library files can be exported. 

Now, See how to do it.

* first, import some frameworks, such as `MobileCoreServices.framework`,`MediaPlayer.framework`,`AVFoundation.framework`
* second, query the media from ipod-library.

		//request videos and music come from system's Videos and Music 
		NSNumber *videoTypeNum = [NSNumber numberWithInteger:MPMediaTypeAny];
		
		MPMediaPropertyPredicate *videoPredicate = [MPMediaPropertyPredicate predicateWithValue:videoTypeNum forProperty:MPMediaItemPropertyMediaType];
		
		MPMediaQuery *videoQuery = [[MPMediaQuery alloc] init];
		[videoQuery addFilterPredicate: videoPredicate];
		_mediaItems = [[videoQuery items] copy];
	    
	    	    
*third, convert MPMediaItem to AVURLAsset, generate file name and file extension, then start export,the format of exported file is "MOV".Finally, i need use NSFileManager rename the mov to mp3.
	
	 MPMediaItem *mediaItem = [_mediaItems objectAtIndex:_count];
	 
	 //get the name of the file.
	 NSString *songTitle = [mediaItem valueForProperty: MPMediaItemPropertyTitle];
	 
	 //convert MPMediaItem to AVURLAsset.
	 AVURLAsset *sset = [AVURLAsset assetWithURL:[mediaItem valueForProperty:MPMediaItemPropertyAssetURL]];
	 
	//get the extension of the file.
	NSString *fileType = [[[[sset.URL absoluteString] componentsSeparatedByString:@"?"] objectAtIndex:0] pathExtension];
	
	//init export, here you must set "presentName" argument to "AVAssetExportPresetPassthrough". If not, you will can't export mp3 correct.
	AVAssetExportSession *export = [[AVAssetExportSession alloc] initWithAsset:sset presetName:AVAssetExportPresetPassthrough];
	
	NSLog(@"export.supportedFileTypes : %@",export.supportedFileTypes);
	//export to mov format.
	export.outputFileType = @"com.apple.quicktime-movie";
	
	export.shouldOptimizeForNetworkUse = YES;
	
	NSString *extension = ( NSString *)UTTypeCopyPreferredTagWithClass(( CFStringRef)export.outputFileType, kUTTagClassFilenameExtension);
	
	NSLog(@"extension %@",extension);
	NSString *path = [NSHomeDirectory() stringByAppendingFormat:@"/Documents/%@.%@",songTitle,extension];
	
	NSURL *outputURL = [NSURL fileURLWithPath:path];
	export.outputURL = outputURL;
	[export exportAsynchronouslyWithCompletionHandler:^{
	         
	   if (export.status == AVAssetExportSessionStatusCompleted)
	     {
	        //then rename mov format to the original format.
	         NSFileManager *manage = [NSFileManager defaultManager];
	         
	         NSString *mp3Path = [NSHomeDirectory() stringByAppendingFormat:@"/Documents/%@.%@",songTitle,fileType];
	         
	         NSError *error = nil;
	         
	         [manage moveItemAtPath:path toPath:mp3Path error:&error];
	         
	         NSLog(@"error %@",error);
	             
	      }
	      else
	      {
	         NSLog(@"%@",export.error);
	      }
	      
	}];


Some points you need notice:

* when you init AVAssetExportSession object, the "presetName" argument must be "AVAssetExportPresetPassthrough".
* The "outputFileType" may be "com.apple.quicktime-movie".

So easy, doesn't it? Good night.



	
	


