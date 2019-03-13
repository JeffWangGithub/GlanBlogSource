---
title: Shell相关
date: 2016-07-02 17:51:57
tags: shell
categories: Mac
---
1. 获取数据的长度
    ```
    array=(1 2 3 6 7)
    echo ${#array[@]}
    ```
2. 在脚本中倒引号表示里边的是命令
    ```
        [`命令`]
    ```
3. dirname和basename
    - dirname $0  获取当前脚本文件所在文件夹的路径
    - basename $0 获取当前脚本文件的名称
    ```
    echo $(dirname $0)
    echo $(basename $0)
    echo $(dirname $0)/$(basename $0)
    ```

4. $+符号相关含义
    - $$:  shell本身的PID(processID)
    - $1:  shell最后运行的后台Process的PID
    - $?:  最后运行的命令的结束代码(返回值)
    - $-: 使用set命令设定的Flag一览
    - `$*:  如果此符号用双引号包裹，则以"$1 $2 … $n"的形式输出所有参数`
    - `$@: 与$*相同`
    - $#:  获取到传递的参数个数
    - $0: Shell本身的文件名
    - `$1～$n` 获取第n个参数
5. local的含义
    ```
        1.    Shell脚本中定义的变量是global的，其作用域从被定义的地方开始，到shell结束或被显示删除的地方为止。
        2.    Shell函数定义的变量默认是global的，其作用域从“函数被调用时执行变量定义的地方”开始，到shell结束或被显示删除处为止。函数定义的变量可以被显示定义成local的，其作用域局限于函数内。但请注意，函数的参数是local的。
        3.    如果同名，Shell函数定义的local变量会屏蔽脚本定义的global变量。
    ```
6. 注意细节
    - shell的赋值"="等号前后不能有空格
    - 判断是否相等的等号(==或=) 前后要有空格
    
