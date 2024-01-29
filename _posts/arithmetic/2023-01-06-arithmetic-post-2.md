---
layout:       post
title: '双指针：秒杀七道数组题目'
date: 2023-01-06
header-style: text
catalog:      true
mathjax:      true
tags:
 - 数据结构
 - 数组
 - 双指针
 - 核心框架
---

读完本文，你不仅学会了算法套路，还可以顺便解决如下题目：

原文：[双指针技巧秒杀七道数组题目 labuladong 的算法笔记 (gitee.io)](https://labuladong.gitee.io/algo/di-ling-zh-bfe1b/shuang-zhi-fa4bd/)

视频：[【labuladong】数组双指针技巧全面汇总_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1iG411W7Wm/?spm_id_from=333.788&vd_source=305afb3d9183503c497178f9d3f3284c)

|                             力扣                             | 难度 |      |
| :----------------------------------------------------------: | :--: | ---- |
| [167. 两数之和 II - 输入有序数组](https://leetcode.cn/problems/two-sum-ii-input-array-is-sorted/) |  🟠   | 完成 |
| [26. 删除有序数组中的重复项](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/) |  🟢   | 完成 |
| [27. 移除元素](https://leetcode.cn/problems/remove-element/) |  🟢   | 完成 |
|   [283. 移动零](https://leetcode.cn/problems/move-zeroes/)   |  🟢   | 完成 |
| [344. 反转字符串](https://leetcode.cn/problems/reverse-string/) |  🟢   | 完成 |
| [5. 最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/) |  🟠   | 完成 |
| [83. 删除排序链表中的重复元素](https://leetcode.cn/problems/remove-duplicates-from-sorted-list/) |  🟢   | 完成 |
| [剑指 Offer 57. 和为s的两个数字](https://leetcode.cn/problems/he-wei-sde-liang-ge-shu-zi-lcof/) |  🟢   |      |
| [剑指 Offer II 006. 排序数组中两个数字之和](https://leetcode.cn/problems/kLl5u1/) |  🟢   |      |

一、快慢指针技巧
======

力扣第 26 题「[删除有序数组中的重复项](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/)」

```java
int removeDuplicates(int[] nums) {
    if (nums.length == 0) {
        return 0;
    }
    int slow = 0, fast = 0;
    while (fast < nums.length) {
        if (nums[fast] != nums[slow]) {
            slow++;
            // 维护 nums[0..slow] 无重复
            nums[slow] = nums[fast];
        }
        fast++;
    }
    // 数组长度为索引 + 1
    return slow + 1;
}
```

<img src="https://s2.loli.net/2024/01/23/n35H1F6yDUJEbiP.gif" alt="img" style="zoom: 33%;" />

第 83 题「[删除排序链表中的重复元素](https://leetcode.cn/problems/remove-duplicates-from-sorted-list/)」

```java
ListNode deleteDuplicates(ListNode head) {
    if (head == null) return null;
    ListNode slow = head, fast = head;
    while (fast != null) {
        if (fast.val != slow.val) {
            // nums[slow] = nums[fast];
            slow.next = fast;
            // slow++;
            slow = slow.next;
        }
        // fast++
        fast = fast.next;
    }
    // 断开与后面重复元素的连接
    slow.next = null;
    return head;
}
```

<img src="https://s2.loli.net/2024/01/23/iTOcw5yZ7GejvIU.gif" alt="img" style="zoom: 33%;" />

第 27 题「[移除元素](https://leetcode.cn/problems/remove-element/)」

```java
int removeElement(int[] nums, int val) {
    int fast = 0, slow = 0;
    while (fast < nums.length) {
        if (nums[fast] != val) {
            nums[slow] = nums[fast];
            slow++;
        }
        fast++;
    }
    return slow;
}
```

第 283 题「[移动零](https://leetcode.cn/problems/move-zeroes/)」

```java
void moveZeroes(int[] nums) {
    // 去除 nums 中的所有 0，返回不含 0 的数组长度
    int p = removeElement(nums, 0);
    // 将 nums[p..] 的元素赋值为 0
    for (; p < nums.length; p++) {
        nums[p] = 0;
    }
}

// 见上文代码实现
int removeElement(int[] nums, int val);
```

滑动窗口的代码框架：

```java
/* 滑动窗口算法框架 */
void slidingWindow(string s, string t) {
    unordered_map<char, int> window;

    int left = 0, right = 0;
    while (right < s.size()) {
        char c = s[right];
        // 右移（增大）窗口
        right++;
        // 进行窗口内数据的一系列更新

        while (window needs shrink) {
            char d = s[left];
            // 左移（缩小）窗口
            left++;
            // 进行窗口内数据的一系列更新
        }
    }
}
```

二、左右指针的常用算法
======

1、二分查找
------

``` java
int binarySearch(int[] nums, int target) {
    // 一左一右两个指针相向而行
    int left = 0, right = nums.length - 1;
    while(left <= right) {
        int mid = (right + left) / 2;
        if(nums[mid] == target)
            return mid; 
        else if (nums[mid] < target)
            left = mid + 1; 
        else if (nums[mid] > target)
            right = mid - 1;
    }
    return -1;
}
```

2、两数之和
------

力扣第 167 题「[两数之和 II](https://leetcode.cn/problems/two-sum-ii-input-array-is-sorted/)」：

```java
int[] twoSum(int[] nums, int target) {
    // 一左一右两个指针相向而行
    int left = 0, right = nums.length - 1;
    while (left < right) {
        int sum = nums[left] + nums[right];
        if (sum == target) {
            // 题目要求的索引是从 1 开始的
            return new int[]{left + 1, right + 1};
        } else if (sum < target) {
            left++; // 让 sum 大一点
        } else if (sum > target) {
            right--; // 让 sum 小一点
        }
    }
    return new int[]{-1, -1};
}
```

另一篇文章 [一个函数秒杀所有 nSum 问题](https://labuladong.gitee.io/algo/di-ling-zh-bfe1b/yi-ge-fang-894da/) 


3、反转数组
------

力扣第 344 题「[反转字符串](https://leetcode.cn/problems/reverse-string/)」

``` java
void reverseString(char[] s) {
    // 一左一右两个指针相向而行
    int left = 0, right = s.length - 1;
    while (left < right) {
        // 交换 s[left] 和 s[right]
        char temp = s[left];
        s[left] = s[right];
        s[right] = temp;
        left++;
        right--;
    }
}
```

4、回文串判断
------


力扣第 5 题「[最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/)」：

```java
// 在 s 中寻找以 s[l] 和 s[r] 为中心的最长回文串
String palindrome(String s, int l, int r) {
    // 防止索引越界
    while (l >= 0 && r < s.length()
            && s.charAt(l) == s.charAt(r)) {
        // 双指针，向两边展开
        l--; r++;
    }
    // 返回以 s[l] 和 s[r] 为中心的最长回文串
    return s.substring(l + 1, r);
}
```

```
for 0 <= i < len(s):
    找到以 s[i] 为中心的回文串
    找到以 s[i] 和 s[i+1] 为中心的回文串
    更新答案
```

```java
String longestPalindrome(String s) {
    String res = "";
    for (int i = 0; i < s.length(); i++) {
        // 以 s[i] 为中心的最长回文子串
        String s1 = palindrome(s, i, i);
        // 以 s[i] 和 s[i+1] 为中心的最长回文子串
        String s2 = palindrome(s, i, i + 1);
        // res = longest(res, s1, s2)
        res = res.length() > s1.length() ? res : s1;
        res = res.length() > s2.length() ? res : s2;
    }
    return res;
}
```

|                             力扣                             |      |
| :----------------------------------------------------------: | ---- |
| [1. 两数之和](https://leetcode.cn/problems/two-sum/?show=1)  |      |
| [125. 验证回文串](https://leetcode.cn/problems/valid-palindrome/?show=1) |      |
| [131. 分割回文串](https://leetcode.cn/problems/palindrome-partitioning/?show=1) |      |
| [281. 锯齿迭代器](https://leetcode.cn/problems/zigzag-iterator/?show=1)🔒 |      |
| [42. 接雨水](https://leetcode.cn/problems/trapping-rain-water/?show=1) |      |
| [658. 找到 K 个最接近的元素](https://leetcode.cn/problems/find-k-closest-elements/?show=1) |      |
| [80. 删除有序数组中的重复项 II](https://leetcode.cn/problems/remove-duplicates-from-sorted-array-ii/?show=1) |      |
| [82. 删除排序链表中的重复元素 II](https://leetcode.cn/problems/remove-duplicates-from-sorted-list-ii/?show=1) |      |
| [9. 回文数](https://leetcode.cn/problems/palindrome-number/?show=1) |      |
| [剑指 Offer 21. 调整数组顺序使奇数位于偶数前面](https://leetcode.cn/problems/diao-zheng-shu-zu-shun-xu-shi-qi-shu-wei-yu-ou-shu-qian-mian-lcof/?show=1) |      |
| [剑指 Offer 57. 和为s的两个数字](https://leetcode.cn/problems/he-wei-sde-liang-ge-shu-zi-lcof/?show=1) |      |
| [剑指 Offer II 018. 有效的回文](https://leetcode.cn/problems/XltzEq/?show=1) |      |
