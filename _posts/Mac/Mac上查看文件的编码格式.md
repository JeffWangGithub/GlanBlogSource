---
title: Mac上查看文件的编码格式
date: 2016-07-08 18:29:38
categories: Mac
tags:
---
Mac上竟然真心不知道改怎么查看一个文件的编码格式，于是乎google了一把，查到了一个比较简介的命令:enca
1. 查看你的系统有没有安装此命令，直接在终端执行enca命令，查看
2. 如果没有安装，直接brew install enca即可，安装完毕即可使用。(不要问哥brew是什么)

```
- enca file 查看file文件的编码格式
- enca -x utf-8 file 将file文件转换为utf-8的格式
- 其他更多的使用查看 enca --help
```
