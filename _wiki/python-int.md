---
layout: wiki
title: python int
cate1: python
cate2:
description: python int
keywords: python
type:
link:
---

python 中 int 对象是 built-in 对象，可以表示任意长度的整型值，其底层使用变长对象存储整型值，通过移位、拼接等操作合成整型值。

## 负数

对于负数，内存中存储的是**补码**，注意不同函数的输出：

```python
a=-1
bin(a) # '-0b1'
hex(a) # '-0x1'

a.to_bytes(4, byteorder='big', signed=True) # b'\xff\xff\xff\xff'

a & 0xffffffff # 4294967295

hex(a & 0xffffffff) # '0xffffffff'
```

补码的最高位表示符号位，因此当使用 python 按位处理整型时，最高位需要特殊处理（要显式地让变量为负值，否则 python 不知道这个值是负值补码还是正值），比如按位将 `a=-1` 拷贝到变量 `b`:

```python
a = -1
b = 0
for i in range(32):
    if (a >> i) & 1:
        if i == 31:
            # 补码最高位表示符号位，如果是1需要特殊处理
            # b 之前为 0xefffffff
            # 减去 0x8fffffff 之后为 -1
            # 则 b 最终为 -1
            b -= (1 << i)
        else:
            b |= (1 << i)
print(b) # -1

```

## 小对象共享

[-5, 256] 之间的小整型值采用预分配对象共享的方式节约内存和减少分配：

```python
a, b = -5, -5
a is b # True

a, b = 256, 256
a is b # True
```
