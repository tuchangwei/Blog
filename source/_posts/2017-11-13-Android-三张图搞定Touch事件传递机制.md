title: Android-三张图搞定Touch事件传递机制
date: 2017-11-13 10:56:59

categories: Android

tags: touch
---

转自: [Android-三张图搞定Touch事件传递机制](http://hanhailong.com/2015/09/24/Android-三张图搞定Touch事件传递机制/)

之前看了很多关于Android事件Touch传递机制的文章，感觉还是老外讲的最清楚。原版PDF地址：[Mastering the Android Touch System](http://wugengxin.cn/download/pdf/android/PRE_andevcon_mastering-the-android-touch-system.pdf)，github的demo地址：[demo](https://github.com/devunwired/custom-touch-examples)

上图之前先讲下Android事件的基础知识：

1. 所有的Touch事件都封装到MotionEvent里面
2. 事件处理包括三种情况，分别为：传递—-dispatchTouchEvent()函数、拦截——onInterceptTouchEvent()函数、消费—-onTouchEvent()函数和OnTouchListener 
3. 事件类型分为ACTION_DOWN, ACTION_UP, ACTION_MOVE, ACTION_POINTER_DOWN, ACTION_POINTER_UP, ACTION_CANCEL等，每个事件都是以ACTION_DOWN开始ACTION_UP结束

Android事件传递流程：

1. 事件都是从Activity.dispatchTouchEvent()开始传递

2. 事件由父View传递给子View，ViewGroup可以通过onInterceptTouchEvent()方法对事件拦截，停止其向子view传递

3. 如果事件从上往下传递过程中一直没有被停止，且最底层子View没有消费事件，事件会反向往上传递，这时父View(ViewGroup)可以进行消费，如果还是没有被消费的话，最后会到Activity的onTouchEvent()函数。

4. 如果View没有对ACTION_DOWN进行消费，之后的其他事件不会传递过来，也就是说ACTION_DOWN必须返回true，之后的事件才会传递进来

5. OnTouchListener优先于onTouchEvent()对事件进行消费

   ----

   效果图如下：

   1. View不处理事件流程图（View没有消费事件）

      {%asset_img view_touch_ignorant.png.jpeg %}

   2. View处理事件

      {%asset_img view_touch_interested.png.jpeg %}

   3. 事件拦截

      {%asset_img view_touch_intercept.png.jpeg %}

