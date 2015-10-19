---
layout: post
title: "说说翻墙软件 GoAgentX"
date: 2013-03-10 08:19
comments: true
categories: Wall
---


翻墙软件对程序猿来说应该都是必备的吧，最近翻墙又出现了问题，所以又捣鼓了一遍。现在记下来，免得以后出问题又要翻半天。

1. [申请Google Appengine](https://appengine.google.com/)。这是需要注意的是：申请时填写的app id不要包含移动平台的字段，比如iphone或者android。不然进入facebook，twitter网站会默认进入移动版。
2. [下载GoAgentX](https://github.com/ohdarling/GoAgentX/downloads)。最新的版本里已经包含了goagent，所以不再需要下载goagent。
3. 安装GoAgentX并配置。这里要注意的是：服务配置选项卡里面的服务密码要填写。其他设置中的“导入goagent根证书”要点击导入。一个是为了防止出现url/2的错误，一个是为了防止无法登陆https网站。
4. chrome安装[SwitchySharp插件](https://chrome.google.com/webstore/detail/proxy-switchysharp/dpplabbmogkhghncfbfdeeokoefdjegm),并在插件中导入<http://goagent.googlecode.com/files/SwitchyOptions.bak>

那么，希望各位能在一个开放自由的互联网环境中学习，成长。