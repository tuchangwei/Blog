title: Two Tips When You Setup Firebase For Remote Notification
date: 2017-04-08 16:24:00

category: Android

tags: Firebase
---

I finished my first android app written by Kotlin language recently. Today I want to record what I learned when I setup Firebase for remote notification.

There are two cases to disucuss.

## 1. When the app is in foreground.

This case is easier, when the push is coming, the callback method `onMessageReceived` in the class which extends `FirebaseMessagingService` will be invoked. 



## 2. When the app is in the background or is killed.

Under this case, the notification will show in the system tray first. when we click the notification, the app launcher will be open by default. We can get the push information from its `intent`.

But if you don't want the default action, for example, in my app, the launcher is a Splash activity, I don't want to deal with push information in the activity, how we can do?

Now, let us see how to send push to devices.

```
Method - POST

EndPoint - https://fcm.googleapis.com/fcm/send

Header - 

Authorization:key=appkey

Content-Type:application/json

Body - 

{"notification": {
		"title": "Your Title",
		 "text": "Your Text",
		 "body": "Your body",
		 "click_action" : "PUSH"
	},
"data": {
    	"url": "xxx"
    },
"to" : "pushToken"
} 
```

The key-value `click_action` is the key point. I want MainActivity to initialize when the push is coming, so I config MainActivity in `AndroidManifest.xml` file

```java
<activity    android:name=".activities.MainActivity"    android:configChanges="keyboardHidden|orientation"    
android:launchMode="singleInstance"    
android:screenOrientation="portrait">    
	<intent-filter>        
	<action android:name="PUSH" />        
	<category android:name="android.intent.category.DEFAULT" />   
	</intent-filter>
</activity>
```

Note the action's name in `intent-filter` is same with the value of the key `click_action`.

Further more, if the MainActivity is in the background, I don't want to initilize a new one, so I set `android:launchMode="singleInstance" ` . This way, when the push is coming out, we can get its `intent` from the following callback method:

```java
override fun onNewIntent(intent: Intent?) {
        if (intent != null && intent.action == "PUSH" && intent.getStringExtra("url") != null) {
            requestPushContent(intent.getStringExtra("url"))
        }
}
```

Done!