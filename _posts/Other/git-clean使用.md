---
title: git-clean使用
date: 2018-07-09 14:12:57
tags: git clean
categories: Other
---
### 问题
一个分支结构目录发生了很大的改变(例如新增了几个 module)或者是进行了代码工程结构的调整；此时如果从该分支切换到其他分支，由于配置了 .gitignore 文件，新增的 module产生的临时文件或者一些untracked 文件不会被自动删除；此时工程目录结构就会产生很多对当前分支来说多余的文件夹。那么遇到这种问题如何解决呢？
### git clean
`git-clean - Remove untracked files from the working tree ` 这是 git 对 git clean 的解释，删除untracked 文件从 git 工作树中。
 -d 删除包含 untracked文件的 untracked 目录。如果此 untracked 文件夹被其他 git 仓库管理，也不会被删除。
-f --force 强制执行 git clean 的操作。默认 git clean 需要配置 clean.requireForce 属性
-n --dry-run 不实际的进行移除操作，仅仅列出需要 clean 的文件
-X 仅删除.gitignore里标记过的文件，那些既不被git版本控制，又不在.gitignore中的文件会被保留。

因此，针对上述的问题，我们只需要在 git checkout 目录之后执行 git clean -df 命令就可以清理待多余的文件夹，拥有一个清新的目录结构