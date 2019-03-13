---
title: git仓库迁移
date: 2017-06-27 09:10:41
tags: git迁移
categories: Other
---
git仓库从一台服务器迁移到另外一台服务器:
1) 从原有仓库克隆一份裸版库，sample

```
git clone --bare git://github.com/username/project.git
```
裸版意思是没有工作区，clone的是当前的整个版本库
2) 在新的git服务器上创建一个新的项目，比如gitPro
3) 以镜像的方式推送到gitPro服务器上

```
cd project
git push --mirror git@gitPro.com/username/newproject.git
```
4) 本地代码修改origin url或者重新clone
修改origin url

```
cd 进入之前的本地仓库目录
git remote set-url origin git@gitPro.com/username/newproject.git
```
或者重新clone一个工程
```
git clone git@gitPro.com/username/newproject.git
```
OK，搞定，尽情的享受吧。