---
layout: post
title: "Add Sina Weibo Support for Greyshade theme of Octopress"
date: 2013-09-10 21:07
comments: true
categories: Octopress
---
I am not familiar with CSS and ROR until now. It is not easy for me to add Sina Weibo support as the slidebar shows. So let me note it.

* Firstly, add the following code to "octopress/_config.yml" file. Note that the weibo_user should be your weibo id, not your nickname.

{% codeblock %}

# Weibo link
weibo_user: thankforyou

{% endcodeblock %}

Behind of the following code:

{% codeblock %}

# ----------------------- #
#   3rd Party Settings    #
# ----------------------- #

{% endcodeblock %}

* Secondly, add the "weibo.png" to "octopress/source/images/social" dirctory.
* Thirdly, add the following code to "octopress/sass/parts/_header.scss" file.
	
{% codeblock %}
	
&.weibo{
	background: image-url('social/weibo.png') center no-repeat #e32529;
	border: 1px solid #e32529;
	&:hover{
		border: 1px solid darken(#e32529, 10%);
	}
}

{% endcodeblock %}
	
In front of the following code:

{% codeblock %}

&.facebook{
	background: image-url('social/facebook.png') center no-repeat #3B5998;
	border: 1px solid #3B5998;
	&:hover{
		border: 1px solid darken(#3B5998, 10%);
	}
}
&.google{
	background: image-url('social/google.png') center no-repeat #C83D20;
	border: 1px solid #C83D20;
	&:hover{
		border: 1px solid darken(#C83D20, 10%);
	}
}
&.twitter{
	background: image-url('social/twitter.png') center no-repeat #55CFF8;
	border: 1px solid #55CFF8;
	&:hover{
		border: 1px solid darken(#55CFF8, 10%);
	}
}
……			

{% endcodeblock %}

* Fourthly, add the following code to "octopress/source/_includes/header.html" file.

{% codeblock %}
	
{% if site.weibo_user %}
	<a class="weibo" href="http://www.weibo.com/{{ site.weibo_user }}" title="Weibo">Weibo</a>
{% endif %}
	
{% endcodeblock %}

Behind of the following code:

{% codeblock %}

……
<nav id="sub-nav">
	<div class="social">

{% endcodeblock %}

Ok, have done.