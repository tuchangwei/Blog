---
layout: post
title: "Octopress 怎么添加分类"
date: 2013-02-16 11:07
comments: true
categories: octopress
---

大致步骤为：
###增加category_list插件
保存以下代码到plugins/category_list_tag.rb：

	module Jekyll
	  class CategoryListTag < Liquid::Tag
	    def render(context)
	      html = ""
	      categories = context.registers[:site].categories.keys
	      categories.sort.each do |category|
	        posts_in_category = context.registers[:site].categories[category].size
	        category_dir = context.registers[:site].config['category_dir']
	        category_url = File.join(category_dir, category.gsub(/_|\P{Word}/, '-').gsub(/-{2,}/, '-').downcase)
	        html << "<li class='category'><a href='/#{category_url}/'>#{category} (#{posts_in_category})</a></li>\n"
	      end
	      html
	    end
	  end
	end
	
	Liquid::Template.register_tag('category_list', Jekyll::CategoryListTag)
这个插件会向liquid注册一个名为category_list的tag，该tag就是以li的形式将站点所有的category组织起来。如果要将category加入到侧边导航栏，需要增加一个aside。
###增加aside
复制以下代码到source/_includes/asides/category_list.html。(请将下面的$号换成%号)

	<section>
	  <h1>Categories</h1>
	  <ul id="categories">	  
	  	{$ category_list $}	    
	  </ul>
	</section>

配置侧边栏需要修改_config.yml文件，修改其default_asides项：

	default_asides: [asides/category_list.html, asides/recent_posts.html]

###使用
当新建markdown文件时，顶部会自动显示：“categories:”，在其后写上自己的分类名即可。
###参考
<http://codemacro.com/2012/07/18/add-category-list-to-octopress/>

  <!--more-->