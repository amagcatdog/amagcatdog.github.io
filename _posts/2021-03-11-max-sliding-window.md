---
layout: post
title: 滑动窗口最大值
categories: [Algorithm, Leetcode]
description: 滑动窗口最大值
keywords: Algorithm, Leetcode, 单调队列, 最小堆
---

## 问题描述

<https://leetcode-cn.com/problems/sliding-window-maximum>

给你一个整数数组 nums，有一个大小为 k 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 k 个数字。滑动窗口每次只向右移动一位。

返回 滑动窗口中的最大值。

```txt
示例 1：

输入：nums = [1,3,-1,-3,5,3,6,7], k = 3
输出：[3,3,5,5,6,7]
解释：
滑动窗口的位置                最大值
---------------               -----
[1  3  -1] -3  5  3  6  7       3
 1 [3  -1  -3] 5  3  6  7       3
 1  3 [-1  -3  5] 3  6  7       5
 1  3  -1 [-3  5  3] 6  7       5
 1  3  -1  -3 [5  3  6] 7       6
 1  3  -1  -3  5 [3  6  7]      7
示例 2：

输入：nums = [1], k = 1
输出：[1]
```

提示：

```txt
1 <= nums.length <= 105
-104 <= nums[i] <= 104
1 <= k <= nums.length
```

## 问题分析

### 方法一：最小堆

最先想到的是用最小堆实现，窗口移动时，进入窗口的右边元素入堆，最左边元素需要出堆，一般我们不会直接定位元素进行出堆，因为这样会破坏堆的结构，一般采用标记法进行“惰性出堆”。

可以看出元素是否需要出堆与其下标是否在窗口中有关，入堆时同时记录堆内元素在数组中的下标，窗口滑动时，判断堆顶元素下标是否在窗口内，不再窗口内则出堆，直到堆顶元素在窗口内，将堆顶元素加入结果。

本题求最大值，用最小堆的话需要将元素加上负号。

最小堆时间复杂度：T=O(n\*log(n))，空间复杂度：S=O(n)。极端情况下只有入堆没有出堆，比如数组递增。

### 方法二：单调队列

继续分析可以发现，目标是求滑动窗口的最大值，窗口滑动时，只需要保留滑动窗口中较大的值，较小的值可以直接扔掉不管，因此可以用单调队列替代上面的最小堆。

窗口滑动时，窗口右边新元素从队尾入队，从队尾依次出队，直到队尾元素比新元素大，或者队列空，此时将新元素入队尾。

取滑动窗口最大值时，判断队头元素是否在窗口范围内，如果不在则队头出栈，直到队头元素在窗口范围内，将队头元素加入结果。

单调队列时间复杂度：T=O(n)，S=O(k)。最多每个元素入队出队一次，队列长度不超过k。

## 编码实现

### 最小堆

```python
import heapq
class Solution:
    def maxSlidingWindow(self, nums: List[int], k: int) -> List[int]:
        res = []
        q = [(-nums[i], i) for i in range(k-1)]
        heapq.heapify(q)
        for i in range(k-1, len(nums)):
            heapq.heappush(q, (-nums[i], i))
            while q[0][1] <= i-k:
                heapq.heappop(q)
            res.append(-q[0][0])
        return res
```

### 单调队列

```java
import java.util.Deque;
import java.util.LinkedList;
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        n = nums.length;
        int[] res = new int[n-k+1];
        Deque<Integer> qSub = new LinkedList<Integer>();
        for (int i = 0; i < n; ++i) {
            while (!qSub.isEmpty() && nums[i] >= nums[qSub.peekLast()]) { // get last
                qSub.pollLast(); // remove last
            }
            qSub.offerLast(i); // add last

            while (qSub.peekFirst() <= i-k) { // get first
                qSub.pollFirst(); // remove first
            }

            if (i >= k-1) {
                res[i-k+1] = nums[qSub.peekFirst()]; // get first
            }
        }
        return res;
    }
}
```
