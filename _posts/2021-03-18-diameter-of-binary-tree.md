---
layout: post
title: 二叉树的直径
categories: [Algorithm, Leetcode]
description: 二叉树的直径
keywords: Algorithm, Leetcode, 二叉树
---

## 问题描述

leetcode 543

<https://leetcode-cn.com/problems/diameter-of-binary-tree>

给定一棵二叉树，你需要计算它的直径长度。一棵二叉树的直径长度是任意两个结点路径长度中的最大值。这条路径可能穿过也可能不穿过根结点。

示例 :

```txt
给定二叉树

          1
         / \
        2   3
       / \     
      4   5    
返回 3, 它的长度是路径 [4,2,1,3] 或者 [5,2,1,3]。

注意：两结点之间的路径长度是以它们之间边的数目表示。
```

## 问题分析

二叉树问题一般将二叉树看做根节点、左子树、右子树三个部分，进行递归求解。关键是要能够把问题分解到子树上，本题求叶子节点间的最远距离，从根节点、左子树、右子树角度看，即求左右子树的高度和，同时左右子树自身内部可能存在更大的最远距离，左右子树内部的求解只需要分别递归左右子树即可。

## 编码实现

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def diameterOfBinaryTree(self, root: TreeNode) -> int:
        def getHeight(root):
            '''
            计算树的高度
            '''
            if not root: return 0
            return 1+max(getHeight(root.left), getHeight(root.right))
        if not root: return 0
        v1 = v2 = h1 = h2 = 0
        if root.left:
            v1 = self.diameterOfBinaryTree(root.left)
            h1 = getHeight(root.left)
        if root.right:
            v2 = self.diameterOfBinaryTree(root.right)
            h2 = getHeight(root.right)
        return max(v1, v2, h1+h2)

    def diameterOfBinaryTree(self, root: TreeNode) -> int:
        def helper(root):
            '''
            return: root树的直径，root树高度
            '''
            if not root: return 0,0
            d1, h1 = helper(root.left)
            d2, h2 = helper(root.right)
            return max(d1, d2, h1+h2), 1+max(h1, h2)
        return helper(root)[0]
```
