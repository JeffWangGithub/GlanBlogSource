---
Title: Mac下配置hosts
date: 2015-04-03 10:30
status: public
categories: Mac
---

###   Mac OS中对hosts文件进行又该的几个命令使用


下面简单记录以下mac os下修改hosts的命令：
Mac OS下hosts文件所在的位置为/etc/hosts
1, 获取su权限,并修改hosts的权限。
`sudo chmod 777 /etc/hosts`
2, 使用vi 命令对hosts文件进行编辑
`vi /etc/hosts`

###   Vi命令的简单使用
```
1. 在默认的"指令模式"下按 i 进入编辑模式 
2. 在非指令模式下按 ESC 返回指令模式 
3. 在"指令模式"下输入: 
:w 保存当前文件 
:q 退出编辑,如果文件为保存需要用强制模式 
:q! 强制退出不保存修改 
:wq 组合指令, 保存并退出 
在「命令行模式（command mode）」下按一下字母「i」就可以进入「插入模式（Insert mode）」，这时候你就可以开始输入文字了
4. 在"指令模式"下移动: 
h 左 
j 下 
k 上 
l 右 
```