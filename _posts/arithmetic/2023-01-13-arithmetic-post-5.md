---
layout:       post
title: '动态规划解题套路框架'
date: 2023-01-13
header-style: text
catalog:      true
mathjax:      true
tags:
 - 动态规划
 - 核心框架
---

读完本文，你不仅学会了算法套路，还可以顺便解决如下题目：

原文：[动态规划解题套路框架 labuladong 的算法笔记 (gitee.io)](https://labuladong.gitee.io/algo/di-ling-zh-bfe1b/dong-tai-g-1e688/)

视频：[【labuladong】滑动窗口算法核心模板框架_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1AV4y1n7Zt/?spm_id_from=333.788&vd_source=305afb3d9183503c497178f9d3f3284c)

|                             力扣                             | 难度 |      |
| :----------------------------------------------------------: | :--: | ---- |
|  [322. 零钱兑换](https://leetcode.cn/problems/coin-change/)  |  🟠   | 完成 |
| [509. 斐波那契数](https://leetcode.cn/problems/fibonacci-number/) |  🟢   | 完成 |
| [剑指 Offer II 103. 最少的硬币数目](https://leetcode.cn/problems/gaM7Ch/) |  🟠   |      |

明确 base case -> 明确「状态」-> 明确「选择」 -> 定义 dp 数组/函数的含义。

```python
# 自顶向下递归的动态规划
def dp(状态1, 状态2, ...):
    for 选择 in 所有可能的选择:
        # 此时的状态已经因为做了选择而改变
        result = 求最值(result, dp(状态1, 状态2, ...))
    return result

# 自底向上迭代的动态规划
# 初始化 base case
dp[0][0][...] = base case
# 进行状态转移
for 状态1 in 状态1的所有取值：
    for 状态2 in 状态2的所有取值：
        for ...
            dp[状态1][状态2][...] = 求最值(选择1，选择2...)
```



一、斐波那契数列
======

1、暴力递归
------

力扣第 509 题「[斐波那契数](https://leetcode.cn/problems/fibonacci-number/)」

``` java
int fib(int N) {
    if (N == 1 || N == 2) return 1;
    return fib(N - 1) + fib(N - 2);
}
```

<img src="https://s2.loli.net/2024/01/23/5F1PM78tDmChNox.jpg" alt="img" style="zoom:33%;" />

2、带备忘录的递归解法
------

```java
int fib(int N) {
    // 备忘录全初始化为 0
    int[] memo = new int[N + 1];
    // 进行带备忘录的递归
    return dp(memo, N);
}

// 带着备忘录进行递归
int dp(int[] memo, int n) {
    // base case
    if (n == 0 || n == 1) return n;
    // 已经计算过，不用再计算了
    if (memo[n] != 0) return memo[n];
    memo[n] = dp(memo, n - 1) + dp(memo, n - 2);
    return memo[n];
}
```

<img src="https://s2.loli.net/2024/01/23/1PS6DdxBhqEnpw2.jpg" alt="img" style="zoom:33%;" />

<img src="https://s2.loli.net/2024/01/23/3w27QZW9tRHSB14.jpg" alt="img" style="zoom:33%;" />

3、`dp` 数组的迭代（递推）解法
------

```java
int fib(int N) {
    if (N == 0) return 0;
    int[] dp = new int[N + 1];
    // base case
    dp[0] = 0; dp[1] = 1;
    // 状态转移
    for (int i = 2; i <= N; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];
    }

    return dp[N];
}
```

```java
int fib(int n) {
    if (n == 0 || n == 1) {
        // base case
        return n;
    }
    // 分别代表 dp[i - 1] 和 dp[i - 2]
    int dp_i_1 = 1, dp_i_2 = 0;
    for (int i = 2; i <= n; i++) {
        // dp[i] = dp[i - 1] + dp[i - 2];
        int dp_i = dp_i_1 + dp_i_2;
        // 滚动更新
        dp_i_2 = dp_i_1;
        dp_i_1 = dp_i;
    }
    return dp_i_1;
}
```

二、凑零钱问题
======

力扣第 322 题「[零钱兑换](https://leetcode.cn/problems/coin-change/)」：

```java
// 伪码框架
int coinChange(int[] coins, int amount) {
    // 题目要求的最终结果是 dp(amount)
    return dp(coins, amount)
}

// 定义：要凑出金额 n，至少要 dp(coins, n) 个硬币
int dp(int[] coins, int n) {
    // 做选择，选择需要硬币最少的那个结果
    for (int coin : coins) {
        res = min(res, 1 + dp(coins, n - coin))
    }
    return res
}
```

```java
int coinChange(int[] coins, int amount) {
    // 题目要求的最终结果是 dp(amount)
    return dp(coins, amount)
}

// 定义：要凑出金额 n，至少要 dp(coins, n) 个硬币
int dp(int[] coins, int amount) {
    // base case
    if (amount == 0) return 0;
    if (amount < 0) return -1;

    int res = Integer.MAX_VALUE;
    for (int coin : coins) {
        // 计算子问题的结果
        int subProblem = dp(coins, amount - coin);
        // 子问题无解则跳过
        if (subProblem == -1) continue;
        // 在子问题中选择最优解，然后加一
        res = Math.min(res, subProblem + 1);
    }

    return res == Integer.MAX_VALUE ? -1 : res;
}
```

![img](https://labuladong.gitee.io/algo/images/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92%E8%AF%A6%E8%A7%A3%E8%BF%9B%E9%98%B6/coin.png)

<img src="https://s2.loli.net/2024/01/23/fq2dbniSR4LG97K.jpg" alt="img" style="zoom:33%;" />

2、带备忘录的递归
------

```java
class Solution {
    int[] memo;

    int coinChange(int[] coins, int amount) {
        memo = new int[amount + 1];
        // 备忘录初始化为一个不会被取到的特殊值，代表还未被计算
        Arrays.fill(memo, -666);

        return dp(coins, amount);
    }

    int dp(int[] coins, int amount) {
        if (amount == 0) return 0;
        if (amount < 0) return -1;
        // 查备忘录，防止重复计算
        if (memo[amount] != -666)
            return memo[amount];

        int res = Integer.MAX_VALUE;
        for (int coin : coins) {
            // 计算子问题的结果
            int subProblem = dp(coins, amount - coin);
            // 子问题无解则跳过
            if (subProblem == -1) continue;
            // 在子问题中选择最优解，然后加一
            res = Math.min(res, subProblem + 1);
        }
        // 把计算结果存入备忘录
        memo[amount] = (res == Integer.MAX_VALUE) ? -1 : res;
        return memo[amount];
    }
}
```

3、dp 数组的迭代解法
------

```java
int coinChange(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    // 数组大小为 amount + 1，初始值也为 amount + 1
    Arrays.fill(dp, amount + 1);

    // base case
    dp[0] = 0;
    // 外层 for 循环在遍历所有状态的所有取值
    for (int i = 0; i < dp.length; i++) {
        // 内层 for 循环在求所有选择的最小值
        for (int coin : coins) {
            // 子问题无解，跳过
            if (i - coin < 0) {
                continue;
            }
            dp[i] = Math.min(dp[i], 1 + dp[i - coin]);
        }
    }
    return (dp[amount] == amount + 1) ? -1 : dp[amount];
}
```

<img src="https://s2.loli.net/2024/01/23/lEMi8zW4PSreL3Q.jpg" alt="img" style="zoom:33%;" />

|                             力扣                             |      |
| :----------------------------------------------------------: | ---- |
| [111. 二叉树的最小深度](https://leetcode.cn/problems/minimum-depth-of-binary-tree/?show=1) |      |
| [112. 路径总和](https://leetcode.cn/problems/path-sum/?show=1) |      |
| [115. 不同的子序列](https://leetcode.cn/problems/distinct-subsequences/?show=1) |      |
| [139. 单词拆分](https://leetcode.cn/problems/word-break/?show=1) |      |
| [1696. 跳跃游戏 VI](https://leetcode.cn/problems/jump-game-vi/?show=1) |      |
| [221. 最大正方形](https://leetcode.cn/problems/maximal-square/?show=1) |      |
| [240. 搜索二维矩阵 II](https://leetcode.cn/problems/search-a-2d-matrix-ii/?show=1) |      |
| [256. 粉刷房子](https://leetcode.cn/problems/paint-house/?show=1)🔒 |      |
| [279. 完全平方数](https://leetcode.cn/problems/perfect-squares/?show=1) |      |
| [343. 整数拆分](https://leetcode.cn/problems/integer-break/?show=1) |      |
| [365. 水壶问题](https://leetcode.cn/problems/water-and-jug-problem/?show=1) |      |
| [542. 01 矩阵](https://leetcode.cn/problems/01-matrix/?show=1) |      |
| [576. 出界的路径数](https://leetcode.cn/problems/out-of-boundary-paths/?show=1) |      |
| [62. 不同路径](https://leetcode.cn/problems/unique-paths/?show=1) |      |
| [63. 不同路径 II](https://leetcode.cn/problems/unique-paths-ii/?show=1) |      |
| [70. 爬楼梯](https://leetcode.cn/problems/climbing-stairs/?show=1) |      |
| [91. 解码方法](https://leetcode.cn/problems/decode-ways/?show=1) |      |
| [剑指 Offer 04. 二维数组中的查找](https://leetcode.cn/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/?show=1) |      |
| [剑指 Offer 10- I. 斐波那契数列](https://leetcode.cn/problems/fei-bo-na-qi-shu-lie-lcof/?show=1) |      |
| [剑指 Offer 10- II. 青蛙跳台阶问题](https://leetcode.cn/problems/qing-wa-tiao-tai-jie-wen-ti-lcof/?show=1) |      |
| [剑指 Offer 14- I. 剪绳子](https://leetcode.cn/problems/jian-sheng-zi-lcof/?show=1) |      |
| [剑指 Offer 46. 把数字翻译成字符串](https://leetcode.cn/problems/ba-shu-zi-fan-yi-cheng-zi-fu-chuan-lcof/?show=1) |      |
| [剑指 Offer II 091. 粉刷房子](https://leetcode.cn/problems/JEj789/?show=1) |      |
| [剑指 Offer II 097. 子序列的数目](https://leetcode.cn/problems/21dk04/?show=1) |      |
| [剑指 Offer II 098. 路径的数目](https://leetcode.cn/problems/2AoeFn/?show=1) |      |
| [剑指 Offer II 103. 最少的硬币数目](https://leetcode.cn/problems/gaM7Ch/?show=1) |      |