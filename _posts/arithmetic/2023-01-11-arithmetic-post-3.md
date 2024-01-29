---
layout:       post
title: '小而美的算法技巧：前缀和数组'
date: 2023-01-11
header-style: text
catalog:      true
mathjax:      true
tags:
 - 数组
 - 前缀和数组
---

读完本文，你不仅学会了算法套路，还可以顺便解决如下题目：

原文：[小而美的算法技巧：前缀和数组 labuladong 的算法笔记 (gitee.io)](https://labuladong.gitee.io/algo/di-yi-zhan-da78c/shou-ba-sh-48c1d/xiao-er-me-03265/)

视频：[【labuladong】前缀和/差分数组技巧精讲_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1NY4y1J7xQ/?spm_id_from=333.788&vd_source=305afb3d9183503c497178f9d3f3284c)

|                             力扣                             | 难度 |      |
| :----------------------------------------------------------: | :--: | ---- |
| [303. 区域和检索 - 数组不可变](https://leetcode.cn/problems/range-sum-query-immutable/) |  🟢   | 完成 |
| [304. 二维区域和检索 - 矩阵不可变](https://leetcode.cn/problems/range-sum-query-2d-immutable/) |  🟠   | 完成 |
| [剑指 Offer II 013. 二维子矩阵的和](https://leetcode.cn/problems/O4NDxx/) |  🟠   |      |

一、一维数组中的前缀和
======

 303 题「[区域和检索 - 数组不可变](https://leetcode.cn/problems/range-sum-query-immutable/)」

```java
class NumArray {

    public NumArray(int[] nums) {}
    
    /* 查询闭区间 [left, right] 的累加和 */
    public int sumRange(int left, int right) {}
}
```

`sumRange` 函数需要计算并返回一个索引区间之内的元素和，没学过前缀和的人可能写出如下代码：

```java
class NumArray {

    private int[] nums;

    public NumArray(int[] nums) {
        this.nums = nums;
    }
    
    public int sumRange(int left, int right) {
        int res = 0;
        for (int i = left; i <= right; i++) {
            res += nums[i];
        }
        return res;
    }
}
```

这样，可以达到效果，但是效率很差，因为 `sumRange` 方法会被频繁调用，而它的时间复杂度是 `O(N)`，其中 `N` 代表 `nums` 数组的长度。

这道题的最优解法是使用前缀和技巧，将 `sumRange` 函数的时间复杂度降为 `O(1)`，说白了就是不要在 `sumRange` 里面用 for 循环，咋整？

直接看代码实现：

```java
class NumArray {
    // 前缀和数组
    private int[] preSum;

    /* 输入一个数组，构造前缀和 */
    public NumArray(int[] nums) {
        // preSum[0] = 0，便于计算累加和
        preSum = new int[nums.length + 1];
        // 计算 nums 的累加和
        for (int i = 1; i < preSum.length; i++) {
            preSum[i] = preSum[i - 1] + nums[i - 1];
        }
    }
    
    /* 查询闭区间 [left, right] 的累加和 */
    public int sumRange(int left, int right) {
        return preSum[right + 1] - preSum[left];
    }
}
```

<img src="https://s2.loli.net/2024/01/23/VaA1H79hb6uLGpi.jpg" alt="img" style="zoom:33%;" />

```java
int[] scores; // 存储着所有同学的分数
// 试卷满分 100 分
int[] count = new int[100 + 1]
// 记录每个分数有几个同学
for (int score : scores)
    count[score]++
// 构造前缀和
for (int i = 1; i < count.length; i++)
    count[i] = count[i] + count[i-1];

// 利用 count 这个前缀和数组进行分数段查询
```

二维矩阵中的前缀和
======

力扣第 304 题「[二维区域和检索 - 矩阵不可变](https://leetcode.cn/problems/range-sum-query-2d-immutable/)」

<img src="https://s2.loli.net/2024/01/23/uizLAaNtwsCMPdH.jpg" alt="img" style="zoom:33%;" />

```java
class NumMatrix {
    // 定义：preSum[i][j] 记录 matrix 中子矩阵 [0, 0, i-1, j-1] 的元素和
    private int[][] preSum;
    
    public NumMatrix(int[][] matrix) {
        int m = matrix.length, n = matrix[0].length;
        if (m == 0 || n == 0) return;
        // 构造前缀和矩阵
        preSum = new int[m + 1][n + 1];
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                // 计算每个矩阵 [0, 0, i, j] 的元素和
                preSum[i][j] = preSum[i-1][j] + preSum[i][j-1] + matrix[i - 1][j - 1] - preSum[i-1][j-1];
            }
        }
    }
    
    // 计算子矩阵 [x1, y1, x2, y2] 的元素和
    public int sumRegion(int x1, int y1, int x2, int y2) {
        // 目标矩阵之和由四个相邻矩阵运算获得
        return preSum[x2+1][y2+1] - preSum[x1][y2+1] - preSum[x2+1][y1] + preSum[x1][y1];
    }
}
```

|                             力扣                             |      |
| :----------------------------------------------------------: | ---- |
| [1314. 矩阵区域和](https://leetcode.cn/problems/matrix-block-sum/?show=1) |      |
| [1352. 最后 K 个数的乘积](https://leetcode.cn/problems/product-of-the-last-k-numbers/?show=1) |      |
| [238. 除自身以外数组的乘积](https://leetcode.cn/problems/product-of-array-except-self/?show=1) |      |
| [325. 和等于 k 的最长子数组长度](https://leetcode.cn/problems/maximum-size-subarray-sum-equals-k/?show=1)🔒 |      |
| [327. 区间和的个数](https://leetcode.cn/problems/count-of-range-sum/?show=1) |      |
| [437. 路径总和 III](https://leetcode.cn/problems/path-sum-iii/?show=1) |      |
| [523. 连续的子数组和](https://leetcode.cn/problems/continuous-subarray-sum/?show=1) |      |
| [525. 连续数组](https://leetcode.cn/problems/contiguous-array/?show=1) |      |
| [560. 和为 K 的子数组](https://leetcode.cn/problems/subarray-sum-equals-k/?show=1) |      |