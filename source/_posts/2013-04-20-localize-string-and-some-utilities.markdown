---
layout: post
title: "Localize strings using some utilities"
date: 2013-04-20 22:07
comments: true
categories: Localize 
---
When release applications, one thing will not be avoided, it is localizing. According to the different user demands, we can divide them into `Set language through Settings` and `Set language inside App`

###Set language through Settings

The user demand is easier than another one, when we change language by "Settings -> General -> international", all applications will restart. So, the .strings file will be loaded again and the language is changed.

Now, let's see how to do it.

* Use the macro `NSLocalizedString(@"user_label_text", nil);` to wrap your strings.
* Add one kind of language to project as follow, Xcode will generate a forder named `language-lproj`.
  <a href="http://www.flickr.com/photos/tuchangwei/8664824887/" title="Flickr 上 涂 涂 的 412C868B-805A-405A-B5FB-B8F4FB0B96CC"><img src="http://static.flickr.com/8244/8664824887_29f530680e.jpg" width="500" height="284" alt="412C868B-805A-405A-B5FB-B8F4FB0B96CC"></a>
* Use the commend tool `genstrings` to generate Localizable.strings file.

		genstrings *m -a -o  language-lproj
* Change the value of the key for different language.
	
		"user_label_text" = "用户登陆"
		
The game seems over, but it is not true. The genstrings commend either replaces your earily changes, or just follows the changes, does not merge them. This is boring.

So what can i do? You know, we do not be defeated by Ants. Some advice as follows:

1. [Localizable Strings Merge ](http://www.delitestudio.com/app/localizable-strings-merge/) needs $4.99.
2. [AGi18n](AGi18n) free and can auto convert strings in xib.
3. my shells.
	
	localizedstring.sh
	
		#!/bin/sh
		echo "input the language you want localize:"
		read language
		echo $language
		desFilePath=$PWD/"$language".lproj/Localizable.strings
		echo des file path is:"$desFilePath"
		if [ ! -e $desFilePath ] ; then
		    echo "des file doed not exit, touch it."
		    touch $desFilePath
		fi
		tmpDir=$PWD/"$language".lproj/tmp
		tmpFile="$tmpDir"/Localizable.strings
		tmpFile1="$tmpDir"/Localizable1.strings
		
		rm -rf $tmpDir
		mkdir $tmpDir
		touch $tmpFile1
		find . -name '*.m' | xargs genstrings -macRoman -a -o  $tmpDir
		find . -name '*.mm' | xargs genstrings -macRoman -a -o  $tmpDir
		iconv -f utf-16le  -t utf-8 $tmpFile > $tmpFile1
		chmod 755 loop.sh
		echo  tempfile=$tmpFile1
		echo des=$desFilePath
		./loop.sh $tmpFile1 $desFilePath


	loop.sh
	
		#!/bin/sh  
		echo 1=$1
		echo 2=$2
		SRC=$1
		DES=$2
		if [ $# -ne 2 ] ; then
			echo Usage: loop.sh src des
			exit 1
		fi
		echo arg OK
		while read line; do
			#line is NULL
		#echo line=$line
			if [ "$line" == "" ] ; then
				echo $line >> $2
				continue
			fi	
			equal=`echo $line|grep "="`
			if [ "$equal" == "" ] ; then
				echo "$line" >> $2
				continue
			fi	
			left=`echo $line| cut -d "=" -f1`
			num=`cat $2|grep "$left"|wc -l`
		#echo num=$num
		#	sleep 1
			if [ $num -eq 0 ]; then
		#   echo "$line insert $2 !!!!!!!!!!!!"
				echo $line >> $2
			fi
		done < $SRC
		
4. [Lin](https://github.com/questbeat/Lin) can reduce workload for you.

	![](https://a248.e.akamai.net/camo.github.com/e089475332f6d71842cf1769d80aaccb14b5e809/687474703a2f2f61646f746f75742e73616b7572612e6e652e6a702f6769746875622f4c696e2f6c696e5f737330312e706e67)
	
	![](https://a248.e.akamai.net/camo.github.com/6c4fc4630543be9addbb384133d07074866d0a8a/687474703a2f2f61646f746f75742e73616b7572612e6e652e6a702f6769746875622f4c696e2f6c696e5f737330322e706e67)
	
	![](https://a248.e.akamai.net/camo.github.com/cd16d08d3838fbe83fc72922046ca2e1a03fe7a3/687474703a2f2f61646f746f75742e73616b7572612e6e652e6a702f6769746875622f4c696e2f6c696e5f737330332e706e67)
	
	![](https://a248.e.akamai.net/camo.github.com/ee79f2b06520190d207530b805dc4e656c396efa/687474703a2f2f61646f746f75742e73616b7572612e6e652e6a702f6769746875622f4c696e2f6c696e5f737330342e706e67)

###Set language inside App

For this, we need an Object to load stings information real-time.

LanguageHelper.h

	#import <Foundation/Foundation.h>
	#define CustomLocalizedString(str,comment) [LanguageHelper  get:str alter:nil]
	@interface LanguageHelper : NSObject
	
	+(void)initialize;
	+(void)setLanguage:(NSString *)lan;
	+(NSString *)get:(NSString *)key alter:(NSString *)alternate;
	
	/* @brief 返回当前语言
	 * @return 返回当前设置的语言
	 */
	+(NSString *)currentLanguage;
	@end
	
	
LanguageHelper.m

	#import "LanguageHelper.h"
	
	@implementation LanguageHelper
	
	static NSBundle *bundle = nil;
	
	+(void)initialize {
	    
	    
	    NSUserDefaults* defs = [NSUserDefaults standardUserDefaults];
	    NSString *current = @"";
	    if ([defs valueForKey:@"userLanguage"])
	    {
	        current = [defs valueForKey:@"userLanguage"];
	    }
	    else
	    {
	        NSArray* languages = [defs objectForKey:@"AppleLanguages"];
	        current =[languages objectAtIndex:0];
	        
	       
	        NSArray *supportLanguage = [[NSBundle mainBundle] objectForInfoDictionaryKey:@"SupportedLanguage"];
	        if (![supportLanguage containsObject:current])
	        {
	            current = [[NSBundle mainBundle] objectForInfoDictionaryKey:@"DefaultLanguage"];
	        }
	        [defs setValue:current forKey:@"userLanguage"];
	        [defs synchronize];
	    }
	   
	    [self setLanguage:current];
	    
	}
	+(void)setLanguage:(NSString *)lan {
	    NSLog(@"preferredLang: %@", lan);
	    NSString *path = [[ NSBundle mainBundle ] pathForResource:lan ofType:@"lproj" ];
	    bundle = [NSBundle bundleWithPath:path];
	}
	
	+(NSString *)get:(NSString *)key alter:(NSString *)alternate {
	    return [bundle localizedStringForKey:key value:alternate table:nil];
	}
	
	+(NSString *)currentLanguage{
	    NSUserDefaults* defs = [NSUserDefaults standardUserDefaults];
	    
	    return [defs objectForKey:@"userLanguage"];
	}
	
	@end
	
First, we need replace `NSLocalizable(string,commit)` to `CustomLocalizable(string,commit)`, then when the app initializes, we call the method `+(void)initialize`, when we  change another language, we call the method `+(void)setLanguage:(NSString *)lan`, then we reload the data on views, the function come true.

###Attention！！！
Today, the earthquake attracked YaAn. They need our help, you can donate money through [壹基金](https://love.alipay.com/donate/itemDetail.htm?name=201304201216494301), God blesses YaAn and us.
