---
layout: post
title: "How should we use NSLog"
date: 2013-03-26 00:30
comments: true
categories: NSLog
---
As we all know, every programming language has a selector to print the value on the Console which we want to. This way to debug is too violence, too simple and underestimated by someone. Other way is setting breakpoints, after application runs, it will stop when meets a breakpoint. Someone like this way, he need not see the Console and restart the application again and again to print. 

But i want to say, unitl now, i really have been tend to print value. Because  when some mulitithreadings run at the same time, it is difficult to track the breakpoint. However other way is easy just as “History don't lie”. We can print the system time,the line of printing, the selector, the value of some object，the class as so on.

So,let us see the follow codes.

	#if DEBUG
	#define NSLog(FORMAT, ...) {\
	NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];\
	[dateFormatter setDateStyle:NSDateFormatterMediumStyle];\
	[dateFormatter setTimeStyle:NSDateFormatterShortStyle];\
	[dateFormatter setDateFormat:@"HH:mm:ss:SSSSSS"]; \
	NSString *str = [dateFormatter stringFromDate:[NSDate date]];\
	[dateFormatter release];\
	fprintf(stderr,"[--%s--]*[--%s--]*[--%s:%d--]\n",[str UTF8String], 	[[NSString stringWithFormat:FORMAT, ##__VA_ARGS__] UTF8String],[[[NSString 	stringWithUTF8String:__FILE__] lastPathComponent] UTF8String], __LINE__);\
	}
	#else
	#define NSLog(FORMAT, ...) nil
	#endif 
	
This is a macro definition. You can copy this to your CommonHead.h file and import the 	CommonHead.h to your .pch file so that you can use it at anywhere in your project. The macrodefinition is great, when you use debug mode to print value, you will find the value, the system time, the line and the object where the selector in, yet using the release mode, you will see nothing. The reason is as follow:

<a href="http://www.flickr.com/photos/tuchangwei/8590218136/" title="Flickr 上 涂 涂 的 0F804C40-EA0B-42C5-8058-7E3230E01A13"><img src="http://static.flickr.com/8108/8590218136_2798079941.jpg" width="500" height="237" alt="0F804C40-EA0B-42C5-8058-7E3230E01A13"></a>

Why we need print the line of the selector? Yes, you guess it, press "cmd" + "i" when using Xcode, you will find the selector right now!