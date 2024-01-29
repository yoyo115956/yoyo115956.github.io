---
layout:       post
title: '我写了首诗，把二分搜索算法变成了默写题'
date: 2023-01-14
header-style: text
catalog:      true
mathjax:      true
tags:
  - 数组
  - 双指针
  - 二分查找
  - 核心框架
---

读完本文，你不仅学会了算法套路，还可以顺便解决如下题目：

原文：[我写了首诗，把二分搜索算法变成了默写题 labuladong 的算法笔记 (gitee.io)](https://labuladong.gitee.io/algo/di-ling-zh-bfe1b/wo-xie-le--3c789/)

视频：[【labuladong】二分搜索核心框架套路_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Gt4y1b79Q/?spm_id_from=333.788&vd_source=305afb3d9183503c497178f9d3f3284c)

|                             力扣                             | 难度  |      |
| :----------------------------------------------------------: | :---: | ---- |
| [34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/) |   🟠   | 完成 |
| [704. 二分查找](https://leetcode.cn/problems/binary-search/) |   🟢   | 完成 |
| [剑指 Offer 53 - I. 在排序数组中查找数字 I](https://leetcode.cn/problems/zai-pai-xu-shu-zu-zhong-cha-zhao-shu-zi-lcof/) | **🟢** |      |

零、二分查找框架
======

```java
int binarySearch(int[] nums, int target) {
    int left = 0, right = ...;

    while(...) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            ...
        } else if (nums[mid] < target) {
            left = ...
        } else if (nums[mid] > target) {
            right = ...
        }
    }
    return ...;
}
```

一、寻找一个数（基本的二分搜索）
------

力扣第 704 题「[二分查找](https://leetcode.cn/problems/binary-search/)」

```java
int binarySearch(int[] nums, int target) {
    int left = 0; 
    int right = nums.length - 1; // 注意

    while(left <= right) {
        int mid = left + (right - left) / 2;
        if(nums[mid] == target)
            return mid; 
        else if (nums[mid] < target)
            left = mid + 1; // 注意
        else if (nums[mid] > target)
            right = mid - 1; // 注意
    }
    return -1;
}
```

二、寻找左侧边界的二分搜索
------

```java
int left_bound(int[] nums, int target) {
    int left = 0;
    int right = nums.length; // 注意
    
    while (left < right) { // 注意
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            right = mid;
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid; // 注意
        }
    }
    return left;
}
```

```java
int left_bound(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    // 搜索区间为 [left, right]
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            // 搜索区间变为 [mid+1, right]
            left = mid + 1;
        } else if (nums[mid] > target) {
            // 搜索区间变为 [left, mid-1]
            right = mid - 1;
        } else if (nums[mid] == target) {
            // 收缩右侧边界
            right = mid - 1;
        }
    }
    // 判断 target 是否存在于 nums 中
    // 如果越界，target 肯定不存在，返回 -1
    if (left < 0 || left >= nums.length) {
        return -1;
    }
    // 判断一下 nums[left] 是不是 target
    return nums[left] == target ? left : -1;
}
```


三、寻找右侧边界的二分查找
======

```java
int right_bound(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        } else if (nums[mid] == target) {
            // 这里改成收缩左侧边界即可
            left = mid + 1;
        }
    }
    // 最后改成返回 left - 1
    if (left - 1 < 0 || left - 1 >= nums.length) {
        return -1;
    }
    return nums[left - 1] == target ? (left - 1) : -1;
}
```

<img src="https://s2.loli.net/2024/01/23/w64ah7esnyOL1Qk.jpg" alt="img" style="zoom:33%;" />

```java
int right_bound(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        } else if (nums[mid] == target) {
            // 这里改成收缩左侧边界即可
            left = mid + 1;
        }
    }
    // 最后改成返回 right
    if (right < 0 || right >= nums.length) {
        return -1;
    }
    return nums[right] == target ? right : -1;
}
```

四、逻辑统一
======

力扣第 34 题「[在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/)」

**第一个，最基本的二分查找算法**：

```java
因为我们初始化 right = nums.length - 1
所以决定了我们的「搜索区间」是 [left, right]
所以决定了 while (left <= right)
同时也决定了 left = mid+1 和 right = mid-1

因为我们只需找到一个 target 的索引即可
所以当 nums[mid] == target 时可以立即返回
```

**第二个，寻找左侧边界的二分查找**：

```java
因为我们初始化 right = nums.length
所以决定了我们的「搜索区间」是 [left, right)
所以决定了 while (left < right)
同时也决定了 left = mid + 1 和 right = mid

因为我们需找到 target 的最左侧索引
所以当 nums[mid] == target 时不要立即返回
而要收紧右侧边界以锁定左侧边界
```

**第三个，寻找右侧边界的二分查找**：

```java
因为我们初始化 right = nums.length
所以决定了我们的「搜索区间」是 [left, right)
所以决定了 while (left < right)
同时也决定了 left = mid + 1 和 right = mid

因为我们需找到 target 的最右侧索引
所以当 nums[mid] == target 时不要立即返回
而要收紧左侧边界以锁定右侧边界

又因为收紧左侧边界时必须 left = mid + 1
所以最后无论返回 left 还是 right，必须减一
```

对于寻找左右边界的二分搜索，比较常见的手法是使用左闭右开的「搜索区间」，**我们还根据逻辑将「搜索区间」全都统一成了两端都闭，便于记忆，只要修改两处即可变化出三种写法**：

```java
int binary_search(int[] nums, int target) {
    int left = 0, right = nums.length - 1; 
    while(left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1; 
        } else if(nums[mid] == target) {
            // 直接返回
            return mid;
        }
    }
    // 直接返回
    return -1;
}

int left_bound(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        } else if (nums[mid] == target) {
            // 别返回，锁定左侧边界
            right = mid - 1;
        }
    }
    // 判断 target 是否存在于 nums 中
    if (left < 0 || left >= nums.length) {
        return -1;
    }
    // 判断一下 nums[left] 是不是 target
    return nums[left] == target ? left : -1;
}

int right_bound(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        } else if (nums[mid] == target) {
            // 别返回，锁定右侧边界
            left = mid + 1;
        }
    }
    // 判断 target 是否存在于 nums 中
    // if (left - 1 < 0 || left - 1 >= nums.length) {
    //     return -1;
    // }
    
    // 由于 while 的结束条件是 right == left - 1，且现在在求右边界
    // 所以用 right 替代 left - 1 更好记
    if (right < 0 || right >= nums.length) {
        return -1;
    }
    return nums[right] == target ? right : -1;
}
```

|                             力扣                             |      |
| :----------------------------------------------------------: | ---- |
| [1201. 丑数 III](https://leetcode.cn/problems/ugly-number-iii/?show=1) |      |
| [162. 寻找峰值](https://leetcode.cn/problems/find-peak-element/?show=1) |      |
| [240. 搜索二维矩阵 II](https://leetcode.cn/problems/search-a-2d-matrix-ii/?show=1) |      |
| [33. 搜索旋转排序数组](https://leetcode.cn/problems/search-in-rotated-sorted-array/?show=1) |      |
| [35. 搜索插入位置](https://leetcode.cn/problems/search-insert-position/?show=1) |      |
| [658. 找到 K 个最接近的元素](https://leetcode.cn/problems/find-k-closest-elements/?show=1) |      |
| [74. 搜索二维矩阵](https://leetcode.cn/problems/search-a-2d-matrix/?show=1) |      |
| [793. 阶乘函数后 K 个零](https://leetcode.cn/problems/preimage-size-of-factorial-zeroes-function/?show=1) |      |
| [81. 搜索旋转排序数组 II](https://leetcode.cn/problems/search-in-rotated-sorted-array-ii/?show=1) |      |
| [852. 山脉数组的峰顶索引](https://leetcode.cn/problems/peak-index-in-a-mountain-array/?show=1) |      |
| [剑指 Offer 04. 二维数组中的查找](https://leetcode.cn/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/?show=1) |      |
| [剑指 Offer 53 - I. 在排序数组中查找数字 I](https://leetcode.cn/problems/zai-pai-xu-shu-zu-zhong-cha-zhao-shu-zi-lcof/?show=1) |      |
| [剑指 Offer 53 - II. 0～n-1中缺失的数字](https://leetcode.cn/problems/que-shi-de-shu-zi-lcof/?show=1) |      |
| [剑指 Offer II 068. 查找插入位置](https://leetcode.cn/problems/N6YdxV/?show=1) |      |
| [剑指 Offer II 069. 山峰数组的顶部](https://leetcode.cn/problems/B1IidL/?show=1) |      |