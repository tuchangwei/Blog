title: 增加Document Provider Extensions时遇到的问题
date: 2016-03-06 09:15:26
tags: Extensions
categories: iOS
---

最近有个需求是在其他的应用中可以选择自己app中的文件，于是接触到了Document Provider Extensions，其实我所遇到的问题在其他Extensions中也可能会遇到，做个总结。

首先这里有几个名词，`host app`,`UIDocumentPickerViewController`,`Document Provider Extensions`,这些名词的关系如下图所示：

{%asset_img QQ20160306-2@2x.png%}

也就是说怎么显示`Document Provider Extensions`菜单，那是host app的事，我们能做的是在container app中开发这个Extension,它就会显示在这个菜单上。不论中间的过程怎样，最终我们只要**通过这个Extension给host app回传一个文件URL**，我们目标就完成了。

### 问题一，要不要创建File Provider Extension
突然蹦出的这个Extension是干嘛的？这个Extension是没有界面的,官方说法是它相当于`Document Picker extension`的后端。当你在xcode中选择创建`Document Provider`target时，它会弹一个复选框让你选择要不要同时创建`File Provider extension`。那要不要同时创建呢？这个要看你的需求了，官方和raywenderlich的资料中都说的比较含糊，我自己的理解是，如果你要分享的数据在`Document Picker extension`里能拿到，而不是通过URL从container app中取的，那你就没有必要创建`File Provider extension`。在我这个需求中是不需要通过URL从container app中取数据的，所以我没有创建这个extension。那第二个问题来了。

### 问题二，Document Picker Extension怎样回传给host app 一个URL
这里的关键是`UIDocumentPickerExtensionViewController`中的`dismissGrantingAccessToURL` 方法。
官方的说法是:


>Dismisses the document picker.
>
>Call this method when the user selects a document or destination. This method dismisses the document picker view controller in the host app and triggers the appropriate file transfer. After the transfer is complete, the method passes the provided URL to the host app’s documentPicker:didPickDocumentAtURL: delegate method.
>
>The URL must meet all of the following conditions:
>
>* Import Document Picker mode. Provide a URL for the selected file. The URL only needs to be accessible by the Document Picker View Controller extension.
>
>...

也就是说只要把date写到`Document Picker View Controller extension`能访问的地方就行了，那刚好就用公共区域好了。

```
NSURL *containerURL = [fileManager containerURLForSecurityApplicationGroupIdentifier:@"group.xxx"];
```

### 第三个问题，啥是公共区。
这个公共区是你的extensions和container app都能访问的区域，在你的extensions和container app的target的配置中，你会配置一个App Groups，拥有相同App Groups的targets会有一个都可访问的公共区。那么第四个问题也来了。

### 第四个问题，extensions和container app共享的数据放在哪？

这个问题在第三个问题中已经回答了，但是恰好你用的是MagicalRecord，那么你可以把你的数据库创建在公共区域内，用来共享，相关代码是:

```
NSFileManager *fileManager = [NSFileManager defaultManager];
NSURL *containerURL = [fileManager containerURLForSecurityApplicationGroupIdentifier:@"group.xxx"];
NSURL *storeURL = [containerURL URLByAppendingPathComponent:[MagicalRecord defaultStoreName]];
[MagicalRecord setupCoreDataStackWithAutoMigratingSqliteStoreAtURL:storeURL];
```

如果你恰好也用了CocoaPads去管理你的第三方代码库,那么第五个问题也粗线了。

### 第五个问题，怎样用CocoaPads在你的container app和extensions 中共享第三方代码库?

在你的Podfile中加一行代码，然后pod install就解决了。

```
link_with 'Your container app', 'Your extension'
```
上面的代码不是很推荐，你也可以用下面的代码。

```
target 'Haystack_V2' do
    pod 'MagicalRecord', '~> 2.3.0'
    pod 'SVProgressHUD', '~> 1.1.3'
    pod 'SwipeBack', '~> 1.0'
end
target 'Photo Search' do
    pod 'MagicalRecord', '~> 2.3.0'
    pod 'SwipeBack', '~> 1.0'
end
```
所以你也可能遇到第六个问题，如果你也想在extension中使用SVProgressHUD这个库的话。
### 第六个问题，在Extensions中使用SVProgressHUD库

请看下图:

{%asset_img QQ20160306-3@2x.png%}

好像就这些了。

Happy coding。









