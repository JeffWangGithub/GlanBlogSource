---
title: Git常用命令以及常见问题处理--update
date: 2016-11-11 10:29:24
tags: git 
categories: Other
---
1. 直接clone远程指定分支到本地指定的路径
     ```
      git clone -b 远程分支名 git远程地址 本地存储目录
    ```

2.  基于远程的分支创建新的分支    
    ```
        git checkout -b 新建分支名 origin/远程分支名
    ```
3. 代码恢复
     git reset 后会有很多子指令，常用的又三中soft, mixed, hard三中的区别

    ```
        - soft 代码最小影响指令: 使用soft时，仅仅改变了head的指向，不会影响代码的索引。
        - mixed 混合指令: 此指令的影响范围
         - head指向恢复的索引
         - 取消diff代码的索引
        - hard 强硬指令: 强制回复到指定提交处，不保留任何任何diff代码  
    ```
4. git cherry-pick 不太常用，了解

    ```
        - 将某个commit取出来，并入其他分支上。此功能可以用patch来替换解决
        - 重建一系列的提交。即改变各个提交的顺序。可能产生一些列的冲突
    ```
5. patch简单使用
    patch的生成又多重方式git diff和git formate-patch都是命令都过于复杂灵活度不高，一般生成patch时都是使用一个图形化工具。

    ```
    - git formate-patch -M 指定分支: 当前分支与指定分支对比生成patch文件，每一个commit生成一个patch文件
    - git formate-patch commitID: 某次提交之后所有的change生成patch,每一个commit生成一个patch文件
    - git formate-patch commitID1..commitID2: 两个commit之间的所有patch文件
    - git am patchName :应用补丁    
    ```
6. 本地分支与远程分支进行关联
    切换分支后忘记关联远程分支，又不想将代码merge到原分支，可以将当前分支与远程分支进行关联，然后pull，push即可将修改提到服务器

    ```
    git branch --set-upstream-to=origin/...
    ```
7. 将本地分支与远程分支解除关联：
    ``` 
    没有找到git相关命令，但是知道其原理，最终都是修
    改config 文件，所以只需要使用vi命令打开.git/config文件，然后
    将 [branch "分支名"]以及其中的内容删除了即可。
    ```
8. 修改上一次的commit内容
    ```
        - 修改需要更改的文件，然后执行git add 修改后的文件
        - 执行 git commit --amend   修改最后一次的commit，
    ```
9. 将某些文件视为未修改的假状态(不太常用)
    ```
      - git update-index --assume-unchanged filePath  将某个文件的修改视作未修改，git status命令中不现实修改
      - git update-index --no-assume-unchanged filePath 回复filePath的视作未修改
    ```
10. 去除git追踪
    ```
        git rm --cached file 把文件标记为未追踪,此命令也可以将文件从已经暂存状态变成未暂存未追踪态
       git rm file  将文件从版本库和工作目录中同时删除
    ```
11. 合并多个commit， 优化log信息
 git rebase -i commitID   从commitID到head之间的所有commit都会被合并到一个

    ```
    pick 9a54fd4 添加commit的说明
        pick 0d4a808 添加pull的说明
    
        # Rebase 326fc9f..0d4a808 onto d286baa
        #
        # Commands:
        #  p, pick = use commit
        #  r, reword = use commit, but edit the commit message
        #  e, edit = use commit, but stop for amending
        #  s, squash = use commit, but meld into previous commit
        #  f, fixup = like "squash", but discard this commit's log message
        #  x, exec = run command (the rest of the line) using shell
        # If you remove a line here THAT COMMIT WILL BE LOST.
        # However, if you remove everything, the rebase will be aborted.
    ```
    - 将除第一个以外的commit 前的pick修改为squash，然后保存退出
12. .gitignore 配置攻略
    ```
    gitignore 配置文件用于配置不需要加入Git版本管理的文件，配置好该文件可以为我们的版本管理带来很大的便利。2. 配置语法
    以斜杠 “/” 开头表示目录；以星号 “*” 通配多个字符；以问号 “?” 统配单个字符；以方括号 “[]” 包含单个字符的匹配列表；以感叹号 “!” 表示不忽略（跟踪）匹配到的文件或目录。3. 示例
    规则： fd1/*
    说明： 忽略目录 fd1 下面的全部内容；注意，不管是根目录下的 /fd1 目录，还是某个子目录 /child/fd1/ 目录，都会被忽略；
    规则：/fd1/*
    说明：忽略根目录下的 /fd1/ 目录的全部内容；
    规则：
    /*
    !.gitignore
    !/fw/bin/
    !/fw/sf/
    说明：忽略全部内容，但是不忽略 .gitignore 文件、根目录下的 /fw/bin/ 和 /fw/sf/ 目录。
    ```

13. git 移除add过的文件
方法一:git  rm --cache 文件
方法二: git reset head 文件/文件夹


