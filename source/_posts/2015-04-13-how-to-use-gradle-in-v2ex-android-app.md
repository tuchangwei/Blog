title: "[V2ex客户端源码分析]Gradle的使用"
s: how_to_use_gradle_in_v2ex_android_app
date: 2015-04-13 17:36:49
tags: Gradle
categories: Android
---
最近在学习Android开发，看教程太枯燥，网上找了几个不错的源码，试着分析clone，并把学习的点记录下来。

先看看根目录的build.gradle文件。

```buildscript {
    repositories {
        //mavenCentral()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.1.0'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        jcenter()
    }
}
```
这个文件中`repositories `是必须要配置的，gradle会根据这个仓库去查找你要包含到项目中的第三方库，老版本的Android Studio 用的仓库是`mavenCentral()`, 新版本换成了这个`jcenter()`,具体区别见[这里](http://blog.bintray.com/2015/02/09/android-studio-migration-from-maven-central-to-jcenter/)。

再看看app目录里的build.gradle。

```
apply plugin: 'com.android.application'

android {
    compileSdkVersion 21
    buildToolsVersion "21.1.2"

    defaultConfig {
        minSdkVersion 14
        targetSdkVersion 19
        versionCode 6
        versionName "1.3.0"
    }

}

dependencies {
//  The shortcut form for declaring external dependencies looks like “group:name:version”.
    compile 'com.android.support:support-v4:19.0.0'
    compile 'com.android.support:support-v13:19.0.0'
    compile 'uk.co.chrisjenx.calligraphy:calligraphy:0.6.0'
    compile 'com.github.chrisbanes.actionbarpulltorefresh:library:0.9.9'
    compile 'com.etsy.android.grid:library:1.0.5'
    compile 'com.astuetz:pagerslidingtabstrip:1.0.1'
    compile 'com.loopj.android:android-async-http:1.4.6'
    compile fileTree(dir: 'libs', include: ['*.jar'])
}

```
这里面的`dependencies `就是从`jcenter()`那里下载的第三方库，它的格式是:`group:name:version`，其他key值的解释见[这里](http://tools.android.com/tech-docs/new-build-system/user-guide)。