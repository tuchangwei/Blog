---
layout: post
title: "Use Command line to pack applications"
date: 2013-03-30 17:02
comments: true
categories: Utility
---
All right. It is time for me to write something. My wife has stayed in her hometown about half a month. I miss her very much. It is dark out of the window.  You know, in summer, the rain is rich in ShenZhen. Sometimes, I am very anxious about doing something which is matter with opportunity and ability. Anyway, let us do something about work.

Sometimes we need to build apps for many channels, we needn't change many codes but some configurations. As we all know, we use Xcode to build and run application. many channels many building. So boring! So let us see how to do it using command line.

* first shell, it is used for generating binary to upload AppStore.

		#!/bin/sh
		target="Power7 MobileView"
		distDir="/Users/tuchangwei/Documents/WorkSpace/Release/${factory}/${appName}_V${version}"
		rm -rdf "$distDir"
		mkdir "$distDir"

		uploadDir="${distDir}/Upload AppStore"
		rm -rdf "$uploadDir"
		mkdir "$uploadDir"
		/usr/bin/xcodebuild -target "$target" clean
		echo "******************start build*********************"
		/usr/bin/xcodebuild -target "$target" CODE_SIGN_IDENTITY="$distribution"
		echo "******************start pick .ipa*********************"
		/usr/bin/xcrun -sdk iphoneos PackageApplication -v build/Release-iphoneos/*.app -o "${uploadDir}/${appName}.ipa" --sign "$distribution"
		echo "******************start pick test .ipa*********************"
		codesign -f --sign "iPhone Developer: Nelson Chen (JPMGJWJFAQ)" build/Release-iphoneos/*.app
		/usr/bin/xcrun -sdk iphoneos PackageApplication -v build/Release-iphoneos/*.app -o "${distDir}/${appName}_V${version}.ipa" --sign "iPhone Developer: Nelson Chen (JPMGJWJFAQ)" --embed "/Users/tuchangwei/Library/MobileDevice/Provisioning Profiles/D788EEA0-848E-4F5E-AA30-87D38154DA9B.mobileprovision"
		
* second shell is used for generating test binary

		#!/bin/sh
		target="Power7 MobileView"
		distDir="/Users/tuchangwei/Desktop/Test"
		rm -rdf "$distDir"
		mkdir "$distDir"
		/usr/bin/xcodebuild -target "$target" clean
		echo "******************start build*********************"
		/usr/bin/xcodebuild -target "$target" 
		echo "******************start pick test .ipa*********************"
		codesign -f --sign "iPhone Developer: Nelson Chen (JPMGJWJFAQ)" build/Release-iphoneos/*.app
		/usr/bin/xcrun -sdk iphoneos PackageApplication -v build/Release-iphoneos/*.app -o "${distDir}/test.ipa" --sign "iPhone Developer: Nelson Chen (JPMGJWJFAQ)" --embed "/Users/tuchangwei/Library/MobileDevice/Provisioning Profiles/D788EEA0-848E-4F5E-AA30-87D38154DA9B.mobileprovision"

	
* third shell is used for deleting .svn files in project.

		#!/bin/sh
		echo "start delete .svn..."
		find $PWD/ -name .svn -print0 | xargs -0 rm -rf
		echo "finish delete .svn."
		
So, you know, i also use some shell to improve efficiency. See you,O(∩_∩)O~.


