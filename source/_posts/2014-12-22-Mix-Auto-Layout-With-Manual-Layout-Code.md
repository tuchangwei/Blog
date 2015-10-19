title: Mix Auto Layout With Manual Layout Code
date: 2014-12-22 21:09:01
tags: AutoLayout
---

1. Set `translates​Autoresizing​Mask​Into​Constraints = NO` on the view since you will position and size it manually, anyway :)
2. Override `layoutSubviews` (or `viewDidLayoutSubviews` if you want to do this in a view controller), call super (which performs the Auto Layout step) and then perform any manual layout we want to add