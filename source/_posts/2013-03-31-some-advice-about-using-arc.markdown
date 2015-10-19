---
layout: post
title: "Some advice about Using ARC"
date: 2013-03-31 21:46
comments: true
categories: ARC
---

1. As long as there is a variable pointing to an object, that object stay in memory. When the pointer gets a new value or ceases to exist, the associated object is released.This is true for all variables:instance variables,synthesized properties, adn even local variables.
2. "weak" pointer. Variables that are weak can still point to objects but they do not become owners.
3. Please change the `Complier for C/C++/Object-c` option to `Apple LLVM compiler 3.0`.
4. Please set `Other Waring Flags` option to `-Wall`.
5. Please set `Run Static Analyzer` option to `Yes`.
6. Please set `Objective-c Automatic Reference Counting` option to `Yes`.
7. Some files needn't to convert to ARC support, use the `-fno-objc-arc` flag.
8. You will get this error when your code does the following:
	
		switch (X)
		{	
		    case Y:
		        NSString *s = ...;
		        break;
		}
	you need do this:
	
		switch (X)
		{	
		   case Y:
		   {
		       NSString *s = ...;
		       break;
		   }
		}

9. If instace variables are a part of the internals of your class and not something you want expose in its public interface. you can move the instace variables from interface section to implementation section. some codes from .h file as following:
	
	
		#import <UIKit/UIKit.h>
		 
		@interface MainViewController : UIViewController 
		  <UITableViewDataSource, UITableViewDelegate, UISearchBarDelegate, 
		  NSXMLParserDelegate>
		 
		@property (nonatomic, retain) IBOutlet UITableView *tableView;
		@property (nonatomic, retain) IBOutlet UISearchBar *searchBar;
		 
		@end
		
	move the instace variables to implementation section as following:


		@implementation MainViewController
		{
			NSOperationQueue *queue;
			NSMutableString *currentStringValue;
		}

10. Generally, dealloc method isn't need.The only reason for keeping a dealloc method around is when you need to free certain resources that do not fall under ARC’s umbrella. Examples of this are calling CFRelease() on Core Foundation objects, calling free() on memory that you allocated with malloc(), unregistering for notifications, invalidating a timer, and so on.
11. Toll-Free Bridging

 Change ownership from Core Foundation to Objecttive-C, use CFBridgingRelease().
 
 Change ownership from Objective-C to Core Foundation, use CFBridgingRetain().
 
 Want to use one type temporarily as if it were another without owership change, use __bridge.
 


