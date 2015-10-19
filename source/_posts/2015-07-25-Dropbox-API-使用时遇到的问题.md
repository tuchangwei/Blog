title: Dropbox API 使用时遇到的问题
date: 2015-07-25 19:37:19
tags: Dropbox
categories: iOS
---

最近的项目需要整合Dropbox。需求是，拿到token和userids传到服务器。

记得上一次做OAuth2.0是2011年，那时候刚刚到第一个公司，新浪微博那时貌似还是OAuth1.0授权，而且文档不全，要靠试错来确定参数。那时没有集成好的类库可调用，自己也是第一次接触，用了一个星期时间把新浪微博，腾讯微博，开心网，人人网，qq空间都集成了项目内，慢慢的对OAuth2.0有些了解。

这里面主要用到两块知识，一块就是OAuth2.0的流程，另一个就是授权后如何跳转到自己的app内。

今天重做时，这两块各遇到一个问题，下面说说这两个问题。

###OAuth2.0
这个发送授权请求时，有一个参数是response_type，它的值可取`token `和`code`,我看例子请求用的是`code`，我也用`code`，结果在Safira内授权后死活都不忘自己app内跳转。后来在停留的网页上的一行小字发现了秘密。

{%asset_img QQ20150725-1@2x.png%}

于是改成`token`试一下就可以了。然后仔细看文档才发现，原来它这个流程分为成两个:

>####Code flow

>These parameters are passed in the query string (after the ? in the URL):

>>**code** The authorization code, which can be used to attain a bearer token by calling /oauth2/token.
>>
>>**state** The state content, if any, originally passed to /oauth2/authorize.

>>**Sample response**

>> 
```
 [REDIRECT_URI]?code=ABCDEFG&state=[STATE]
```

>####Token flow

>These parameters are passed in the URL fragment (after the # in the URL):

>>**access_token** A token which can be used to make calls to the Dropbox API.
>>
>>**token_type** The type of token, which will always be bearer.
>>
>>**uid** The Dropbox user ID of the authorized user.
>>
>>**state** The state content, if any, originally passed to /oauth2/authorize.
>>
>>**Sample response**
>>
```
[REDIRECT_URI]#access_token=ABCDEFG&token_type=bearer&uid=12345&state=[STATE]
```


### Custome URL Schemes
我在`URL Schemes`框框内填了一个xxx://xxx的格式，其实只要填xxx就可以了。

{%asset_img scheme.png%}