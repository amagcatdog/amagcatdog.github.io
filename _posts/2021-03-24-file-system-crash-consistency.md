---
layout: post
title: 文件系统一致性问题
categories: [OS, 文件系统]
description: 系统断电或奔溃时，文件系统如何保证数据一致性？
keywords: 文件系统, 磁盘, 日志型文件系统
---

## 文件系统结构

一个文件在磁盘上的分配一般包括：文件的 inode 、文件的 data block 以及记录分配状态的 inode bitmap 和 data bitmap。

```txt
inode bitmap | data bitmap | inodes | data blocks
```

## 问题描述

假设为了完成某项写入任务，文件系统需要同时更新磁盘上的数据 A 和数据 B ，因为磁盘是顺序写的，必然造成 A 和 B 其中一个先被写入磁盘，如果此时发生断电或者系统奔溃，磁盘上的数据就会造成不一致的情况。

## 解决方案

### fsck

较老的文件系统采用的方法，文件系统检查程序（File System Checker）是文件系统启动时进行的一致性检查，属于事后修复。只能保证文件系统元数据内部的一致性，不能保证用户数据的一致性。另一大缺点是慢，需要扫描整个磁盘。

### 日志型文件系统

日执型文件系统的设计思路来源于数据库管理系统的 WAL 日志（write ahead logging）和事务（transaction）功能。

更新磁盘前，先往磁盘的其它地方写入一些信息，这些信息主要描述接下来要更新什么，是本次更新的注记，即预写日志（journaling）。这样，系统崩溃后可以从日志中恢复对磁盘的写入操作。

假设现在要向一个文件中追加数据，需要将这个文件的 inode（I[v2]）, bitmap（B[v2]）和 data block （Db）写入磁盘，在将它们写入磁盘前，需要先将它们作为一个事务写入日志：

```txt
TxB | I[V2] | B[V2] | Db | TxE
```

一旦这个事务日志安全写到磁盘，就可以覆写文件系统中的旧结构，这个过程称为加检查点（checkpointing）。

如果检查点成功覆写，则本次磁盘写入完成，可以删除事务日志。如果在覆写数据过程中发生 crash ，系统重启后将从日志中尝试重新执行写入操作。

如果在写入日志期间发生 crash ，可能会出现这种情况：`TxB|I[V2]|B[V2]|TxE`块都被写入了，而用户数据 Db 块并未写入，这种情况文件系统重启后将重放日志事务，由于用户数据可能是任意的数据，文件系统重启后无法知道数据是否有效，将导致垃圾数据写入磁盘。

解决上述问题的一种方法是采用顺序写，一次发出一个块，等待上一个块写完再发出下一个块，最终写入 TxE 终止块，但这太慢了。我们希望的是一次发出所有的五个块，但由于磁盘内部可以执行调度并以任何顺序完成大批写入的小块，即可能导致上面的垃圾数据问题。

为了避免上面问题，日志写入分两次发出，第一次发出除 TxE 块之外的其它块，这些块可以按任意顺序写入，等待这些块都写入后（可以插入一个 write barrier 保证写入磁盘，ext4 支持校验和不需要 Barrier），第二次发出 TxE 块，磁盘可以保证一个扇区（512字节）原子写入，因此 TxE 块设计为一个512字节的块，即可保证原子性写入。这种情况下一个完整的数据写入操作分为三步：

1．日志写入（journal writ）：将事务的内容（包括 TxB 、元数据和数据）写入日志，等待这些写入完成

2．日志提交（journal commit）：将事务提交块（包括 TxE ）写入日志，等待写完成，事务被认为已提交（ committed ）

3．加检查点（checkpointing）：将更新内容（元数据和数据）写入其最终的磁盘位置

另一种更简单高效的解决方法是在开始和结束块中增加日志内容的校验和，这样可以使文件系统立即写入整个事务而不产生等待，重启回放日志时，通过校验和校验日志事务的完整性，不完整的日志将被直接丢弃。

Linux ext2 文件系统没有使用日志，ext3/ext4 文件系统是日志型文件系统，ext4 增加了日志事务数据的校验和。

上面介绍的是日志文件系统基本协议，基本协议对性能有较大影响，改进方法是将多个更新缓冲到全局事务中，通过批处理更新提高效率。

另外还有 Ordered 和 Writeback 两种模式，这两种模式保证文件系统元数据的一致性，但不保证用户数据的一致性，对一致性作出一些妥协，性能较基本协议有所提升。

Ordered 模式是 ext4 默认方式，先进行数据块的真实写操作，再写入事务日志，事务日志只包括 inode 和 bitmap 元数据信息，最后进行元数据的真实写操作，如果 crash 发生在元数据日志写入操作后，重启可以通过重放日志更新元数据信息，不会造成数据不一致或者数据丢失；如果 crash 发生在用户数据块的真实写操作期间，可能导致正在覆写的用户数据块数据错误或造成新附加的用户数据块丢失（附加的文件一般比覆盖的文件更普遍，所以这种影响较小）。

Writeback 模式较 Ordered 模有更好的性能，但一致性更弱些。该模式和 Ordered 类似，事务日志只包含元数据信息，同时进一步放宽限制，不要求数据块真实写操作在事务日志前发生，因此用户数据的真实写操作可能发生在日志事务和写检查点前后的任何时刻。如果 crash 发生在日志写操作之后，元数据的真实写操作之前（假设也在用户数据的写操作之前），那么进行文件系统的重放时， 元数据的写操作会被重放，但是用户数据的写操作不会，这将有可能造成同一文件的元数据和用户数据的不一致。

## 参考

![日志型文件系统 - 原理和优化](https://zhuanlan.zhihu.com/p/107558961)
![系统可能在任何两次写入之间崩溃或断电，崩溃后，如何更新磁盘？](https://blog.csdn.net/epubit17/article/details/99277412)