---
title: 打包脚本梳理.md
date: 2016-06-29 14:59:59
tags:
---
### 总体原理
```flow
st=>start: Start
e=>end: End
op1=>operation: 获取渠道信息:\n 
op2=>operation: 获取git源码:\n 
op3=>operation: 替换gradle编译脚本:\n 用服务器的autoPkg.gradle脚本\n 替换代码中的autoPkg.gradle脚本 
op4=>operation: Start build 开始编译
op5=>operation:  注入渠道信息\n重新进行签名打包\n:genchal.sh执行
st->op1->op2->op3->op4->op5->e
```
### 步骤分析
- 获取渠道信息:
    1. qudaoV0.1.sh执行，解析.xls文件中的渠道信息。通过xlsx2cvs插件解析xlsx文件中的渠道信息
- 获取git源码
    1. git_getcode.sh脚本执行，获取所有分支源代码
    2. 根据git_getcode.sh脚本中指定的分支切换到对应分支
    3. 将对应分支的代码copy到newreader指定目录下
- 替换.gradle编译脚本
    1. 使用服务现有的gradle编译脚本替换代码中的脚本文件， 防止git服务商的gradle脚本被错误修改
- 开始编译源码
    1. 执行build.gradle脚本文件开始编译源码
    2. 将生成的apk copy到渠道信息所在目录
- 注入渠道信息
    1. 解压apk文件，遍历注入渠道信息和配置信息(向ntescfg中注入相关配置信息信息)
    2. 重新压缩签名，生成新的apk，并放置到不同的渠道目录下