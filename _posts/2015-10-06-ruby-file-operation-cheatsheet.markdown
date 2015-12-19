---
layout: post
title:  "Ruby 文件操作 Cheatsheet"
tags: [tech, ruby]
date:   2015-10-06 15:51
comments: true
---
每次都要查，真是蛋疼，不如一次性总结一下，以后再不记得就来这里找好了。  
以下代码中需要用到的文件名：`filename = ‘testfile.txt’`

- 读取其中的全部内容：`File.read(filename)`
- 将一个字符串一次性写入这个文件：
`File.write(filename, str)`
- 读取文件内容，依次处理其中的每一行：

```
File.foreach(filename) do |line|
    #process each line
end
```

- 判断文件是否存在：`File.exists?(filename)`
- 删除文件：`File.delete(filename)`
- 文件重命名：`File.rename(filename, new_name)`
- 文件的名字：`file.path  # file是一个File对象`
- 文件的绝对路径：`File.absolute_path(filename)`

- 当前所在目录：`Dir.pwd`
- 当前登录用户的home目录：`Dir.home`
- 创建文件夹：`Dir.mkdir(dir_name) #注意，如果父目录不存在的话，这里无法创建子目录`
- 删除文件夹：`Dir.rmdir(dir_name)`
- 创建多级目录(mkdir -p)：`FileUtils.mkdir_p(path) #这里需要require fileutils`
- 找出当前目录下的所有文件或目录：`Dir.glob("*")`
- 找出当前目录下的所有Ruby文件：`Dir.glob("*.rb")`
- 判断目录是否存在：`Dir.exists?(dir_name)`
- 用文件夹和文件组成一个路径：File.join(dir_name, filename)

- 当前文件（正在执行的文件）相对于当前所在目录（pwd目录）的相对路径：`__FILE__`
- 当前文件（正在执行的文件）的目录相对于当前所在目录（pwd目录）的相对路径：`File.dirname(__FILE__)`
- 当前文件（正在执行的文件）的绝对路径：`File.expand_path(__FILE__)`
- 当前文件（正在执行的文件）所在目录的绝对路径：`File.expand_path(File.dirname(__FILE__))`

最后附上`File.open(filename, mode)` 中的mode各种取值以及含义：  
Mode |  Meaning  
-----+--------------------------------------------------------  
"r"  |  Read-only, starts at beginning of file  (default mode).  
"r+" |  Read-write, starts at beginning of file.  
"w"  |  Write-only, truncates existing file, to zero length or creates a new file for writing.  
"w+" |  Read-write, truncates existing file to zero lengthor creates a new file for reading and writing.  
"a"  |  Write-only, starts at end of file if file exists, otherwise creates a new file for writing.  
"a+" |  Read-write, starts at end of file if file exists, otherwise creates a new file for reading and writing.  
 "b" |  Binary file mode (may appear with any of the key letters listed above).   Suppresses EOL <-> CRLF conversion on Windows. And sets external encoding to ASCII-8BIT unless explicitly specified.  
 "t" |  Text file mode (may appear with any of the key letters listed above except "b").
