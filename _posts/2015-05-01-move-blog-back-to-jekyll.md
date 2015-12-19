---
layout: post
title:  "博客换回Jekyll，顺带简介一下Jekyll"
date:   2015-05-01 15:21
tags: [life]
comments: true
excerpt_separator: <!--more-->
---

个人网站开起来有一段时间了，当时目标是每周写一篇文章，不过到现在也没写多少篇哈哈。。
刚开始的时候是用Jekyll来做的，后来觉得不够自由，版式太固定不好扩展。于是又推倒，用Rails自己写了一个，用Jekyll的默认主题作为样式风格，用Redcarpet来做markdown的rendering。这些都很简单，唯一的问题在于，Jekyll的默认主题虽然简洁，但是太简陋了，于是自己试着去改一些布局啊，样式啊，都是CSS的东西。折腾了几次以后，我终于要承认，我并不是一个CSS guy，我并不是非常enjoy这个过程，而且还觉得挺烦人的。所以想想，还是算了，我想写东西，那就专注于写东西吧，样式的东西交给别人去做。
于是乎，又想到了Jekyll，心想像Jekyll这样一个popular的博客引擎，应该有很多很漂亮的主题吧，于是搜了一下，找到了[这个](http://jekyllthemes.org/themes/hmfaysal-omega-theme/)，也就是你现在看到的这个，个人感觉这是个非常适合blog的主题，既漂亮，同时也是content focused，于是决定就用这个了。从Rails app搬回Jekyll，最大的遗憾，就是之前的一些评论不能一起搬走，好可惜的感觉！

### 关于Jekyll
<!--more-->
稍微介绍一下[Jekyll](http://jekyllrb.com/)，Jekyll是一个static page generator，应该是如今用的最广泛的吧，因为github pages用的是这个。
Jekyll的工作原理，类似于一个文本转化器，它的原材料是[markdown](http://www.chriszou.com/learn-markdown-in-one-minute)文本，经过它的转换以后，变成了由HTML/CSS/JS组成的静态页面，这些静态页面产生以后，你把他们放到nginx或apache的root directory，就可以通过互联网访问了。是的，没有“后台”！没有数据库！没有动态代码（除非你把前端的JS代码看作“动态代码”）！！！Everything is static, thus everything is simple!

要使用Jekyll也很简单，首先你要有Ruby，要有Gem，然后

```Ruby
gem install jekyll
jekyll new my-blog
cd my-blog
jekyll serve
```

用浏览器打开[localhost:4000](localhost:4000)，你就可以看到你创建的第一个jekyll博客网站了。以后你要写自己的文章，只需要用markdown编辑器写好，保存为.md文件，然后放到my-blog/\_post里面。系统会自动将你在\_post下面的文章一个个遍历出来，放在首页。
更多的信息你可以在[Jekyll官网](http://jekyllrb.com)看到。

### 免费托管到github pages
用Jekyll的一个好处是，你可以用github pages来免费托管你的博客，将你的blog目录（如前面的my-blog）关联一个github pages专用的github repo。然后只管git push到repo就好了，简直不要太方便。
具体操作请看[这里](https://help.github.com/articles/using-jekyll-with-pages/)。

### 用Disqus来给你的blog加上评论功能
评论是很重要的，每一个写文章的人，除了希望自己写的东西能让别人看到，帮助到别人，还希望能得到读者的支持和鼓励，评论是最重要的一个途径。此外，评论还能让别人指出你文章中的错误，或者与别的读者的交流等等。
现在问题来了，前面提到，Jekyll产生的是静态页面，没有数据库，那么评论怎么保存呢？答案是我们不保存，或者说的更清楚一点，我们不自己保存评论。我们可以用第三方的专业的评论管理工具，来帮助保存，显示评论等等一系列相关功能。我这里选的是国外的[Disqus](https://disqus.com)，这是用的最广泛的评论管理服务。使用了以后，发现这里还有一个非常大的好处，那就是你注册一个账号，就可以直接用来评论别人的文章了。我的意思是，比如说，另外一个人也开了一个自己的博客网站，也使用了Disqus，那么你可以直接用你的Disqus账号评论他的文章，再也不用输入所谓的用户名，email等等一系列烦人的东西。
使用Disqus加上评论功能真的是非常非常简单，不信请看[这里](https://help.disqus.com/customer/portal/articles/472138-jekyll-installation-instructions)。
Disqus的简单让我产生了一个很严重的疑问，它是怎么来防伪的呢？我看了它的代码，并没有任何secret key这样的东西。它是怎么防止别人用我的账号来给他的文章加评论呢？或者说，它能不能防止？
Anyway，这个东西很好用，就用这个了。
国内有类似的服务，叫“多说”，这个之前Rails版的博客有人评论，我才知道这个东西的，当时我上去看了一下，貌似界面很丑，所以还是冒着会被The Fucking GFW屏蔽的危险，继续使用Disqus。

### 小结
写文章，最重要的还是写“文章”，至于用什么写，写在哪里，都是次要的。找一个适合自己的工具（所谓的适合自己，指的是自己用起来觉得舒服，顺手），然后坚持写下去。
