title: "How to understand the '!' in the end of IBOutlet Properties in Swift"
date: 2014-12-19 10:46:29
tags: Swift
---
Let us see the follow code:

```Swift
@IBOutlet var mapView: MKMapView!
```
As we all know, the `mapView` is a variable from Interface Builder, but why there is a `!` behind the property, let's analyze it.

*** Firstly, Let's ask what is the meaning of `!` behind of variable? ***

It means the variable is an optional type, further, it is an implicityly unwrapped optional which  we don't need unwrap and use directly.
	
*** Then, why we need an optional type here? ***
	
Because of a rule. We know the initializer function in Swift must initialize all of properties which are not optional type and Swift doesn't "know" that Interface Builder is supplying the views at run time.
