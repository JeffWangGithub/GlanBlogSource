---
title: Mac提升效率的常用命令
date: 2018-08-24 10:29:38
categories: Mac
tags:
---
### 1.  open
open 命令在命令行中打开某个软件，相当于双击打开
- open `文件` -e : 强制在 EditText 软件中打开此文件
- open `目录` : 使用 finder 打开目录， open 。 打开当前目录
- open `文件` -a ` 应用`: 使用某个应用打开文件 

### 2. find
find命令用来在指定目录下查找文件。任何位于参数之前的字符串都将被视为欲查找的目录名。如果使用该命令时，不设置任何参数，则find命令将在当前目录下查找子目录与文件。并且将查找到的子目录和文件全部进行显示。
- find `路径` -name `正则表达式` : 在指定路径下查找符合正则的文件
- find `路径` -iname `正则表达式` : 在指定路径下查找符合正则的文件,忽略文件大小写
- find `路径` -name  `条件1` -o -name 条件2  : 查找条件一或条件2
-  find `路径` -path `条件` : 匹配文件路径或者文件
- find `路径` -regex `正则表达式` :指定目录下根据正则匹配文件路径
- find `路径` ! -name `条件` : 否定参数，指定路径下条件不成立的文件
- find `路径` type `类型` : 根据文件类型查找。常用类型: f 普通文件； d 目录
- find `路径` type  f -size `文件大小` : 根据文件大小匹配，大小中正负号代表大于和小于。 find . -type f -size +10K 大于10K 的文件
- find `路径` type  f -name `条件` -delete :根据条件删除
- find `路径` -empty :列出长度为0的文件
### 3.  pbcopy 和 pbpaste
pbcopy 指 copy 到剪切板； pbpaster 指从剪切板复制出来。
- ls ~ | pbcopy :利用管道，将~目录下的文件 copy 到剪切板
- pbcopy < a.txt : 将文件内容 copy 到剪切板
- pbpaste >> b.txt :将剪切板内容粘贴到文件

### 4. say 
一个有趣的命令，将文本转换成语音。
- say "hello world ; 王"
- say < ~/a.txt :朗读文件内容
### 5. brew
homebrew 一个mac 上的软件下载安装工具，详情搜索 homebrew 官网