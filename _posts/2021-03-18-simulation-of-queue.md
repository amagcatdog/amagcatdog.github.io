---
layout: post
title: 模拟队列
categories: [Algorithm, Leetcode]
description: 模拟队列
keywords: Algorithm, Leetcode, 队列, 数组, 栈
---

## 问题描述

1. 用一个定长数组模拟 FIFO 队列的 push 和 pop

2. 用两个 FILO 栈模拟 FIFO 队列的 push 和 pop

## 问题分析

1. 用下标变量 cur 记录当前的 push 位置，用 size 变量记录当前元素个数。push 操作先判断 size 如果等于数组容量，抛出异常，否则增加 cur 下标，如果 cur 下标到达数组末尾，则返回下标 0 继续。pop 操作先判断 size 是否为0，如果为0则抛出异常，否则通过 cur 和 size 计算头部位置下标，返回头部位置下标元素，并更新 size。

2. 栈先进后出，利用两次栈操作，模拟先进先出队列。一个栈 S1 作为入队操作，另一个栈 S2 用于出队操作，如果出队时 S2 空，则检查 S1 是否有元素，如果 S1 非空，则将 S1 所有元素出栈再入栈到 S2 供出队。

## 编码实现

```python
class ArrayQueue:
    '''
    固定数组实现的FIFO队列
    '''
    def __init__(self, capacity):
        self.capacity = capacity
        self.queue = [None] * self.capacity
        self.count = 0
        self.tailPos = 0
    
    def push(self, val):
        if self.count == self.capacity:
            raise ValueError('full queue')
        if self.tailPos == self.capacity:
            self.tailPos = 0
        self.queue[self.tailPos] = val
        self.count += 1
        self.tailPos += 1
    
    def pop(self):
        if self.count == 0:
            raise ValueError('empty queue')
        headPos = self.tailPos - self.count
        # 也可以保持 headPos 为负数，表示取倒数第几个
        if headPos < 0:
            headPos = self.capacity + headPos
        self.count -= 1
        return self.queue[headPos]
```

```python
class StackQueue:
    '''
    两个栈模拟FIFO队列
    '''
    def __init__(self):
        self.stackPush = []
        self.stackPop = []

    def push(self, x: int) -> None:
        self.stackPush.append(x)

    def pop(self) -> int:
        if not self.stackPop:
            while self.stackPush:
                self.stackPop.append(self.stackPush.pop())
        if self.stackPop:
            return self.stackPop.pop()
        return None

    def peek(self) -> int:
        if not self.stackPop:
            while self.stackPush:
                self.stackPop.append(self.stackPush.pop())
        if self.stackPop:
            return self.stackPop[-1]
        return None

    def empty(self) -> bool:
        return not (self.stackPush or self.stackPop)
```
