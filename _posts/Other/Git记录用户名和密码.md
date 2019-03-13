---
title: Git常用命令以及常见问题处理
date: 2016-08-18 10:29:24
tags: git 
categories: Other
---
Mac下使用git进行操作时，总是会提示输入用户名和密码，特别蛋疼，今天查了一下如何让系统记录我们的用户名和密码
`git config --global credential.helper store`
使用这个会在当前用户根目录下创建一个.git-credentials的文件用于明文保存用户名密码及相关链接。 OK试试吧，看看系统是不是自动记录了你的用户名和密码
