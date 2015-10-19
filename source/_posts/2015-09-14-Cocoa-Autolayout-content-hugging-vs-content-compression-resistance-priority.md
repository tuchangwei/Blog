title: "Cocoa Autolayout: content hugging vs content compression resistance priority"
date: 2015-09-14 22:11:15
tags: Autolayout
categories: iOS
---

###Intrinsic Content Size

The intrinsic content size is the size a view prefers to have for a specific content it displays. For example, UILabel has a preferred height based on the font, and a preferred width based on the font and the text it displays. A UIProgressView only has a preferred height based on its artwork, but no preferred width. A plain UIView has neither a preferred width nor a preferred height.

###Compression Resistance and Content Hugging

Each view has content compression resistance priorities and content hugging priorities assigned for both dimensions. These properties only take effect for views which define an intrinsic content size, otherwise there is no content size defined that could resist compression or be hugged.

Behind the scenes, the intrinsic content size and these priority values get translated into constraints. For a label with an intrinsic content size of { 100, 30 }, horizontal/vertical compression resistance priority of 750, and horizontal/vertical content hugging priority of 250, four constraints will be generated:

~~~
H:[label(<=100@250)] 
H:[label(>=100@750)]
V:[label(<=30@250)]
V:[label(>=30@750)]
~~~

If you’re not familiar with the visual format language for the constraints used above, you can read up about it in Apple’s documentation. Keeping in mind that these additional constraints are generated implicitly helps to understand Auto Layout’s behavior and to make better sense of its error messages.

---
Quick summary of the concepts:

* Hugging => content does not want to grow
Compression 
* Resistance => content does not want to shrink

and an example:

Say you've got button like this:

`[       Click Me      ]`

and you've pinned the edges to a larger superview with priority 500.

Then, if Hugging priority > 500 it'll look like this:

`[Click Me]`

If Hugging priority < 500 it'll look like this:

`[       Click Me      ]`

If superview now shrinks then, if the Compression Resistance priority > 500, it'll look like this

`[Click Me]`

Else if Compression Resistance priority < 500, it could look like this:

`[Cli..]`

If it doesn't work like this then you've probably got some other constraints going on that are messing up your good work!

E.g. you could have it pinned to the superview with priority 1000. Or you could have a width priority. If so, this can be helpful:

Editor > Size to Fit Content


