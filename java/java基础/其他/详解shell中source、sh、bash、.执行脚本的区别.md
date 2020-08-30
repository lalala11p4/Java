### 详解shell中source、sh、bash、./执行脚本的区别

**1、source命令用法：**

　　source FileName

　　作用:在当前bash环境下读取并执行FileName中的命令。该filename文件可以无"执行权限"

  注：该命令通常用命令“.”来替代。

  如：source .bash_profile

​    . .bash_profile两者等效。

  source(或点)命令通常用于重新执行刚修改的初始化文档。

  source命令(从 C Shell 而来)是bash shell的内置命令。

  点命令，就是个点符号，(从Bourne Shell而来)。

**2、sh和bash命令用法：**

   sh FileName

   bash FileName

   作用:在当前bash环境下读取并执行FileName中的命令。该filename文件可以无"执行权限"

   注：两者在执行文件时的不同，是分别用自己的shell来跑文件。

  sh使用“-n”选项进行shell脚本的语法检查，使用“-x”选项实现shell脚本逐条语句的跟踪，

  可以巧妙地利用shell的内置变量增强“-x”选项的输出信息等。

**3、./的命令用法：**

   ./FileName

   作用:打开一个子shell来读取并执行FileName中命令。

   注：运行一个shell脚本时会启动另一个命令解释器.

​     每个shell脚本有效地运行在父shell(parent shell)的一个子进程里.

​      这个父shell是指在一个控制终端或在一个xterm窗口中给你命令指示符的进程.

​     shell脚本也可以启动他自已的子进程.

​      这些子shell(即子进程)使脚本并行地，有效率地地同时运行脚本内的多个子任务.

**shell的嵌入命令：**

: 空，永远返回为true
.  从当前shell中执行操作
break 退出for、while、until或case语句
cd 改变到当前目录
continue 执行循环的下一步
echo 反馈信息到标准输出
eval 读取参数，执行结果命令
exec 执行命令，但不在当前shell
exit 退出当前shell
export 导出变量，使当前shell可利用它
pwd 显示当前目录
read 从标准输入读取一行文本
readonly 使变量只读
return 退出函数并带有返回值
set 控制各种参数到标准输出的显示
shift 命令行参数向左偏移一个
test 评估条件表达式
times 显示shell运行过程的用户和系统时间
trap 当捕获信号时运行指定命令
ulimit 显示或设置shell资源
umask 显示或设置缺省文件创建模式
unset 从shell内存中删除变量或函数
wait 等待直到子进程运行完毕

**下面再看下 shell 脚本各种执行方式（source ./\*.sh, . ./\*.sh, ./\*.sh）的区别**

结论一: ./*.sh的执行方式等价于sh ./*.sh或者bash ./*.sh，此三种执行脚本的方式都是重新启动一个子shell,在子shell中执行此脚本。

结论二: .source ./*.sh和 . ./*.sh的执行方式是等价的，即两种执行方式都是在当前shell进程中执行此脚本，而不是重新启动一个shell 而在子shell进程中执行此脚本。

验证依据：没有被export导出的变量（即非环境变量）是不能被子shell继承的

验证结果：

```shell
[root@localhost ~]#name=dangxu    //定义一般变量 
[root@localhost ~]# echo ${name} 
dangxu 
[root@localhost ~]# cat test.sh   //验证脚本，实例化标题中的./*.sh 
#!/bin/sh 
echo ${name} 
[root@localhost ~]# ls -l test.sh  //验证脚本可执行 
-rwxr-xr-x 1 root root 23 Feb 6 11:09 test.sh 
[root@localhost ~]# ./test.sh    //以下三个命令证明了结论一 
[root@localhost ~]# sh ./test.sh 
[root@localhost ~]# bash ./test.sh 
[root@localhost ~]# . ./test.sh   //以下两个命令证明了结论二 
dangxu 
[root@localhost ~]# source ./test.sh 
dangxu 
[root@localhost ~]# 
```
