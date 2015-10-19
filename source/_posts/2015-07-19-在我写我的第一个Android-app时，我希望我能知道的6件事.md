title: 在我写我的第一个Android app时，我希望我能知道的6件事
date: 2015-07-19 20:04:21
tags: Android-6-Things
categories: Android
---
我的第一个app是糟糕的。实际上，它太糟糕了，所以我把它从商店中移除，我甚至不再因我把它列在我的简历之中而烦恼。如果我在写它之前，知道一些有关Android开发的事，那个app就不会那么糟糕。

当你写你的第一个Android app时，这有一个列表你需要记着。这些经验教训是我在写第一个app源码时，犯下的真实的错误。我将会在后面展示这些错误。记住这些事情能帮助你写一个让你感到有点自豪的app。

当然，如果你作为一个Android开发的学生正在做正确的工作，无论如何，不久后你都有可能恨你写的app，正如@codestandards所说，如果你一年前码的代码并不能让你感觉到不适，那你有可能没怎么学习。

如果你是一个有经验的Java开发者， 第1，2和第5条可能并不能让你感兴趣。另外的第3，4条可能展示你一些很酷的但你可能还不知道的东西。

###对`Contexts`不要有静态引用
```
public class MainActivity extends LocationManagingActivity implements ActionBar.OnNavigationListener,
        GooglePlayServicesClient.ConnectionCallbacks,
        GooglePlayServicesClient.OnConnectionFailedListener {
            
    //...
 
    private static MeTrackerStore mMeTrackerStore; 
    
    //...
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        //...
        
        mMeTrackerStore = new MeTrackerStore(this);
    }
}
```
这可能看起来对任何人来说，都是一个不可能犯的错误。但并非这样，我就犯过。我也看到其他人犯过这样的错误，并且我也看到很多人并不能在第一时间指出为什么这是一个错误。不要这样做。它是一个`noob move`

如果当这个`Activity`传递到它的构造函数里，`MeTrackerStore`持有这个`Activity`的引用，那这个`Activity`将仍然不会被回收（除非这个静态变量被重新赋值给另一个`Activity`）。这是因为`mMeTrackerStore `是静态的，静态变量的内存是不会被回收的，直到程序里正在运行的进程停止。如果你发现自己尝试这么做，那么你的代码可能有一些严重的错误。寻找帮助的话，可以看看Google's Udacity里的课程[ “Android Development for Beginners”](https://www.udacity.com/course/android-development-for-beginners--ud837)

注意：技术上讲，你可以hold一个对`Context`的静态引用，但不会引起内存泄漏，但我不会推荐你这么做。

###当心对那些你无法控制它的生命周期的对象进行隐式引用

```
public class DefineGeofenceFragment extends Fragment {
    public class GetLatAndLongAndUpdateMapCameraAsyncTask extends AsyncTask<String, Void, LatLng> {
 
        @Override
        protected LatLng doInBackground(String... params) {
            //...
            try {
                //Here we make the http request for the place search suggestions
                httpResponse = httpClient.execute(httpPost);
                HttpEntity entity = httpResponse.getEntity();
                inputStream = entity.getContent();
                //..
            }
        }
    }
    
    
}
```
这段代码有很多问题，但我将关注其中的一个。在Java中，非静态的内部类对包含它的类对象有一个隐式的引用。

在这个例子中，任何`GetLatAndLongAndUpdateMapCameraAsyncTask `对象都将有个`DefineGeofenceFragment `对象的引用。匿名类也是如此：它会对包含它的类对象有个隐式的引用。

这个`GetLatAndLongAndUpdateMapCameraAsyncTask `对象对`Fragment`对象有个隐式的的引用，一个我们无法控制它生命周期的对象。Android SDK负责适当的创建和销毁`Fragment`对象，如果因为`GetLatAndLongAndUpdateMapCameraAsyncTask `对象正在执行所以不能被回收的话，那它持有对象也无法被回收。

这里有一个很棒的Google IO 视频[解释为什么这种事会发生](https://www.youtube.com/watch?v=_CruQY55HOk)

###让Android Studio为你工作
```
public ViewPager getmViewPager() {
    return mViewPager;
}
```
这个片段是我使用"Generate Getter"代码补全时，Android Studio为我生成的，这个getter方法对这个实例变量保持了'm'前缀。这并不理想。

(另外，你一定想知道为毛实例变量神明的时候要带个'm'前缀：这个'm'常常被约定作为实例变量的前缀。它代表了'member'。)

不论你是否认为'm'作为你实例变量的前缀是一个好主意，这里有一个经验：Android Studio可以帮你按照你养成的习惯去编写代码。比如说，你可以使用Android Studio中的代码风格框去让Android Studio自动的加上'm'到你的实例变量并且自动移除'm'当它生成getters，setters，和构造参数时。

{%asset_img Screen-Shot-2015-07-09-at-4.16.13-PM.png%}

Android Studio可以做很多事情，学习[快捷键](http://www.developerphil.com/android-studio-tips-of-the-day-roundup-1/)和[活动模版](https://www.jetbrains.com/idea/help/live-templates.html)会是一个好的开始。

###方法应该只做一件事

有一个方法我写超过了1000行。这样的方法很难读懂，修改和重用。试着写仅仅做一件事的方法。典型的，这意味着你应该怀疑那些你写超过20行的代码。这里你可以招募Android Studio去帮助你指出有问题的方法:

{%asset_img Screen-Shot-2015-07-09-at-4.25.00-PM.png%}

###向那些比你更聪明，更有经验的人学习

这好像挺起来不重要，但是这是我写第一个app时犯下的错误。

当你正在写程序时，你将会犯错。其他的人已经犯过这样的错误了。向其他人学习。如果你重复那些可以避免的错误，那你就是在浪费时间。

读[Pragmatic Programmer](http://www.amazon.com/The-Pragmatic-Programmer-Journeyman-Master/dp/020161622X). 然后读[Effective Java](http://www.amazon.com/Effective-Java-Edition-Joshua-Bloch/dp/0321356683).这两本书会帮助你避免犯一些常见的错误。当你读完这两本书后，保持向聪明的人学习。

###使用库
当你写一个app，你可能会遇到那鞋前人已经解决了的问题。而且，大量的解决办法都是开放的作为资源库。 好好利用他们。

在我的第一个app中，我写的功能已经被其他库所提供了，它们中的一些库来自于标准的java中的一部分。另一些则是像`Retrofit `和`Picasso`这样的库。如果你不确定你要应该用什么库，你能做3件事：

1. 听[Google IO Fragmented podcast episode](http://fragmentedpodcast.com/episodes/9/)
2. 订阅[Android Weekly](http://androidweekly.net/)
3. 寻找解决类似问题的开源应用。你可能发现它们用了第三发的库(third-party library)或者用了你并没有在意的标准的java库。

###总结
写一个好的Android app是非常难的。不要因为重复我的错误让它变的更加艰难。

翻译自:[6 THINGS I WISH I KNEW BEFORE I WROTE MY FIRST ANDROID APP](http://www.philosophicalhacker.com/2015/07/09/6-things-i-wish-i-knew-before-i-wrote-my-first-android-app/)
