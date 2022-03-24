---
layout: post
title: 使括号有效的最少添加
categories: [Algorithm, Leetcode]
description: 使括号有效的最少添加
keywords: Algorithm, Leetcode, 括号问题
---

## 问题描述

leetcode 921

<https://leetcode-cn.com/problems/minimum-add-to-make-parentheses-valid>

只有满足下面几点之一，括号字符串才是有效的：

它是一个空字符串，或者
它可以被写成 AB （A 与 B 连接）, 其中 A 和 B 都是有效字符串，或者
它可以被写作 (A)，其中 A 是有效字符串。
给定一个括号字符串 s ，移动N次，你就可以在字符串的任何位置插入一个括号。

例如，如果 s = "()))" ，你可以插入一个开始括号为 "(()))" 或结束括号为 "())))" 。
返回 为使结果字符串 s 有效而必须添加的最少括号数。

```txt
示例 1：

输入：s = "())"
输出：1
示例 2：

输入：s = "((("
输出：3
```

```txt
提示：

1 <= s.length <= 1000
s 只包含 '(' 和 ')' 字符。
```

## 问题分析

遍历每一个字符，统计遍历过程中左括号和右括号个数，如果右括号比左括号多，则必须增加左括号，此时更新结果，并且更新左括号个数等于右括号个数。

遍历结束后，左括号可能比右括号多，需要增加右括号个数。

## 编码实现

```python
class Solution:
    def minAddToMakeValid(self, s: str) -> int:
        left = right = res = 0
        for ch in s:
            if ch == '(':
                left += 1
            else:
                right += 1
            if left < right:
                res += right - left
                left = right
        res += (left - right)
        return res
```
