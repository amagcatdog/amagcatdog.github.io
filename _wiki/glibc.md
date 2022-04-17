---
layout: wiki
title: glibc
cate1: c++
cate2: 内存
description: glibc 内存相关问题
keywords: c++, glibc, 内存
type:
link:
---

今天遇到一个业务部门的程序core在平台接口中，查看 coredump 文件发现 glibc 的内存池数据可能损坏，初步定为发生了内存越界访问，让业务同事注释掉一些可疑代码，发现程序可以正常运行到平台接口，证明注释的代码中存在内存越界的可能。借这次问题排查将 glibc 内存相关问题做一个总结，便于日后查阅。

## 内存越界

内存越界是指程序使用了不该使用的内存，带来的后果是不确定的。

- 如果破坏了glibc在堆中的内存分配信息数据，core文件可能会出现如下的错误：

    ```txt
    free(): invalid pointer
    malloc(): memory corruption
    double free or corruption (out)
    corrupted double-linked list
    malloc_consolidate(av=av@entry=xxx <main_arena>) at malloc.c
    ```

- 如果破坏了程序申请的其它内存数据，那么可能造成程序执行的不确定性，进而引发coredump。

- 如果破坏的是空闲内存块，那么很幸运不会造成问题。

一般内存越界时程序不会立即发生问题，当访问到被越界写入的内存数据时，才可能造成程序异常，这给问题定位带来了很大难度。

针对内存越界，glibc 有一个环境变量可以检测 malloc 和 free 相关问题：

```txt
MALLOC_CHECK_=0 关闭所有检查
MALLOC_CHECK_=1 当有错误被探测到时，在标准错误输出（stderr）上打印错误信息
MALLOC_CHECK_=2 当有错误被探测到时，不显示错误信息，直接调用 abort 进行中断
MALLOC_CHECK_=3 当有错误被探测到时，同时执行 1 和 2
```

当程序出现内存越界时，可以通过设置上述环境变量，让 glibc 及时检测出内存问题。

## 内存池参数

linux通过brk、mmap/munmap系统调用来分配内存，频繁的系统调用对于系统性能有很大的损耗，glibc为了减少内存分配时的系统调用，实现了一个内存池功能，当内存池中有空闲内存时，优先返回内存池中的内存，否则通过系统调用向系统申请内存。glibc相当于一个内存分配代理，提供了malloc/free函数给用户调用。

glibc的内存池是以 arena 内存分配区为单位进行管理的，每个进程都有一个 main_arena 内存主分配区和若干非主分配区 non_main_arena ，主分配区和非主分配区用环形链表进行管理，每个分配区采用互斥锁实现多线程互斥访问。如果线程申请内存时所有分配区都已经被加锁，那么glibc将会向系统申请内存生成一个新的非主分配区。

64位系统每个内存分配区 64MB 大小，默认情况下每个线程会预分配一个私有的 arena ，这样可以减少多线程内存分配时的锁竞争，但也会带来一个问题：随着线程数增加，非主分配区越来越多，占用的虚拟内存越来越大，如果系统通过 limit 设置了虚拟内存限制，达到预设限制后，进程将无法创建新的线程。

上面的问题通过`top -p [pid]`命令可以看到进程的虚拟内存随着线程增加而变大，`pmap -p [pid]`命令可以看到大量 64MB 的 anon 内存分配块。这个问题可以通过设置 `MALLOC_ARENA_MAX` 环境变量来限制 arena 的最大数量。一般可以设置成4个，太小影响性能，太大影响资源占用，需要在二者之间进行 trade off 。

## 参考

- [glibc内存管理——Linux内存管理小结二](https://www.jianshu.com/p/b0a6ac5bf55d)