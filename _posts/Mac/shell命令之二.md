---
date: 2015-08-5 09:30
status: public
title: shell命令之二
categories: Mac
---
1. dirname 用于取指定路径所在的目录。
    如 dirname /home/ikidou   结果为 /home    
2. $(命令) 返回该命令的结果
    返回上层路径: echo `dirname $(pwd)`
3. xargs :将参数列表转换成小块分段传递给其他命令，以避免参数列表过长的问题。
-a file 从file中读入作为标准输入sdtin
`cat log.txt | xargs echo`  打印文件内容
-e flag 注意有的时候可能会是-E，flag必须是一个以空格分隔的标志，当xargs分析到含有flag这个标志的时候就停止。
    $ xargs -E 'ddd'  -a 1.txt echo
    aaa bbb ccc
    $ cat 1.txt |xargs -E 'ddd' echo
    aaa bbb ccc
-n num 后面加次数，表示命令在执行的时候一次用的argument的个数，默认是用所有的。
    $ cat 1.txt |xargs -n 2 echo
    aaa bbb
    ccc ddd
    a b
例子:删除git中除master意外的分支
git branch | grep -v master |xargs git -D branch 





