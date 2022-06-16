---
layout: post
title: rocksdb skiplist
categories: [rocksdb, skiplist]
description: rocksdb skiplist
keywords: rocksdb, skiplist
---

## skiplist

branching_factor 表示相邻两层之间元素个数的比值，例如 branching_factor = 4 表示有 25% 的元素将会被选中作为上一层元素，实际代码使用了随机数，带有随机性。

每个 Skiplist::Node 对象包含两个内部对象：

```c++
struct SkipList::Node {
public:
  Key const key;
private:
  std::atomic<Node*> next_[1];
}
```

Node 结构体最后是长度为1的 Node 指针数组 `Node* next_[1]` ，这是常用的结构体后接连续内存的管理方法，Node 分配内存时，按 `sizeof(Node) + (kMaxHeight_-1) * sizeof(std::atomic<Node*>)` 分配连续空间，连续空间的开始位置存储该 Node 节点，后面依次存储到 `1~kMaxHeight_` 层的指针，`next_[layerN]` 即为该节点指向第 layerN 层节点的指针，`next_[layerN]->next_[layerN]` 即为在 layerN 层链表的下一个节点。

[skiplist](/images/posts/rocksdb/01-skiplist.drawio)

## inline skiplist

InlineSkipList 相较 SkipList 内存更节约，且具有更好的缓存局部性 (cache locality), InlineSkipList::Node 对象只包含一个 Node 指针：

```c++
struct InlineSkipList::Node {
private:
  std::atomic<Node*> next_[1];
}
```

InlineSkipList::Node 分配内存时，按 `sizeof(std::atomic<Node*>) * (kMaxHeight_ - 1) + sizeof(Node) + key_size` 分配连续空间，该 Node 节点存储在偏移量为 `sizeof(std::atomic<Node*>) * (kMaxHeight_ - 1)` 的位置， Node 节点前面存储到 `1~kMaxHeight_` 层的指针，Node 节点后面存储 key 。`&next_[0] - layerN` 为该节点指向第 layerN 层节点的指针，`(&next_[0] - layerN)->next_[0] - layerN` 即为在 layerN 层链表的下一个节点。`next_[0]` 后面存储 key，即 `&next_[1]` 为 key 指针。

[inline-skiplist](/images/posts/rocksdb/02-inline-skiplist.drawio)

## 参考

- [Skip List--跳表（全网最详细的跳表文章没有之一）](https://www.jianshu.com/p/9d8296562806)
- [让链表跳起来--SkipList的原理及实现](http://www.coolsite.top/archives/189)
