---
layout: post
title: "How to customize the UI of MPMoviePlayerViewController"
date: 2013-04-06 17:18
comments: true
categories: MPMoviePlayerViewController
---
As we all know, the user interface of MPMoviePlayerViewController is ugly. Sometimes, our customer needs a better UI, so let's do it.

Some codes like this:
	
	MPMoviePlayerViewController *player = [[MPMoviePlayerViewController alloc] init];
        player.moviePlayer.contentURL = self.url;
        //here, i set the controlStyle equal to MPMovieControlStyleNone, as a result, we only see the movie view, yet volume button, progress slider, play button as so on.
        player.moviePlayer.controlStyle = MPMovieControlStyleNone;
        player.moviePlayer.view.frame = self.playerViewController.view.frame;
        //next, i create a UIViewController named playerViewController, and insert the moverPlay view to the view of playerViewController.
        [self.playerViewController.view insertSubview:player.moviePlayer.view atIndex:0];
        [player.moviePlayer play];
        [self presentModalViewController:self.playerViewController animated:YES];

after do that, we need add a view to playerViewController, and set the view is transparent	, we can add custom button on the view to control the move playing. The effect as below:

<a href="http://www.flickr.com/photos/tuchangwei/8624399482/" title="Flickr 上 涂 涂 的 Screenshot 2013.04.03 09.32.26"><img src="http://static.flickr.com/8111/8624399482_3801facaf7.jpg" width="500" height="375" alt="Screenshot 2013.04.03 09.32.26"></a>
