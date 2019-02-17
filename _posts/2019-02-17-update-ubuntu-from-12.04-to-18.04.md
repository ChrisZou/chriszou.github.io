---
layout: post
title: update ubuntu from 12.04 to 18.04
date: 2019-02-17T18:00:19+08:00
comments: true
---

# 记打卡团服务器的一（三）次升级：从 Ubuntu 12.04 到 18.04

正月初六，乘着春节假期的最后一天，我窝在家里，一鼓作气，把我的一个业余项目（见证打卡团）的服务器，从古老的 Ubuntu 12.04 连续升级了三次，到了最新的 LTS 版本，18.04。
Ubuntu 12.04 -\> Ubuntu 14.04
Ubuntu 14.04 -\> Ubuntu 16.04
Ubuntu 16.04 -\> Ubuntu 18.04
你可能会问，为什么不直接从 12.04 升级到 18.04 呢？这是因为系统不支持这样升级，只能两个大版本一步一步升级。

## 升级方法

其实三次升级的方法（或者说过程）都是一样的。网上文章也有很多，我主要参考的是[这篇文章](https://www.mobibrw.com/2016/3789)，一方面是因为这篇文章的作者的服务器也是在阿里云上，跟我的一样，二是因为这篇文章写得比较清楚，一步一步截图都有。
在开始之前，可以考虑把 apt 的 sources list 换成国内的镜像。如果是阿里云服务器，建议换成阿里云的镜像，腾讯云的服务器，换成腾讯云的镜像，其他云的服务器也类似。如果不是哪个云的服务器，可以考虑换成中科大或清华等国内镜像。这样，在升级过程中，下载很多 package 的速度会快很多很多。我的服务器是阿里云的，因此换成了阿里云的镜像。
换好之后，执行：

```
$sudo apt-get update
$sudo apt-get upgrade
$sudo apt-get dist-upgrade
$sudo apt-get install update-manager-core
$sudo do-release-upgrade
```

执行 `do-release-upgrade` 之后，系统升级便开始了，这个过程中会下载很多 package，所以最好保证你的服务器磁盘剩余空间是足够的（\>2G 就差不多了）。
在升级过程中，会询问你很多次“某个文件已被修改，是使用新系统版本的替换，还是保留当前版本”这样的问题。如下。

我的选择是，一律保留当前版本。

## 途中遇到的几个问题

### JDK8 下载时间太长了

在从 12.04 升级到 14.04 的过程中，系统给 JDK8 的小版本也做了一次升级，从某一个小版本升级到 u201。这个过程是直接从 oracle 的官网下载 JDK8 的安装包（jdk-8u201-linux-x64.tar.gz）。由于在国内的原因，连接 oracle 官网去下载这个将近 200M 的文件非常的慢。只有十几 K 的速度，预估需要 7、8 个小时。这可是不能接受的。由于我没有在服务器上面设置 proxy，临时去配置 FQ 的工作还是比较麻烦的。解决的办法是，先终止这个升级的过程，用自己的 Mac 下载好那个安装包文件，然后找到服务器上 apt 的缓存目录，scp 过去，然后再次启动升级，一切就很顺利了。

### locale 的问题

Locale 的问题，这三次升级以后都遇到了，每次升级完，在命令行运行任何命令，都总是提示：
`/bin/bash: warning: setlocale: LC_ALL: cannot change locale`
当然，如果只是个 warning，那没太大问题，问题是启动 postgresql 的时候，也老是提示类似的问题，并且导致 postgresql 启动不起来。这个问题折腾了好一会，试过很多办法。有在 environment 里面加了两行`LC_ALL`和`LC_LANG`定义，

    LC_ALL=en_US.UTF-8
    LC_LANG=en_US.UTF-8

也有在命令行里面直接定义 export 这两个环境变量。不过都不管用，我想真正生效的是，切换到 root 用户（`su root`），然后执行`locale-gen en_US.UTF-8`。然后再加上上面提到的步骤。最后才解决这个问题。
令我感到奇怪的是，我直接`sudo locale-gen ...`是不行的，一定要 su 到 root 用户才可以。

### mount 被锁住

从 12.04 升级到 14.04 之后，想继续升级到 16.04，执行 `do-release-upgrade`，运行了一段时间之后，升级失败，抛出一个错误。

    Calculating the changes

    Could not calculate the upgrade

    An unresolvable problem occurred while calculating the upgrade.

    This can be caused by:

    * Upgrading to a pre-release version of Ubuntu

    * Running the current pre-release version of Ubuntu

    * Unofficial software packages not provided by Ubuntu

    If none of this applies, then please report this bug using the

    command 'ubuntu-bug ubuntu-release-upgrader-core' in a terminal.

    Restoring original system state

    Aborting

    Reading package lists... Done

    Building dependency tree

    Reading state information... Done

在网上搜了下，提示我查看错误日志：`/var/log/dist-upgrade/main.log`，日志里面提示：`Dist-upgrade failed: 'E:Unable to correct problems, you have held broken packages.'` 但是没有跟具体的提示，没有说明是哪个 broken package 导致了这个问题。网上搜了下，找到[这个答案](https://askubuntu.com/questions/805175/unable-to-upgrade-from-14-04-to-16-04-could-not-calculate-the-upgrade-an-unres)，提示执行 `grep Broken /var/log/dist-upgrade/apt.log` 把 broken 的 package 一个个删了，但是我在我的服务器上执行这个命令，grep 出来有 800 多个 broken package，一个个删，显然是不现实的。正想放弃的时候，又想到文章开头提到的那篇文章的作者，说不定他也遇到了这样的问题，说不定还写了另外一篇文章。因此翻了下他的博客，果然，他还写了一篇[把 Ubuntu 从 14.04 升级到 16.04 的文章](https://www.mobibrw.com/2017/7068/comment-page-1#comment-3479)。而且文章中也提到了类似的问题，虽然具体的问题描述跟我遇到的不完全一样，但是我还是按照他说的解决办法执行了一遍，果然，问题解决了。
我把其中的那段解决问题的文章原封不动的搬过来在这里：

---

执行如下命令，查看包管理策略：

    $ apt-cache policy

可以看到如下信息：

```
	Pinned packages:

	     mount -> 2.20.1-1ubuntu3

	     linux-image-3.2.0-29-generic -> (not found)
```

信息显示，mount 被锁定到了版本 2.20.1 上，导致无法升级替换。
我们执行如下命令移除锁定策略：

    $ sudo mv /etc/apt/preferences.d /etc/apt/preferences.d.old

接下来继续执行升级命令：

    $ sudo do-release-upgrade

一路回车，什么都不填，一切都不更改，大约 20-30 分钟之后，系统提示重启。

---

BTW，也正是由于这位大神的文章在这次升级中对我的巨大帮助，才促使我写了这篇文章，我想，如果这篇文章能帮助到其他人，也算是一种报恩了。

### /run/权限

第一次系统升级以后，我的一个 rails app 启动不起来。启动 server 的命令提示是成功的，但是网站页面访问不了。由于我的这个 rails app 是用 nginx reverse proxy 到一个 unix socket（其实就是一个文件），再到 rails app server 的，于是我怀疑是不是这个 unix socket 创建没成功。到存放这个 unix socket 文件所在的目录下面去看，果然没有。那么为什么这个文件会创建不成功呢？最常见的问题，就是没有权限。
`ls /the/socket/dir`，显示 owner 是`root`，其他用户只有可读权限。执行
sudo chown `whoami` /the/socket/dir
把这个目录的 owner 改成当前用户，然后再次启动 server，app 就能正常访问了。

虽然这次升级的途中遇到一些问题，然而回过头来想想，应该说，总体来说这三次升级还算是顺利的。尤其是，如果你对比 Windows 的升级的话，这几次 Ubuntu 的升级体验，可以算是丝般顺滑了。
