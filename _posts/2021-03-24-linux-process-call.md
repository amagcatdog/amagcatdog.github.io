---
layout: post
title: linux 进程调用相关函数区别
categories: [OS, process]
description: linux popen/system/fork/exec等系统调用的区别
keywords: OS, process, popen, system, fork, exec
---

## exec 替换进程映像

exec 是一个函数族，用于执行不同的命令参数，除此之外函数族内函数的功能都相同。当一个进程调用了 exec 函数后，它本身就“死亡”了，系统把代码段、数据段替换成 exec 参数指定的程序，并分配新的堆和栈空间，唯一留下的是进程的 ID 等信息，对系统来说还是同一个进程，但实际已经是另一个程序了。

exec 如果成功替换成了新的程序映像运行，将不会有任何返回值，如果执行失败，将会返回 -1 。

## fork 复制进程映像

调用 fork 函数后，进程将会发生分叉，如果 fork 返回值是 -1 ，表示系统调用失败，错误码在 errno 中，如果返回值是 0 ，表示处于子进程，如果返回值是大于 0 的整数，表示处于父进程，且返回值是子进程的 PID 。

子进程继承了父进程的进程空间（代码段、数据段、堆、栈、文件描述符等），注意父子进程之间数据都是单独的一份，不再共享。现代操作系统大多采用写时复制（COW）机制，初始时子进程开辟了一块虚拟内存空间，其指向的物理内存和父进程的物理内存相同，同时物理内存被设置为 readonly ，当父子进程一方修改物理内存数据时，触发写时复制，修改内存数据的一方将会复制要修改的物理内存页，进行修改。

如果需要在父进程运行过程中启动一个子进程运行另一个程序，一般先调用 fork 函数，在子进程代码分支中立即调用 exec 函数，将子进程替换为 exec 参数的指定的程序映像，这种情况一般操作系统可以优先调度子进程，这样可以避免不必要的写时复制。

## system 启动新进程，带返回值

system 系统调用用于在进程运行过程中调用另一个程序，主进程将会阻塞等待子进程完成，并返回子进程的执行结果。其内部使用了 fork + exec 的机制。

需要注意的是，这种方式是通过启动一个 shell 并调用子程序实现的，即 exec 执行的命令相当于`/bin/sh –c cmd`，因此 system 函数并非启动其他进程的理想手段，因为对 shell 的依赖较大。

system 方式启动的子进程拥有与父进程完全不同的代码段、数据段、堆和栈。

另外 system 的返回值有如下几种可能：

- 父进程检查 cmd 字符串为空，返回非零值，一般为 1

- 父进程 fork 失败或者 waitpid 被信号中断，返回 -1，且 errno 中设置了错误类型值

- 子进程执行 exec 失败，返回 127 ，且返回值放在高字节，即实际返回为 0x7f00 >> 8 = 127

- 其它情况返回子进程执行的返回值，且返回值放在高字节，即实际返回 result >> 8

下面的程序调用system函数，并对返回值进行分析：

```cpp
#include <stdio.h>
#include <sys/wait.h>
#include <stdlib.h>
 
void pr_exit(int status){
        printf("status = %d\n", status);
        if(WIFEXITED(status)){
                printf("normal terminaton, exit status = %d\n", WEXITSTATUS(status));
        }else if(WIFSIGNALED(status)){
                printf("abnormal termination, signal number = %d%s\n",
                WTERMSIG(status),
#ifdef WCOREDUMP
                WCOREDUMP(status)?"(core file generated)" : "");
#else
                "");
#endif
        }else if(WIFSTOPPED(status)){
                printf("child stopped, signal number = %d\n", WSTOPSIG(status));
        }
}
 
int main(int argc, char* argv[]){
        if(argc != 2){
                printf("usage:./a.out [cmdstring]\n");
                return -1;
        }
        int status;
        status = system(argv[1]);
        pr_exit(status);
        return 0;

}
```

```txt
执行上面的程序：

1. shell执行失败
yan@yan-vm:~/apue$ ./a.out nosuchcmd
sh: 1: nosuchcmd: not found
status = 32512
normal terminaton, exit status = 127
nosuchcmd不是shell支持的命令，所以，shell命令返回了127（exec失败），对于system函数，返回值为127*256 = 32512；
因为shell的返回值是 system返回值的8～15位（所以在程序中返回超过255的错误代码是无意义的）。

2. 命令成功执行，但命令返回错误码
yan@yan-vm:~/apue$ ./a.out "ls /a/b/c"
ls: cannot access /a/b/c: No such file or directory
status = 512
normal terminaton, exit status = 2
虽然没有这个/a/b/c目录，但是命令是成功执行了，所以没有返回127，而是返回了ls /a/b/c命令的错误代码2（2*256 = 512）。

3. 命令成功执行，且返回成功码
yan@yan-vm:~/apue$ ./a.out "date"
Sat Jul 27 20:48:22 CST 2013
status = 0
normal terminaton, exit status = 0
system成功执行了date，date程序退出码为0。
```

## popen 启动新进程，带输出

popen 函数采用管道进行父子进程间通信，并把子进程的输出重定向到管道中，供父进程读取，函数返回的是一个文件指针，通过文件指针可以读取到子进程的输出内容。

子进程的调用方式也是 fork + exec ，exec 执行的命令也是通过调用 shell 执行。

## 总结

父进程调用子进程的几种方式比较：

- 如果需要获取子进程执行的返回值，用 system
- 如果不需要获取子进程执行的返回值，用 fork + exec 更高效
- 如果需要获取子进程执行的输出，用 popen

## 参考

![system()、exec()、fork()三个与进程有关的函数的比较](https://www.cnblogs.com/qingergege/p/6601807.html)
![linux下execl和system函数](https://www.cnblogs.com/Cccarl/p/6639089.html)
![【进程管理】fork之后子进程到底复制了父进程什么？](https://zhuanlan.zhihu.com/p/370705498)
![popen system fork exec等函数的区别](http://cppblog.com/prayer/archive/2009/09/28/97456.html)