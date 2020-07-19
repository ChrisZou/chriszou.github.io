---
layout: post
title: '标准GNU命令行的格式'
date: "2016-10-01T14:46:53+08:00"
comments: true
post_id: gnu-command-line-structure
---

我们每天都在Terminal下面运行很多命令，但是，你了解一条命令的标准格式吗？对于我来说，以前并没怎么关心过，都是拿来就用，不会就google。这几天继续看[《Build Awesome Command-Line Applications in Ruby 2》](https://pragprog.com/book/dccar2/build-awesome-command-line-applications-in-ruby-2)，其中讲到了GNU规定的命令行的格式，看完以后有种恍然大悟的感觉，才发现原来一条命令还有一个标准格式的，有了这个格式，很多东西都不用记了，一个命令用起来就觉得有章可循了，对一个不熟悉的命令，也更容易猜到，或是找到使用的方法。

废话不多说，一个GNU规定的命令行的格式如下图所示：  

![](http://jietu-10024907.file.myqcloud.com/ggjgjfmiiyeuavgnvbecnnclspfocwov.jpg)

一个完整的Terminal命令主要由4部分组成：

1. 命令名（Executable): git
2. 子命令（Command）: push  
3. 选项（Options）: --no-pager 和 -v都是
4. 参数（arguments）: origin和master

这里重点是选项（Options）的一些格式规定。

#### 选项根据形式分类

从形式上来说，Options有两种形式：简短形式和完整形式。
简短形式一般由一个连接符“-”后面跟一个字母组成：

```
ls -l -a -t # -l, -a, -t都是简短形式的Option
```

其实，几个简短形式的options可以合并写成一个：  
`ls -lat` 效果等同于 `ls -l -a -t`  
完整形式的则一般由两个连接符“--”开头，接着是一个或多个完整的单词，如果有多个单词，那么中间用一个连接符连接，如上面的“--no-pager”。

#### 选项根据功能分类

从功能上来讲，Options一般有两种。一种的是“switch”（开关），用来enable或者是disable（一般以“--no”开头）一些feature，如上面的`--no-pager`，就是用来disable pager这个feature的。除此之外的另外一种是flag。switch一般没有参数，flag则一般有参数。

如果一个flag有参数，那么一般简短形式的flag跟它的参数之间由一个空格分开。而完整形式的flag则用一个“=”连接它的参数，如：

```
curl -X POST http://www.google.com
#POST是-X的参数

curl --request=POST http://www.google.com
#POST是--request的参数。
```

这里要分清楚的是Option的参数和这整个命令的参数，在上面的例子中，`POST`是Option `-X`（或`--request`）的参数，而`http://www.google.com`则是这整个命令的参数。

从文章开关的图也可以看到，对于一个Command Suite（[什么叫Command Suite？](http://chriszou.com/2016/09/22/what-is-single-purpose.html)）来说，Option又分为Global Options（`--no-pager`)和Command Option(`-v`)，这两者是不一样的。一个是对整个Command Suite的Option，一个是对于单个子命令的Option。它们甚至可以有相同的名字，含义却不一样，这点我相信应该是比较好理解的，就不细说了。
