---
layout: post
title: 删除无效的括号
categories: [Algorithm, Leetcode]
description: 删除无效的括号
keywords: Algorithm, Leetcode, 括号问题
---
## 问题描述

leetcode 301 hard

<https://leetcode-cn.com/problems/remove-invalid-parentheses/>

给你一个由若干括号和字母组成的字符串 s ，删除最小数量的无效括号，使得输入的字符串有效。

返回所有可能的结果。答案可以按 任意顺序 返回。

```txt
示例 1：

输入：s = "()())()"
输出：["(())()","()()()"]
示例 2：

输入：s = "(a)())()"
输出：["(a())()","(a)()()"]
示例 3：

输入：s = ")("
输出：[""]
```

提示：

```txt
1 <= s.length <= 25
s 由小写英文字母以及括号 '(' 和 ')' 组成
s 中至多含 20 个括号
```

## 问题分析

要求出所有可能的结果，考虑使用深搜遍历所有可能的解，由于要求删除最小数量的括号，那可以优先考虑使用广度优先搜索，每一轮删除一个括号，直到出现合法字符串，这一轮的所有合法字符串即是所求解。

深搜的话可以采用回溯法，需要记住字符串搜索的起始下标和待删除的左括号和右括号数目。

## 编码实现

```python
class Solution:
    def removeInvalidParentheses(self, s: str) -> List[str]:
        '''
        bfs
        '''
        def valid(s):
            bal = 0
            for ch in s:
                if ch == '(':
                    bal += 1
                elif ch == ')':
                    if bal > 0:
                        bal -= 1
                    else:
                        return False
            return bal == 0
        res = set()
        curSet = set([s])
        while curSet:
            # 先判断当前轮次是否有有效字符串，如果有则
            # 是最短删除有效字符串，返回结果
            for s in curSet:
                if valid(s):
                    res.add(s)
            if res: break
            # 对每个字符串，依次移除一个括号，加入广搜集合
            tmp = set()
            for s in curSet:
                for i, ch in enumerate(s):
                    # 这里可以进行剪枝
                    if i > 0 and ch == s[i-1]:
                        continue
                    if ch in '()':
                        tmp.add(s[:i] + s[i+1:])
            curSet = tmp
        return list(res)

    def removeInvalidParentheses(self, s: str) -> List[str]:
        '''
        dfs
        '''
class Solution:
    def removeInvalidParentheses(self, s: str) -> List[str]:
        res = []
        lremove, rremove = 0, 0
        for c in s:
            if c == '(':
                lremove += 1
            elif c == ')':
                if lremove == 0:
                    rremove += 1
                else:
                    lremove -= 1

        def isValid(str):
            cnt = 0
            for c in str:
                if c == '(':
                    cnt += 1
                elif c == ')':
                    cnt -= 1
                    if cnt < 0:
                        return False
            return cnt == 0

        def helper(s, start, lremove, rremove):
            if lremove == 0 and rremove == 0:
                if isValid(s):
                    res.append(s)
                return

            for  i in range(start, len(s)):
                if i > start and s[i] == s[i - 1]:
                    continue
                # 如果剩余的字符无法满足去掉的数量要求，直接返回
                if lremove + rremove > len(s) - i:
                    break
                # 尝试去掉一个左括号
                if lremove > 0 and s[i] == '(':
                    helper(s[:i] + s[i + 1:], i, lremove - 1, rremove);
                # 尝试去掉一个右括号
                if rremove > 0 and s[i] == ')':
                    helper(s[:i] + s[i + 1:], i, lremove, rremove - 1);
                # 统计当前字符串中已有的括号数量

        helper(s, 0, lremove, rremove)
        return res


```