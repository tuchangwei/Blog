---
layout: post
title: "Avoid retain cycle in block"
date: 2013-04-10 23:08
comments: true
categories: GCD
---
Let us make long story short. Please see the codes as following:

	- (IBAction)didTapUploadButton:(id)sender
	{
	  NSString *clientID = @"YOUR_CLIENT_ID_HERE";
	 
	  NSString *title = [[self titleTextField] text];
	  NSString *description = [[self descriptionTextField] text];
	 
	  [[UIApplication sharedApplication] setNetworkActivityIndicatorVisible:YES];
	 
	  __weak MLViewController *weakSelf = self;
	dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
	    UIImage *image = [UIImage imageNamed:@"balloons.jpg"];
	    NSData *imageData = UIImageJPEGRepresentation(image, 1.0f);
 
    [MLIMGURUploader uploadPhoto:imageData 
                           title:title 
                     description:description 
                   imgurClientID:clientID completionBlock:^(NSString *result) {
      dispatch_async(dispatch_get_main_queue(), ^{
        [[UIApplication sharedApplication] setNetworkActivityIndicatorVisible:NO];
        [[weakSelf linkTextView] setText:result];
      });
    } failureBlock:^(NSURLResponse *response, NSError *error, NSInteger status) {
      dispatch_async(dispatch_get_main_queue(), ^{
      [[UIApplication sharedApplication] setNetworkActivityIndicatorVisible:NO];
      [[[UIAlertView alloc] initWithTitle:@"Upload Failed"
                                  message:[NSString stringWithFormat:@"%@ (Status code %d)", 
                                                   [error localizedDescription], status]
                                 delegate:nil
                        cancelButtonTitle:nil
                        otherButtonTitles:@"OK", nil] show];
      });
    }];
 
	  });
	}

This is a method for uploading images to [imgur](http://imgur.com/). In the section we can find the codes `__weak MLViewController *weakSelf = self;` and use it like `[[weakSelf linkTextView] setText:result];`. So, we have a question: Why we don‘t call self directly,but rather give a weak pointer to point it and use the weak pointer in the block section?

It is what i will say. As we all know, the property of blocks is copy, and the property of self is retain. So, if we use self in the block section, we will get a retain circle. We will never release them. For this reason, we need do it as above.

[Here](http://blog.random-ideas.net/?p=160) is one reference 

Ok, Good night. See you. 