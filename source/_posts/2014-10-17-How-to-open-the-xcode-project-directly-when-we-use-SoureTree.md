title: How to open the xcode project directly when we use SourceTree
date: 2014-10-17 09:32:29
tags: SourceTree
---
More and more projects are controlled by Git, then I start to use [SourceTree](http://www.sourcetreeapp.com) as the Git GUI. So, many times, I like to open SourceTree first, not some forder, not Xcode. Now, there is a question on my table. After I open the SourceTree, I still need to `cd` to my xcode project forder, then open the `.xcodepro` or `.xcodeworkspace` file. It is boring, right?

So, I expect there is a way to simplify the workflow.

Now, it is coming. 

1, created a small automator workflow. There are two workflow files I have written, you can download them in [here](https://github.com/tuchangwei/SoureTreeWorkflow)

![](/2014/10/17/How-to-open-the-xcode-project-directly-when-we-use-SoureTree/QQ20141017-1@2x.png)

2, add `Custom Actions` on SourceTree.

Script to run: automator

Parameters:  -D VariableName=$REPO /Path/To/Workflow.workflow

![](/2014/10/17/How-to-open-the-xcode-project-directly-when-we-use-SoureTree/QQ20141017-2@2x.png)

3, try it!

![](/2014/10/17/How-to-open-the-xcode-project-directly-when-we-use-SoureTree/QQ20141017-3@2x.png)

So, this is all.

Refer to: [http://blog.sourcetreeapp.com/2012/02/08/custom-actions-more-power-to-you/](http://blog.sourcetreeapp.com/2012/02/08/custom-actions-more-power-to-you/)
