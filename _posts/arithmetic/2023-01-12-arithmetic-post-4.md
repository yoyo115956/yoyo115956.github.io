---
layout:       post
title: '小而美的算法技巧：差分数组'
date: 2023-01-12
header-style: text
catalog:      true
mathjax:      true
tags:
 - 数组
 - 差分数组
---

读完本文，你不仅学会了算法套路，还可以顺便解决如下题目：

原文：[小而美的算法技巧：差分数组 labuladong 的算法笔记 (gitee.io)](https://labuladong.gitee.io/algo/di-yi-zhan-da78c/shou-ba-sh-48c1d/xiao-er-me-c304e/)

视频：[【labuladong】前缀和/差分数组技巧精讲_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1NY4y1J7xQ/?spm_id_from=333.788&vd_source=305afb3d9183503c497178f9d3f3284c)

|                             力扣                             | 难度 |      |
| :----------------------------------------------------------: | :--: | ---- |
|   [1094. 拼车](https://leetcode.cn/problems/car-pooling/)    |  🟠   | 完成 |
| [1109. 航班预订统计](https://leetcode.cn/problems/corporate-flight-bookings/) |  🟠   | 完成 |
| [370. 区间加法](https://leetcode.cn/problems/range-addition/)🔒 |  🟠   | 完成 |

前缀和核心代码就是下面这段：

```java
class PrefixSum {
    // 前缀和数组
    private int[] preSum;

    /* 输入一个数组，构造前缀和 */
    public PrefixSum(int[] nums) {
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

**差分数组的主要适用场景是频繁对原始数组的某个区间的元素进行增减**。

```java
int[] diff = new int[nums.length];
// 构造差分数组
diff[0] = nums[0];
for (int i = 1; i < nums.length; i++) {
    diff[i] = nums[i] - nums[i - 1];
}
```

<img src="https://s2.loli.net/2024/01/23/jP78uGW2SvYJFTD.jpg" alt="img" style="zoom:33%;" />

```java
int[] res = new int[diff.length];
// 根据差分数组构造结果数组
res[0] = diff[0];
for (int i = 1; i < diff.length; i++) {
    res[i] = res[i - 1] + diff[i];
}
```

<img src="https://s2.loli.net/2024/01/23/znUqCb84yrXJOPB.jpg" alt="img" style="zoom:33%;" />

```java
// 差分数组工具类
class Difference {
    // 差分数组
    private int[] diff;
    
    /* 输入一个初始数组，区间操作将在这个数组上进行 */
    public Difference(int[] nums) {
        assert nums.length > 0;
        diff = new int[nums.length];
        // 根据初始数组构造差分数组
        diff[0] = nums[0];
        for (int i = 1; i < nums.length; i++) {
            diff[i] = nums[i] - nums[i - 1];
        }
    }

    /* 给闭区间 [i, j] 增加 val（可以是负数）*/
    public void increment(int i, int j, int val) {
        diff[i] += val;
        if (j + 1 < diff.length) {
            diff[j + 1] -= val;
        }
    }

    /* 返回结果数组 */
    public int[] result() {
        int[] res = new int[diff.length];
        // 根据差分数组构造结果数组
        res[0] = diff[0];
        for (int i = 1; i < diff.length; i++) {
            res[i] = res[i - 1] + diff[i];
        }
        return res;
    }
}
```

算法实践
======

力扣第 370 题「[区间加法](https://leetcode.cn/problems/range-addition/)」

```java
int[] getModifiedArray(int length, int[][] updates) {
    // nums 初始化为全 0
    int[] nums = new int[length];
    // 构造差分解法
    Difference df = new Difference(nums);
    
    for (int[] update : updates) {
        int i = update[0];
        int j = update[1];
        int val = update[2];
        df.increment(i, j, val);
    }
    
    return df.result();
}
```

力扣第 1109 题「[航班预订统计](https://leetcode.cn/problems/corporate-flight-bookings/)」

```java
int[] corpFlightBookings(int[][] bookings, int n) {
    // nums 初始化为全 0
    int[] nums = new int[n];
    // 构造差分解法
    Difference df = new Difference(nums);

    for (int[] booking : bookings) {
        // 注意转成数组索引要减一哦
        int i = booking[0] - 1;
        int j = booking[1] - 1;
        int val = booking[2];
        // 对区间 nums[i..j] 增加 val
        df.increment(i, j, val);
    }
    // 返回最终的结果数组
    return df.result();
}
```

力扣第 1094 题「[拼车](https://leetcode.cn/problems/car-pooling/)」

```java
boolean carPooling(int[][] trips, int capacity) {
    // 最多有 1001 个车站
    int[] nums = new int[1001];
    // 构造差分解法
    Difference df = new Difference(nums);
    
    for (int[] trip : trips) {
        // 乘客数量
        int val = trip[0];
        // 第 trip[1] 站乘客上车
        int i = trip[1];
        // 第 trip[2] 站乘客已经下车，
        // 即乘客在车上的区间是 [trip[1], trip[2] - 1]
        int j = trip[2] - 1;
        // 进行区间操作
        df.increment(i, j, val);
    }
    
    int[] res = df.result();
    
    // 客车自始至终都不应该超载
    for (int i = 0; i < res.length; i++) {
        if (capacity < res[i]) {
            return false;
        }
    }
    return true;
}
```