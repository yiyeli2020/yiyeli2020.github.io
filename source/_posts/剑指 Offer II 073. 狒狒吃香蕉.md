---
title: 剑指 Offer II 073. 狒狒吃香蕉

date: 2021-12-17 19:30:12  

categories: 2021年12月

tags: [LeetCode, Array, Binary Search]

---

返回她可以在 H 小时内吃掉所有香蕉的最小速度 K（K 为整数）。

<!-- more -->

[TOC]

珂珂喜欢吃香蕉。这里有 N 堆香蕉，第 i 堆中有 piles[i] 根香蕉。警卫已经离开了，将在 H 小时后回来。

珂珂可以决定她吃香蕉的速度 K （单位：根/小时）。每个小时，她将会选择一堆香蕉，从中吃掉 K 根。如果这堆香蕉少于 K 根，她将吃掉这堆的所有香蕉，然后这一小时内不会再吃更多的香蕉。  

珂珂喜欢慢慢吃，但仍然想在警卫回来前吃掉所有的香蕉。

返回她可以在 H 小时内吃掉所有香蕉的最小速度 K（K 为整数）。

注意：本题与[875. 爱吃香蕉的珂珂](https://leetcode-cn.com/problems/nZZqjQ/)相同

示例 1：

    输入: piles = [3,6,7,11], H = 8
    输出: 4
示例 2：

    输入: piles = [30,11,23,4,20], H = 5
    输出: 30
示例 3：

    输入: piles = [30,11,23,4,20], H = 6
    输出: 23


提示：
    
    1 <= piles.length <= 10^4
    piles.length <= H <= 10^9
    1 <= piles[i] <= 10^9

# 二分查找[^1]

**思路**

如果珂珂能以 `K` 的进食速度最终吃完所有的香蕉（在 `H` 小时内），那么她也可以用更快的速度吃完。

当珂珂能以 `K` 的进食速度吃完香蕉时，我们令 `possible(K)` 为 `true`，那么就存在 `X` 使得当 `K >= X` 时， `possible(K) = True`。

举个例子，当初始条件为 `piles = [3, 6, 7, 11]` 和 `H = 8` 时，存在 `X = 4` 使得 `possible(1) = possible(2) = possible(3) = False`，且 `possible(4) = possible(5) = ... = True`。

**算法**

我们可以二分查找 `possible(K)` 的值来找到第一个使得 `possible(X)` 为 `True` 的 `X`：这将是我们的答案。我们的循环中，不变量 `possible(hi)` 总为 `True`， `lo` 总小于等于答案。有关二分查找的更多信息，请参阅[《力扣探索：二分查找》](https://leetcode-cn.com/explore/learn/card/binary-search/)。

为了找到 `possible(K)` 的值， (即`珂珂`是否能以 `K` 的进食速度在 `H` 小时内吃完所有的香蕉），我们模拟这一情景。对于每一堆（大小 `p > 0`），我们可以推断出珂珂将在 `Math.ceil(p / K) = ((p-1) // K) + 1` 小时内吃完这一堆，我们将每一堆的完成时间加在一起并与 `H` 进行比较。


<details>
    <summary>cpp</summary>

```cpp [2q2E5AzB-C++]
class Solution {
public:
    int minEatingSpeed(vector<int>& piles, int H) {
        int lo = 1, hi = pow(10, 9);
        while (lo < hi) {
            int mi = lo + (hi - lo) / 2;
            if (!possible(piles, H, mi))
                lo = mi + 1;
            else
                hi = mi;
        }

        return lo;
    }

    // Can Koko eat all bananas in H hours with eating speed K?
    bool possible(vector<int>& piles, int H, int K) {
        int time = 0;
        for (int p: piles)
            time += (p - 1) / K + 1;
        return time <= H;
    }
};
```
</details>
<details>
    <summary>Java</summary>

```java [2q2E5AzB-Java]
class Solution {
    public int minEatingSpeed(int[] piles, int H) {
        int lo = 1;
        int hi = 1_000_000_000;
        while (lo < hi) {
            int mi = (lo + hi) / 2;
            if (!possible(piles, H, mi))
                lo = mi + 1;
            else
                hi = mi;
        }

        return lo;
    }

    // Can Koko eat all bananas in H hours with eating speed K?
    public boolean possible(int[] piles, int H, int K) {
        int time = 0;
        for (int p: piles)
            time += (p-1) / K + 1;
        return time <= H;
    }
}
```
</details>
<details>
    <summary>Python</summary>

```python [2q2E5AzB-Python]
class Solution(object):
    def minEatingSpeed(self, piles, H):
        # Can Koko eat all bananas in H hours with eating speed K?
        def possible(K):
            return sum((p-1) / K + 1 for p in piles) <= H

        lo, hi = 1, max(piles)
        while lo < hi:
            mi = (lo + hi) / 2
            if not possible(mi):
                lo = mi + 1
            else:
                hi = mi
        return lo
```

</details>

[^1]:https://leetcode-cn.com/problems/koko-eating-bananas/solution/ai-chi-xiang-jiao-de-ke-ke-by-leetcode/