title: ［译］Android Handler 中的内存泄漏
date: 2015-06-24 09:59:24
tags: Android
categories: Handler
---

Android使用Java作为其开发平台。这帮助我们解决许多低层次的问题，包括内存管理，平台类型依赖（platform type dependencies，怎么翻译？）等等。但是，我们有时候仍然会由于内存泄漏而导致程序崩溃。 那么，垃圾收集器哪里去了呢？这些大对象长时间不能被清理的案例，我将关注其中一种。这种案例最终不会产生内存泄漏——对象在某些时刻都能够被回收——所以我们有时会忽略它。但是这并不是明智的做法，因为有些时候它会导致OOM的错误。我正在描叙的这种案例是 Handler leak，这种案例常常被Lint作为一个警告检测到。##简单的例子

{% asset_img anonymous_runnable_code.png %}

这是一个非常简单的activity。注意这个匿名的 `Runnable` 被这个 `Handler` 发出伴随着一个非常长的延迟。我们运行它并且旋转手机多次，然后查看内存并且分析它。

{% asset_img anonymous_runnable_memory_analyze.png %}

我们现在有7个activities在内存中。这显然不太好。让我们找出为什么GC不能清理掉他们。

>我做的这个查询去得到仍然存在于内存中的Activities是用OQL（Object Query Language）创建的，它很简单，也很强大。

{% asset_img anonymous_runnable_memory_explained.png %}

正如你看到的那样，其中一个activity被 `this$0` 引用。这是一个匿名类对宿主类的间接引用。`this$0` 被 `callback` 对象引用，然后 `callback` 又被一连串的 `Message` 的对象 `next` 引用回到主线程。

>任何时候你在宿主类里面创建一个非静态类，Java都会为宿主对象创建一个间接引用到这个非静态类对象。

一旦你在 `Handler` 里发送一个 `Runnable` 或者 `Message`， 它都会存储这些来自 `LooperThread` 的 `Message` 命令的引用到列表中，直到这个message被执行。发送一些延迟的messages，至少在延迟的这段时间，是一个明显的泄漏。 发送没有延迟的也可能引起短暂的泄漏如果队列里的messages很大的话。

##静态 Runnable 对象的解决方案
让我们试着克服一个内存泄漏通过摆脱 `this$0`, 通过把匿名类转为静态的。

{% asset_img StaticClass_code.png %}

跑跑，旋转，看看内存树。

{% asset_img StaticClass_memory_analyze.png %}

啥情况？让我们看看谁持有了 `Activities` 的引用。

{% asset_img StaticClass_memory_analyze_explained.png %}

看看引用树的底部 -- activity被 `DoneRunnable` 类里的`mTextView` 中的 `mContext` 引用了。使用内部静态类并不足够解决掉内存泄漏， 我们需要再多做一点。

##带有WeakReference对象的静态 Runnable

让我们继续使用迭代的修复方式并且摆脱掉对TextView的引用，这个引用阻止了activity的销毁。

{% asset_img StaticClassWithWeakRef_code.png %}

注意，我们保持了一个对TextView的弱引用，然后跑一跑，旋转并且看下内存树。

>要小心弱引用。它们随时都可能变为null，所以解决它们的第一步是把它赋值给一个强引用并且在使用它时检测它是不是为空（这个和iOS类似啊）。

{% asset_img StaticClassWithWeakRef_memory_analyze.png %}

哈哈哈，仅仅一个activity实例了，这解决了我们内存的问题。到目前为止我们用到途径是：* 使用静态内部类（或者外部类）。
* 对于被Handler/Runnable操纵的对象用弱引用。如果你比较这个代码和前面的代码，你可能会发现有很大的不同在可读性和简洁度方面。前面的代码比较短比较简洁，很直观。写这些样板文件是很乏味的事情，特别是把 `postedDelayed` 设置为50ms。有一些更好更简洁的解决方案。##在 OnDestroy 方法中清除所有 Messages

Handler类有一个很有趣的特性 -- `remvoeCallBacksAndMessages` -- 这个方法允许 `null` 作为参数。它将移除所有的被特定handler发出的 `Runnables` 和 `Messages` 对象。让我们使用它在 `OnDestroy` 回调方法中。

{% asset_img removeCallbacks_code.png %}

让我们跑一跑，旋转并且看下内存树。

{% asset_img removeCallbacks_memory_analyze.png %}

不错，只有一个对象。
这种方式比前一种方式要好，因为它保持代码干净又可读。唯一要记住的是清除所有Messages在 `activity`/`fragment` destroy方法中。
如果你像我一样懒， 我还有另外一种解决办法可能你会更喜欢。:)

##使用 WeakHandler
......


更多精彩，请阅读原作者[博客](https://techblog.badoo.com/blog/2014/08/28/android-handler-memory-leaks)