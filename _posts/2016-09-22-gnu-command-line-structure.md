---
layout: post
title: '一个符合GNU标准的命令行的组成格式'
date: 2016-09-22T21:15:53+08:00
comments: true
post_id: gnu-command-line-structure
---

这几天继续看《Build Awesome Command-Line Applications in Ruby 2》，其中讲到了GNU规定的命令行的格式，如下图所示：  

![](http://jietu-10024907.file.myqcloud.com/ggjgjfmiiyeuavgnvbecnnclspfocwov.jpg)

一个terminal命令主要由4部分组成：
1. 命令名（Executable): git
2. 选项（Options）: --no-pager 和 -v都是
3. 参数（arguments）: origin_master
4. 子命令（Command）: push  

从形式上来说，Options有两种形式：简短形式和完整形式。
简短形式一般由一个连接符“-”后面跟一个字母组成：  

```
ls -l -a -t # -l, -a, -t都是简短形式的Option
```

其实，几个简短形式的options可以合并写成一个：

`ls -lat` 效果等同于 `ls -l -a -t`

完整形式的则一般由两个连接符“--”开头，接着是一个或多个完整的单词，如果有多个单词，那么中间用一个连接符连接，如上面的“--no-pager”。

从功能上来讲，Options一般有两种。一种的是“switch”（开关），用来enable或者是disable（一般以“--no”开头）一些feature，如上面的`--no-pager`，就是用来disable pager这个feature的。除此之外的另外一种是flag。switch一般没有参数，flag则一般有参数。

如果一个flag有参数，那么一般简短形式形式的flag跟它的参数之间由一个空格分开。而完整形式的flag则用一个“=”连接它的参数，如：

```
curl -X POST http://www.google.com #POST是-X的参数
curl --request=POST http://www.google.com #POST是--request的参数。
```

这里要分清楚的是Option的参数和这整个命令的参数，在上面的例子中，`POST`是Option `-X`（或`--request`）的参数，而`http://www.google.com`则是这整个命令的参数。

对于一个Command Suite（[什么叫Command Suite？](http://chriszou.com/2016/09/22/what-is-single-purpose.html)）来说，option又分为Global Options和Command Option，这两者是不一样的。一个是对整个Command Suite的Option，一个是对于单个子命令的Option。它们甚至可以有相同的名字，含义却不一样，这个我相信应该也是比较好理解的。
