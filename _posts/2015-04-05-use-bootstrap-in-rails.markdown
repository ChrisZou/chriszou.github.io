---
layout: post
title:  在Rails app中使用Bootstrap
tags: [tech, rails]
date:   2015-04-05 16:01:31 UTC
comments: true
---
这篇文章基本上算是对[railscast 328: Twitter Bootstrap Basics](http://railscasts.com/episodes/328-twitter-bootstrap-basics) 的一个学习总结。提供一个step-by-step的guide。同时也加入了一些需要注意的地方，比如说这个video cast是2012年做的，到现在为止无论是rails还是bootstrap都变了很多，在rails里面使用bootstrap的方法也有所不同。

1. 让rails支持LESS：Rails使用的CSS precompiler是SASS，而Bootstrap使用的是LESS，因为Rails不支持LESS，所以首先要使用一个第三方的library让Rails支持LESS。
在Gemfile 里面加入
```
gem 'less-rails'
gem 'twitter-bootstrap-rails'
```
在[这里](http://www.sitepoint.com/twitter-bootstrap-less-and-sass-understanding-your-options-for-rails-3-1/)可以了解到更多关于LESS和SASS的信息。
注意：在railscast里面说的是将`gem twitter-bootstrap-rails`加入到Gemfile的assets group里面就好了。这里有两个问题，一是[rails4里面将assets 这个group 去掉了](http://stackoverflow.com/questions/16406204/why-did-rails4-drop-support-for-assets-group-in-the-gemfile)，二是`twitter-bootstrap-rails`这个gem不自带`less-rails`，所以我们要手动添加。
2. 执行`bundle install` to install the gem。
3. 执行`rails g bootstrap:install`这句话的作用是将Bootstrap的一些CSS等文件加入到我们的rails app里面。这个命令的输出如下：
![install bootstrap output](http://chriszou.com/images/6-install-boostrap-output.png)
可以看到，这个命令增加了三个文件，修改了两个文件。到此为止，你就可以在rails里面使用Bootstrap的style了。至于怎么样使用Bootstrap把页面变得更漂亮，那就是CSS跟Bootstrap的事情了。

当然，话说回来，twitter-bootstrap-rails作为一个ruby gem，如果它的作用只是将bootstrap，那也实在是有点弱。其实它还给我们带来了很多使用bootstrap的便利和快捷方法。在这个gem的[github官网](https://github.com/seyhunak/twitter-bootstrap-rails) 可以看到详细的教程。这里介绍一个我试用过的，也是最基本的用法。

`rails g bootstrap:themed [RESOURCE_NAME]`

Resource就是Entity在Rails里面的叫法，类似于一个JavaBean的概念。比如我们创建一个博客文章的评论的entity：

`rails g scaffold Comment content:string email:string post:references`

那么这里的Comment就叫Resource。这时候我们可以使用

`rails g bootstrap:themed Comments`

来给Comment这个resource创建一些使用了Bootstrap style 的view。比如，打开Comments的index view，你可以看到类似于下面的页面：
![comments index with bootstrap](http://chriszou.com/images/6-comments-index-with-bootstrap.png)

是不是比Rails自己产生的style要漂亮多了？

哦对了，个人认为Rails自带generate出来的scaffold.css实在是太丑了，还不如直接删了不带任何style。
