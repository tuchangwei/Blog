title: Some Tips  About NSURLSession
date: 2015-07-18 20:48:02
tags: NSURLSession
categories: iOS
---

* 后台传输时，需要保证队列里有多于一个任务，否则程序仍然会挂起。
* 后台传输服务仅支持`NSURLSessionUploadTask`,`NSURLSessionDownloadTask`,并不支持`NSURLSessionTask`。
* 后台上传时，只能使用`uploadTaskWithRequest:fromFile:`这个方法。如果NSURLSession对象由`backgroundSessionConfiguration`这个参数创建，这个方法`uploadTaskWithRequest:fromData:`不会工作。参见[这里](http://stackoverflow.com/questions/19985353/nsurlsession-uploading-assets-with-background-transfer?rq=1)
* In iOS, when a background transfer completes or requires credentials, if your app is no longer running, iOS automatically relaunches your app in the background and calls the `application:handleEventsForBackgroundURLSession:completionHandler:` method on your app’s `UIApplicationDelegate` object. This call provides the identifier of the session that caused your app to be launched. Your app should store that completion handler, create a background configuration object with the same identifier, and create a session with that configuration object. The new session is automatically reassociated with ongoing background activity. Later, when the session finishes the last background download task, it sends the session delegate a `URLSessionDidFinishEventsForBackgroundURLSession:` message. Your session delegate should then call the stored completion handler.

