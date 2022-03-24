---
layout: post
title: 移除无效的括号
categories: [Algorithm, Leetcode]
description: 移除无效的括号
keywords: Algorithm, Leetcode, 括号问题
---

## 问题描述

leetcode 1249

<https://leetcode-cn.com/problems/minimum-remove-to-make-valid-parentheses/>

给你一个由 '('、')' 和小写字母组成的字符串 s。

你需要从字符串中删除最少数目的 '(' 或者 ')' （可以删除任意位置的括号)，使得剩下的「括号字符串」有效。

请返回任意一个合法字符串。

有效「括号字符串」应当符合以下 任意一条 要求：

空字符串或只包含小写字母的字符串
可以被写作 AB（A 连接 B）的字符串，其中 A 和 B 都是有效「括号字符串」
可以被写作 (A) 的字符串，其中 A 是一个有效的「括号字符串」

```txt
示例 1：

输入：s = "lee(t(c)o)de)"
输出："lee(t(c)o)de"
解释："lee(t(co)de)" , "lee(t(c)ode)" 也是一个可行答案。
示例 2：

输入：s = "a)b(c)d"
输出："ab(c)d"
示例 3：

输入：s = "))(("
输出：""
解释：空字符串也是有效的

提示：

1 <= s.length <= 105
s[i] 可能是 '('、')' 或英文小写字母
```

## 问题分析

遍历字符串 s，有效的括号需满足两个条件：

1. 从 s 开头累计的左括号数量必须大于等于右括号数量，如果右括号数大于左括号数，那么字符串一定无效
2. 按1遍历完后，可能出现左括号比右括号多的情况，需要移除多余的左括号

括号问题一般可以通过栈进行匹配括号的抵消，最后栈中的就是不匹配的括号。

或者记录遍历过程中左右括号的数量，按照上边条件1处理右括号，对条件1处理后的字符串反向遍历，处理条件2。
实际上只需要处理条件1时记录左括号不匹配的数量，反向遍历去掉对应数量的多余左括号即可。

## 编码实现

```python
class Solution:
    def minRemoveToMakeValid(self, s: str) -> str:
        stack = []
        for i, ch in enumerate(s):
            if ch == '(':
                stack.append(i)
            elif ch == ')':
                if stack and s[stack[-1]] == '(':
                    stack.pop()
                else:
                    stack.append()
        res = []
        stack = set(stack)
        for i, ch in enumerate(s):
            if i in stack:
                stack.remove(i)
            else:
                res.append(ch) 
        return ''.join(res)
                
    def minRemoveToMakeValid(self, s: str) -> str:
        def removeRight(lst, left, right):
            res = []
            lcnt = rcnt = 0
            for ch in lst:
                if ch not in '()':
                    res.append(ch)
                    continue
                if ch == left:
                    lcnt += 1
                elif ch == right:
                    rcnt += 1
                if lcnt >= rcnt:
                    res.append(ch)
                else
                    rcnt -= 1
            return res
        res1 = removeRight(s, '(', ')')[::-1]
        res2 = removeRight(res1, ')', '(')[::-1]
        return ''.join(res2)

    def minRemoveToMakeValid(self, s: str) -> str:
        res = []
        lcnt = rcnt = 0
        for ch in s:
            if ch not in '()':
                res.append(ch)
                continue
            if ch == '(':
                lcnt += 1
            elif ch == ')':
                rcnt += 1
            if lcnt >= rcnt:
                res.append(ch)
            else
                rcnt -= 1
        res2 = []
        lkeep = rcnt
        for ch in res:
            if ch == '(' and lkeep == 0:
                continue
            res2.append(ch)
            if ch == '(':
                lkeep -= 1
        return ''.join(res2)

```
