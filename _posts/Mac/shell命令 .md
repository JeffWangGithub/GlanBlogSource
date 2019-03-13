---
date: 2015-04-16 23:18
status: public
title: shell命令
categories: Mac
---

### 1,echo :变量的取用与配置
```
//去除摸个变量
[root@www ~]# echo $PATH
/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin
[root@www ~]# echo ${PATH}
```
```
//修改配置变量，将 myname 这个变量名称的内容配置为 VBird
[root@www ~]# echo $myname
       <==这里并没有任何数据～因为这个变量尚未被配置！是空的！
[root@www ~]# myname=VBird
[root@www ~]# echo $myname
VBird  <==出现了！因为这个变量已经被配置了！
```
```
扩增变量内容时，用 "$变量名称" 或 ${变量} 累加内容，如下所示：
『PATH="$PATH":/home/bin』
```
### 2，export：将变量变成环境变量，一边可以在其他子程序中使用
`什么是『子程序』呢？`就是说，在我目前这个 shell 的情况下，去激活另一个新的 shell ，新的那个 shell 就是子程序啦！在一般的状态下，父程序的自定义变量是无法在子程序内使用的。但是透过 export 将变量变成环境变量后，就能够在子程序底下应用了
```
如何让我刚刚配置的 name=VBird 可以用在下个 shell 的程序？
[root@www ~]# echo ${name}
[root@www ~]# name=VBird
[root@www ~]# bash        <==进入到所谓的子程序
[root@www ~]# echo $name  <==子程序：再次的 echo 一下；
       <==嘿嘿！并没有刚刚配置的内容喔！
[root@www ~]# exit        <==子程序：离开这个子程序
[root@www ~]# export name
[root@www ~]# bash        <==进入到所谓的子程序
[root@www ~]# echo $name  <==子程序：在此运行！
VBird  <==看吧！出现配置值了！
[root@www ~]# exit        <==子程序：离开这个子程序
```
### 3, unset 取消刚刚配置的变量内容
```
[root@www ~]# unset name
```
### 4,特殊符号的使用
### #4.1 变量配置中单引号与双引号的区别
单引号与双引号的最大不同在于双引号仍然可以保有变量的内容，但单引号内仅能是一般字符 ，而不会有特殊符号。我们以底下的例子做说明：假设您定义了一个变量， name=VBird ，现在想以 name 这个变量的内容定义出 myname 显示 VBird its me 这个内容，要如何订定呢？
```
[root@www ~]# name=VBird
[root@www ~]# echo $name
VBird
[root@www ~]# myname="$name its me"
[root@www ~]# echo $myname
VBird its me
[root@www ~]# myname='$name its me'
[root@www ~]# echo $myname
$name its me
```
发现了吗？没错！使用了单引号的时候，那么 $name 将失去原有的变量内容，仅为一般字符的显示型态而已！这里必需要特别小心在意！
### #4.2 反单引号( ` )这个符号代表的意义为何
在一串命令中，在 ` 之内的命令将会被先运行，而其运行出来的结果将做为外部的输入信息！例如 uname -r 会显示出目前的核心版本，而我们的核心版本在 /lib/modules 里面，因此，你可以先运行 uname -r 找出核心版本，然后再以『 cd 目录』到该目录下
```
如何进入到您目前核心的模块目录？
[root@www ~]# cd /lib/modules/`uname -r`/kernel
[root@www ~]# cd /lib/modules/$(uname -r)/kernel
```
`使用『 version=$(uname -r) 』来取代『 version=`uname -r` 』比较好，因为反单引号大家老是容易打错或看错`
### 常用案例
`若你有一个常去的工作目录名称为：『/cluster/server/work/taiwan_2005/003/』，如何进行该目录的简化？`
在一般的情况下，如果你想要进入上述的目录得要『cd /cluster/server/work/taiwan_2005/003/』， 以鸟哥自己的案例来说，鸟哥跑数值模式常常会配置很长的目录名称(避免忘记)，但如此一来变换目录就很麻烦。 此时，鸟哥习惯利用底下的方式来降低命令下达错误的问题：
```
[root@www ~]# work="/cluster/server/work/taiwan_2005/003/"
[root@www ~]# cd $work
```
未来我想要使用其他目录作为我的模式工作目录时，只要变更 work 这个变量即可！而这个变量又可以在 bash 的配置文件中直接指定，那我每次登陆只要运行『 cd $work 』就能够去到数值模式仿真的工作目录了！是否很方便呢？ ^_^


