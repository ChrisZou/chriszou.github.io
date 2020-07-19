---
layout: post
title: 把 Postgresql 数据库内容从 10.xxx 升级到 11.2
date: "2019-03-19T18:00:19+08:00"
comments: true
---

我 Mac 本地的 psql version 不知道何时何因被升级到了 11.1_1。今天尝试使用的时候，提示：

    FATAL: database files are incompatible with server
    DETAIL: The data directory was initialized by PostgreSQL version 10.xxx, which is not compatible with this version 11.1_1

Google 了一下，找到这个命令可以用：

    brew postgresql-upgrade-database

运行过程中，会先下载老版的 10.xx 版本，然后再执行数据库的升级。到后面出现这个错误：

```
==> Moving postgresql data from /usr/local/var/postgres to /usr/local/var/postgres.old...
...
Error：.... Permission denied
```

看错误的提示，应该是`/usr/local/var/`目录的权限问题。执行

    sudo chmod a+w /usr/local/var

然后再执行

    brew postgresql-upgrade-database

这次不提示 permission 的问题了，但是出来另外一个问题：

```
==> Moving postgresql data from /usr/local/var/postgres to /usr/local/var/postgres.old...
Error: Upgrading postgresql data from 10 to 11 failed!
==> Removing empty postgresql initdb database...
==> Moving postgresql data back from /usr/local/var/postgres.old to /usr/local/var/postgres...
==> Successfully started `postgresql` (label: homebrew.mxcl.postgresql)
Error: Failure while executing `/usr/local/Cellar/postgresql/11.2/bin/pg_upgrade -r -b /usr/local/Cellar/postgresql@10/10.6_1/bin -B /usr/local/Cellar/postgresql/11.2/bin -d /usr/local/var/postgres.old -D /usr/local/var/postgres -j 12` exited with 1.
```

最根源的问题应该出现在最后一行：

    Error: Failure while executing /usr/local/Cellar/postgresql/11.2/bin/pg_upgrade -r -b /usr/local/Cellar/postgresql@10/10.6_1/bin -B /usr/local/Cellar/postgresql/11.2/bin -d /usr/local/var/postgres.old -D /usr/local/var/postgres -j 12

可以看到，运行了`/usr/local/Cellar/postgresql/11.2/bin/`目录下面的`pg_upgrade` 这个脚本。这就奇怪了，因为我 local 的 psql version 是 11.1_1 啊，会什么会去找 `/usr/local/Cellar/postgresql/11.2` 这个目录呢？我的猜测是这样的：psql 目前官方最新的发布版本是 11.2，于是执行`brew postgresql-upgrade-database`的时候，brew（或 psql）的升级脚本默认用户这时候已经安装了最新的 psql 版本（11.2），于是到相应的版本目录（`/usr/local/Cellar/postgresql/11.2/bin/`）下面去找`pg_upgrade`这个脚本。而我本地的版本是 11.1_1，于是找不着 11.2 版本的路径，于是出现了上面这个问题。
那么，我把本地的 psql 版本升级到 11.2，总行了吧？执行

    brew upgrade postgres

成功升级到 11.2。再一次执行

```
$ brew postgresql-upgrade-database
==> Upgrading postgresql data from 10 to 11...

Performing Upgrade
...
Upgrade Complete
----------------
...
==> Upgraded postgresql data from 10 to 11!
==> Your postgresql 10 data remains at /usr/local/var/postgres.old
==> Successfully started `postgresql` (label: homebrew.mxcl.postgresql)
```

这次终于成功了，Yay！
