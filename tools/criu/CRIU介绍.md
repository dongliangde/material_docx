# CRIU介绍

## 1、CRIU简介

  CRIU（Checkpoint/Restore In Userspace）运行在linux操作系统上的一个软件工具，其功能是在用户空间实现Checkpoint/Restore功能。使用这个工具，你可以冻结一个正在运行的程序，并且checkpoint它到一系列的文件，然后你就可以使用这些文件在任何主机重新恢复这个程序到被冻结的那个点(白话就是实现对已运行程序的备份和恢复)。所以criu通常被用在程序或者容器的热迁移、快照、远程调试等。CRIU 起初是Virtuozzo的一个项目，随着开源社区的帮助，现在也被整合到OpenVZ（它是 Virtuozzo 的开源版本）, LXC/LXD, Docker, Podman等软件项目里。

`Licence: GPLv2`
`官 网：http://criu.org`
`源码地址：https://github.com/checkpoint-restore/criu`

## 2.CRIU工作原理

 ![img](https://img2018.cnblogs.com/blog/1328430/201809/1328430-20180905115628923-1702357844.png) 

## 3.功能命令以及选项参数含义

　　在上一篇我们已经了解到了**dump** ， **restore** 等基础功能命令，这里我们再重新认识一下它们，知道它们新的含义，同时还有一些新的有意思的命令

　　在上一篇我们已经了解到了**dump** ， **restore** 等基础功能命令，这里我们再重新认识一下它们，知道它们新的含义，同时还有一些新的有意思的命令

```
dump                  功能   没有别的参数的情况下，转储所有进程信息并杀死进程
pre-dump              功能   仅仅转储内存文件，打开内存更改跟踪，并保留程序运行
--track-mem           选项   打开内存更改跟踪
--prev-images-dir     选项   指明上次检查点的文件路径
--leave-running       选项   让进程继续存活
```

##  4.命令和选项的搭配及其含义

- dump 转储所有并杀死进程
- dump --leave-running 转储所有并保留被转储进程继续执行
- dump --track-mem 转储所有杀死进程并打开内存更改追踪（这个通常被认为是没有用处的，因为dump会杀死进程，详细请了解下一篇）
- dump --track-mem --leave-running 转储所有，打开内存更改追踪，同时保留程序继续执行（这个应该是比较有用的）
- dump --track-mem --leave-running --prev-images-dir $path 同上，同时在转储文件的时候检查$path中的文件，跳过已经存在的部分（内存页没有更改的部分）
- pre-dump 仅转储内存，打开内存更改追踪，保留程序继续执行
- pre-dump --prev-images-dir $path 同上但是会检查$path中的文件，跳过已经存在的部分

##  5.测试程序

```
vim test.c
```

编写一个用于测试的C程序： 

```
#include <stdio.h>
#include <unistd.h>

int main(int argc, char const *argv[]) {
    int i = 0;
    while(true) {
        sleep(1);
        printf("%d\n", i);
        i++;
    }

    return 0;
}
```

编译生成可执行文件：

```
gcc test.c -o test
```

## 6.运行测试程序

1）执行测试程序

```
./test

0
1
2
3
killed
```


2）另一个终端查看测试程序pid：

```
ps -ef | grep test

root   1597   1691  0 21:45 pts/2    00:00:00 ./test
```


3）dump：设置检查点并中断程序

```
mkdir img
criu dump -t 7896 -D img/ -o dump.log -v4 --tcp-established -j
```

4）恢复程序：

```
criu restore -D img/ -o restore.log -d --tcp-close -j -v4
```

5）检查进程是否使用原来的进程号继续运行：

```
ps -ef | grep test
```

命令说明：

dump命令：由实验中可以看到，当设置进程1597的checkpoint后，再查找该进程，发现进程不存在，即进程已经退出。查看快照文件目录，生成很多img文件，这些文件主要用于恢复应用。
-D（等价于–images-dir）：选项指定应用的快照文件保存目录
-j（等价于–shell-job）：表示该应用是一个通过shell启动的作业
-t：指定需要checkpoint的应用pid
当对应用设置checkpoint后，应用会自动退出，如果希望应用继续执行，需指定-R或–leave-running选项（见实验2）。

restore命令：由实验中可以看到，恢复后的程序从设置checkpoint的时间点继续运行，程序在输出3时被kill掉，恢复后继续输出4，恢复后查找进程1597，发现进程使用原来的进程号继续运行。