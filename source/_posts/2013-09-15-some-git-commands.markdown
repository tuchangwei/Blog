---
layout: post
title: "Some git commands"
date: 2013-09-15 09:43
comments: true
categories: Git
---
{% codeblock lang:objc %}
	
git config --global color.ui true # support color global.

git reset --hard 89428ab23e4bd3028c95b4b4064075b3901654a3 # reset HEAD, index and working tree

git branch -a # List both remote-tracking branches and local branches.

git branch backups # new branch backups.

git push origin backups:backups # push local branch to remote branch.

git push origin :backups # delete remote branch backups.

git branch -d backups # delete local branch backups.

{% endcodeblock %}


{% codeblock lang:objc %}

git commit -m 'commit data' #forgin a file.

git add another_file # add the file.

git commit --amend # commit again.

{% endcodeblock %}