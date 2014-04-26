---
layout: post
title: "用git管理自己的代码库"
date: 2014-04-26 20:59:40 +0800
comments: true
categories: 
---
刚开始学习git，下面是两篇教你如何使用git管理代码的教程：

[使用git管理自己的代码--简单使用流程](http://my.oschina.net/moishalo/blog/72206)

[在GitHub上分享和展示你的代码](http://serholiu.com/github-share-code)

OK,看完上面这两篇博文差不多了解了。下面将一些基础的命令记录下来。

进入自己需要同步的目录, 建立一个仓库
	cd ~/code
	git init

选择要添加仓库的文件
	git add .
这里'.'代表这个文件夹下的所有文件。

使用git status命令查看我们做过哪些修改，建议在提交前都调用一下这个命令，看看我们做过什么改动。
	git status

使用git commit命令将文件提交到本地的Repository中，也就是离线提交，这个时候是可以没有网络链接的。注意：m参数后面跟的是提交的注释，记录这次提交的改变
	git commit -m 'test'

使用git clone命令将GitHub中创建的Repository同步到目录中
	git clone git@github.com:conpgy/pgy.github.com.git

使用git push命令将代码提交到服务器中。git push命令后面可以跟分支名，新创建的Repository默认分支是master。如果不跟分支名，默认直接提交到主分支master上。
	git push -u origin master

好了，现在你就可以到你的项目网站上查看你的代码了。

