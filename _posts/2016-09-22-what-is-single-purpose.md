---
layout: post
title: what is single purpose
date: 2016-09-22T22:25:09+08:00
comments: true
post_id: what-is-single-purpose
---

# 什么叫"功能单一"

昨天开始看《Build Awesome Command-Line Applications in Ruby 2》这本书，看完第一章。其中提到一个Awesome的Command，应该是功能单一（single-purpose）的，但是具体什么叫功能单一呢？为了解释这个问题，书中举了两个反例，来解释什么不是功能单一。  
第一个例子是一个备份数据库的脚本。其中需要备份的数据库有两个，这个脚本一次性把两个脚本都备份了，脚本大概是这么写的：

```ruby
#!/usr/bin/env ruby
# File: db_backup.rb
#两个数据库的信息
databases = {
  big_client: {
    database: 'big_client',
    username: 'big',
    password: 'big',
  },
  small_client: {
    database: 'small_client',
    username: 'small',
    password: 'p@ssWord!',
  }
}

databases.each do |name,config|
  #依次备份每个数据库
  #...
end
```

使用的时候，只需要运行一个这个命令就好了。但是这个的结构其实并不好，也并不是"功能单一"，之所以这么说，是因为它一次性备份了两个数据库，而不是一个。试想一下，如果后来又有一个数据库需要备份，那么就得改这个脚本，修改上面`databases`这个Hash（Map、Dictionary，whatever）。这个可不是一个好主意，如果别人给你一个备份数据库的脚本，然后你说要再备份一个数据库，结果他说你要改那个脚本，这时候你的感觉可不会太好，如果这个人离职了呢？如果他用的是一门你不熟悉的语言写的呢？如果他用的是write-only的perl写的呢？  
所以对于这个脚本，更好的组织方式是，脚本本身只备份一个数据库，并且需要的数据库信息是从外面传进来的。  
比如可以这样写：

```ruby
#!/usr/bin/env ruby
# File: db_backup.rb
db_name = ARGV[0]  #ARGV[0]就是传给这个脚本的第一个参数，以下类推
username =  ARGV[1]
password = ARGV[2]

# backup database use db_name, username, password
# ...
```

使用的时候：

```shell
./db_backup.rb database1 user1 password1
./db_backup.rb database2 user2 password2
```

如果需要再备份一个数据库，只需要再加一条命令就好了：

```shell
./db_backup.rb database1 user1 password1
./db_backup.rb database2 user2 password2
./db_backup.rb database3 user3 password3
```

实事上，你想备份多少个数据库都可以，都不用去修改代码。  

第二个例子，是一个命令行下的todo list 工具。这个工具有add task、complete task、list tasks三个功能。然后是这样设计的，第一个功能都有一个对应的脚本，即：

```shell
# 添加一个任务
./add_task.rb 'finish this post'

 # 将第一项任务标记为完成
./complete_task.rb 1

# 列出所有的任务
./list_tasks.rb
```

这三种脚本共用一个"tasks.txt"文本文件，作为task数据存储。这个设计也不好，一是这三个脚本共用一个"tasks.txt"，那么他们之间必然会需要share很多的logic，试想如果后面想要给task增加一个field，比如tag，那么这三个文件都得多相应的类似的修改。二是脚本太多暴露给用户，用户会疑惑，也记不清这么多脚本的名字。  
那么解决办法是什么呢？那就是将这些命令组合成一个Command suite，类似于git一样，将很多同一件事的命令归成一个命令，用"子"命令来区别不用的作用：

```shell
# 添加一个任务
./todo new 'finish this post'

 # 将第一项任务标记为完成
./todo complete 1

# 列出所有的任务
./todo list
```

这样只有一个todo脚本，里面很多share logic不会造成重复代码。同时只给一个顶层的命令给用户，更加干净、清爽！

我在想，这里的单一任务的概念和标准，能不能应用到我们做App开发的时候，设计一个类或一个模块呢？

其它的一些收获，Ruby相关，不感兴趣的朋友直接跳过吧。

## Ruby related
1. 向文件末尾添加一行代码：

```ruby
File.open('todo.txt', 'a') do |file|   # 'a' for append
  file.puts "your new line"
end
```

2. Array#shift：返回第一个元素，并从这个数组里面remove掉

```ruby
arr = ["a", "b", "c"]
arr.shift #=> "a"
arr #=> ["b", "c"]
```
3. `a, b = ['hello', 'world']   #=> a = 'hello', b = 'world'`

4. 判断是不是nil：`if str.nil?`，虽然你可以用`if !str`但总感觉不是很优雅，并且可读性也不高。

哦对了，flagship是"旗舰"是意思，比如flagship store，旗舰店。
