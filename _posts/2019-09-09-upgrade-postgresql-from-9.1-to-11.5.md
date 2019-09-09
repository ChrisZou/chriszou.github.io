---
layout: post
title: PostgreSQL 数据库从9.1版本升级到11.5
date: 2019-09-09T08:18:12+08:00
comments: true
post_id: upgrade-postgresql-from-9.1-to-11.5
---

## 起因

===  
如果你不想听我自言自语，请跳到以下“升级”部分。  
===  

随着年龄越来越大，我在技术上的“最求”也变得更保守了，慢慢的，我开始信奉这样一句名言

> Don't fix what's not broken
>
> -- 不知道是谁说的不过不是我

这次选择这么大跨度的一个升级（很大吗？是的，9.1版本是2011年发布的，那一年，我还没毕业）自然是有原因的。  
之前说过，打卡团服务器是用Elixir（一门语言）/Phoenix（一个Web框架）写的，这是个错误。至于为什么说这是个错误，以后有时间写写（这句话好像之前说过？）。  
于是，最近一直想把它迁移到我信仰的Ruby on Rails（RoR）。当然，“迁移”是没时间迁移的，永远都没时间迁移。准确的说，是新功能用RoR开发，原来的功能就留着（“Don't fix what's not broken”，remember?）。  
但是搭建好Rails scaffold之后，发现数据库连接不上。老是提示这样一个错误:

```
Traceback (most recent call last):
        2: from (irb):4
        1: from (irb):4:in `rescue in irb_binding'
ActiveRecord::StatementInvalid (PG::UndefinedTable: ERROR:  relation "pg_range" does not exist)
LINE 3: LEFT JOIN pg_range as r ON oid = rngtypid
```

简单说，就是 `pg_range` 这个table找不到。经过一番google，发现 `pg_range` 这个table是 psql 自带的，自从9.2版本开始引入。所以自然很容易猜想，是不是我Server上的psql版本过低呢？查看一下：

```
$ psql --version
psql (PostgreSQL) 10.6 (Ubuntu 10.6-0ubuntu0.18.04.1)
```

显示版本号是10.6，所以看起来不是这个原因。那又会是什么原因呢？又经过一顿Google猛如虎，还是没辙。只好去StackOverflow（一下简称SO）提问，不久之后，就有个人回答了[我的问题](https://stackoverflow.com/questions/57765528/rails-activerecordstatementinvalid-pgundefinedtable-error-relation-pg-ra)。  
原来，`psql --version` 显示的是psql client的版本号，psql server的版本号跟这个不一定是一样的。按照SO那位盆友的回答，登录psql shell，执行 `SHOW server_version;`，就可以看得很清楚了：

```
$ psql -U xxx
...
xxx=# SHOW server_version;
 server_version
----------------
 9.1.24
(1 row)
```

可以看到，我的psql server版本号其实是9.1.24。并不是我以为的10.6。  
后来我发现，实际上，你每次登录psql shell，它都会提示你版本号信息：

```
psql -U xxx
Password for user xxx:
psql (10.6 (Ubuntu 10.6-0ubuntu0.18.04.1), server 9.1.24)
```

仔细看最后一行结尾部分，人家说的很清楚，**server 9.1.24**。  
只是之前我一直没仔细看，没有注意到这个细节，不然的话，应该可以少走些弯路。

## 升级
所以接下来的工作就很明确了，升级psql server version。

简单google了一下，找到一个[SO回答](https://dba.stackexchange.com/questions/60465/upgrading-from-postgres-9-1-to-9-3-on-ubuntu-server)。照着做，一路下来又是如丝般顺滑的体验（为什么要说“又”），整个过程十分钟之内就搞定了，这真的要给psql点个大大的赞！

具体过程如下：

### 1. 安装新版psql
是的，你没看错，升级psql版本的方式，是先安装两个版本的psql，然后在执行升级，而不是直接在原有的基础上直接执行升级。  
安装新版psql的方式，在[官网](https://www.postgresql.org/download/)上有说明。以下简述在Ubuntu 18.04上面的安装方式。

- 创建文件/etc/apt/sources.list.d/pgdg.list，并写入以下内容

```
deb http://apt.postgresql.org/pub/repos/apt/ bionic-pgdg main
```

- 执行

```
$ wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
$ sudo apt-get update
```

### 2. 检验新版psql安装结果
执行 `sudo pg_lsclusters`，你应该可以看到你机器上有两个psql clusters在运行着。

```
$ sudo pg_lsclusters
Ver Cluster Port Status Owner    Data directory               Log file
9.1 main    dddd online xxx /var/lib/postgresql/9.1/main /var/log/postgresql/postgresql-9.1-main.log
11  main    dddd online xxx /var/lib/postgresql/11/main  /var/log/postgresql/postgresql-11-main.log
```

### 3. 杀掉所有psql进程

```
sudo service postgresql stop
```

### 4. 删除在安装过程中，自动创建的新版psql cluster

```
sudo pg_dropcluster --stop 11 main
```

### 5. 基于老版psql的数据，创建新版psql cluster

```
sudo pg_upgradecluster 9.1 main
```

这是最核心的一个步骤，在这个过程中，psql会自动的把你的psql数据文件从老版数据格式转换成新版数据格式。我在执行这一步的时候，还碰到了一个问题，具体后面再叙述。

### 6. 启动新版psql

```
sudo service postgresql start 11
```

现在，你的psql就已经升级成功，并且运行起来了。  
运行一段时间观察一下，如果没问题的话，你就可以选择把老版的psql数据文件删掉，以节省磁盘空间。

```
pg_dropcluster --stop 9.1 main
```

### 遇到的问题
我在执行以上的第5步的时候，我遇到了一个错误。错误提示是：

```
Error: The locale requested by the environment is invalid:
  LANG: en_US.UTF-8
  LANGUAGE: en_US:en
  LC_CTYPE: UTF-8
Error: Could not create target cluster
```

看起来，是几个locale相关的环境变量没设置正确。其中 `LANGUAGE` 和 `LC_CTYPE` 两个变量一看就不对，因为没有`en_US:en`或`UTF-8`这样的locale值。最简单快捷的办法，是直接在shell里面临时设置成一个万能的值。

```
export LANGUAGE="en_US.UTF-8"
export LC_CTYPE="en_US.UTF-8"
```

再次运行，发现就没问题了。

参考资料：
- https://www.postgresql.org/download/linux/ubuntu/
- https://dba.stackexchange.com/questions/60465/upgrading-from-postgres-9-1-to-9-3-on-ubuntu-server
