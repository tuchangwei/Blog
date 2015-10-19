---
layout: post
title: "How to import one project to another one in Xcode."
date: 2013-09-16 21:42
comments: true
categories: Sub-Project
---
Due to user needs, I have to import one project to another one as a sub-project. The final result should be like the following picture.

{% img http://static.flickr.com/3803/9773083175_1c842daf5c.jpg %}

It is not easy to do this. We need some steps.

* Step 1, Add "iPadBrakeTester.xcodeproj" file to BusinessBaseApp project, note that don't choose the "Copy items into…" checkbox.

{% img http://static.flickr.com/5458/9778845934_895459e499.jpg %}

* Step 2, close the iPadBrakeTester project, because you have opened the project in BusinessBaseApp project. If you don't, you can just see the "iPadBrakeTester.xcodeproj" file and there is no triangle front of it, you will can't expand it.

* Step 3, generate static library. Choose "iPadBrakeTester.xcodeproj",add Target. After generating static library, add .m, .h and .framework files to it. 

{% img http://static.flickr.com/3723/9779659835_52fc37d5b6.jpg %}  
 
{% img http://static.flickr.com/7373/9779677046_8d37cdc81b.jpg %}

* Step 4, generate Bundle, and change the relevant settings to iOS target.
	
{% img http://static.flickr.com/5337/9779800343_4fac8e3d22.jpg %} 

{% img http://static.flickr.com/7311/9779854353_1698045cb4.jpg %}   

Then, add .xib and iamge files to it, like this:

{% img http://static.flickr.com/3743/9779815366_2d5c974210.jpg %}                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                

* Step 5, Use static library and Bundle in BusinessBaseApp project.

{% img http://static.flickr.com/7306/9779890825_2a8960475a.jpg %}

"Target Dependencies" means that it needs to rebuild the app object code if code in the dependency changes.

In "Link Binary With Libraries",beside adding the static library, you need add the .framework files which you import to the sub-project.

* Step 6, add "-ObjC" and "-all_load" flags to "Other Linker Flags".

	**-ObjC** because otherwise the linker does not properly load your classes.
	
	**-all_load** if you have categories in the library, because otherwise those are not loaded.
	
* Step 7, add the sub-project path to "User Header Search Paths", like this:

{% img http://static.flickr.com/2843/9780044276_76d5679574.jpg %}

* Step 8, use Bundle in code.

{% codeblock lang:objc %}

#pragma mark - Setup Views.
- (void)setupContentView
{
     NSBundle *bundle = [NSBundle bundleWithURL:[[NSBundle mainBundle] URLForResource:@"iPadBrakeTesterResources" withExtension:@"bundle"]];
    self.settingsViewController = [[TBrakeTestSettingViewController alloc] initWithNibName:@"TBrakeTestSettingViewController" bundle:bundle];
    
    self.settingsViewController.view.frame = self.view.frame;
    [self addChildViewController:self.settingsViewController];
    [self.view addSubview:self.settingsViewController.view];
    [self.settingsViewController didMoveToParentViewController:self];

    self.brakeTestViewController = [[TBrakeTestViewController alloc] initWithNibName:@"TBrakeTestViewController_Old" bundle:bundle];
    self.brakeTestViewController.view.frame = self.view.frame;
    [self addChildViewController:self.brakeTestViewController];
    [self.view addSubview:self.brakeTestViewController.view];
    [self.brakeTestViewController didMoveToParentViewController:self];

    
    
}
{% endcodeblock %}


Some references:

[Sub-Projects in Xcode](http://www.cocoanetics.com/2011/12/sub-projects-in-xcode/)

[iOS Library With Resources](http://www.galloway.me.uk/tutorials/ios-library-with-resources/)





