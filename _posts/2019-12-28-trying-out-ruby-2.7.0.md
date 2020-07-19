---
layout: post
title: Ruby 2.7.0 上手体验以及问题记录
date: "2019-12-28T14:40:34+08:00"
comments: true
post_id: trying-out-ruby-2.7.0
---

正如过去的每个版本一样，Ruby 最新大版本 2.7.0 在圣诞节那一天正式发布了。空下来以后，我第一时间开始使用了。因为里面有一个我期待已久的新功能，那就是 block default parameters。
在 2.7 以前，Ruby 里面写一个带参数的 block 是这样的。

```
[1， 2， 3].each { |item| puts item }
```

Block 的参数放在两个`|`中间，这跟目前流行的大多数语言都不一样，非常的不直观，而且这个`|`符号也不好敲。
在 2.7 以后，你可以用`_1` 来引用第一个参数，`_2` 来引用第二个参数。所以，上面的例子就变成了：

```
[1, 2, 3].each { puts _1 }
```

少敲几次键盘，简直爽呆了，懒人的幸福就是这么简单。

说一下升级过程中遇到的几个问题。

#### 1. 使用 rvm 安装，下载时间特别长。

如果你使用 rvm 来安装、管理 Ruby，那使用 `rvm install` 来安装一个 Ruby 版本的时候，rvm 会先从[travis-ci](https://travis-ci.org/)寻找 binary。如果找到了，则直接下载 binary 安装。不然的话，会从 [ftp.ruby-lang.org](ftp.ruby-lang.org) 下载源码，然后再执行 make 安装。但是由于 You Know Why 的原因。国内无论访问 travis-ci 还是 ftp.ruby-lang.org，速度都非常慢。解决办法是使用[ruby-china 的国内镜像](https://ruby-china.org/wiki/ruby-mirror)。
使用方法：

```ruby
echo "ruby_url=https://cache.ruby-china.com/pub/ruby" > ~/.rvm/user/db~_
# 然后
rvm install 2.7.0 --disable-binary
```

#### 2. 使用`rvm upgrade`，更方便的安装新版本

在之前，每次 Ruby 出新版本，我都会使用 rvm install 安装一个新版本。由于 rvm 的版本管理方式，每个 Ruby 版本的 gem 是隔离的，相互不共享。于是就会经常遇到，升级了 Ruby 版本之后，运行某些脚本或命令（比如 pry），会提示这个 gem 找不到。然后就只能手动安装了。后来偶然间发现 rvm 有一个 upgrade 命令，可以安装一个指定的版本，然后把老板的 gem 迁移过去。只需要执行：
`rvm upgrade 2.6.5 2.7.0`

#### 3. 启动 Rails 的时候发生错误：`no implicit conversion of String into Integer (TypeError)`

安装好新版，执行 bundle 之后，启动 rails app 的时候遇到错误：

```ruby
bootsnap-1.4.4/lib/bootsnap/load_path_cache/core_ext/kernel_require.rb:22:in `require': no implicit conversion of String into Integer (TypeError)
```

Google 了一下，发现是由于 bootsnap 跟 Ruby2.7.0 的一个不兼容问题，执行`bundle update` 升级 bootsnap 之后，问题就解决了。
