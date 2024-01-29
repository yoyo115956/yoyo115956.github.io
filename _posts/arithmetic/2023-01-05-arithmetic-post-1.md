---
layout:       post
title: '双指针：秒杀七道链表题目'
date: 2023-01-05
header-style: text
catalog:      true
mathjax:      true
tags:
  - 数据结构
  - 链表
  - 双指针
  - 核心框架
---

读完本文，你不仅学会了算法套路，还可以顺便解决如下题目：

原文：[双指针技巧秒杀七道链表题目 labuladong 的算法笔记 (gitee.io)](https://labuladong.gitee.io/algo/di-ling-zh-bfe1b/shuang-zhi-0f7cc/)

视频：[【labuladong】链表双指针技巧全面汇总_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1q94y1X7vy/?spm_id_from=333.788&vd_source=305afb3d9183503c497178f9d3f3284c)



|                             力扣                             | 难度 |      |
| :----------------------------------------------------------: | :--: | ---- |
| [141. 环形链表](https://leetcode.cn/problems/linked-list-cycle/) |  🟢   | 完成 |
| [142. 环形链表 II](https://leetcode.cn/problems/linked-list-cycle-ii/) |  🟠   | 完成 |
| [160. 相交链表](https://leetcode.cn/problems/intersection-of-two-linked-lists/) |  🟢   | 完成 |
| [19. 删除链表的倒数第 N 个结点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/) |  🟠   | 完成 |
| [21. 合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists/) |  🟢   | 完成 |
| [23. 合并K个升序链表](https://leetcode.cn/problems/merge-k-sorted-lists/) |  🔴   | 完成 |
| [86. 分隔链表](https://leetcode.cn/problems/partition-list/) |  🟠   | 完成 |
| [876. 链表的中间结点](https://leetcode.cn/problems/middle-of-the-linked-list/) |  🟢   | 完成 |
| [剑指 Offer 22. 链表中倒数第k个节点](https://leetcode.cn/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/) |  🟢   |      |
| [剑指 Offer 25. 合并两个排序的链表](https://leetcode.cn/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof/) |  🟢   |      |
| [剑指 Offer 52. 两个链表的第一个公共节点](https://leetcode.cn/problems/liang-ge-lian-biao-de-di-yi-ge-gong-gong-jie-dian-lcof/) |  🟢   |      |
| [剑指 Offer II 021. 删除链表的倒数第 n 个结点](https://leetcode.cn/problems/SLwz0R/) |  🟠   |      |
| [剑指 Offer II 022. 链表中环的入口节点](https://leetcode.cn/problems/c32eOV/) |  🟠   |      |
| [剑指 Offer II 023. 两个链表的第一个重合节点](https://leetcode.cn/problems/3u1WK4/) |  🟢   |      |
| [剑指 Offer II 078. 合并排序链表](https://leetcode.cn/problems/vvXgSW/) |  🔴   |      |





一、合并两个有序链表
======

力扣第 21 题「[合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists/)」：

```java
ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    // 虚拟头结点
    ListNode dummy = new ListNode(-1), p = dummy;
    ListNode p1 = l1, p2 = l2;
    
    while (p1 != null && p2 != null) {
        // 比较 p1 和 p2 两个指针
        // 将值较小的的节点接到 p 指针
        if (p1.val > p2.val) {
            p.next = p2;
            p2 = p2.next;
        } else {
            p.next = p1;
            p1 = p1.next;
        }
        // p 指针不断前进
        p = p.next;
    }
    
    if (p1 != null) {
        p.next = p1;
    }
    
    if (p2 != null) {
        p.next = p2;
    }
    
    return dummy.next;
}

```

<img src="https://s2.loli.net/2024/01/23/6jp7KSQvdtCayfk.gif" alt="img" style="zoom: 33%;" />

二、单链表的分解
======

力扣第 86 题「[分隔链表](https://leetcode.cn/problems/partition-list/)」：

```java
ListNode partition(ListNode head, int x) {
    // 存放小于 x 的链表的虚拟头结点
    ListNode dummy1 = new ListNode(-1);
    // 存放大于等于 x 的链表的虚拟头结点
    ListNode dummy2 = new ListNode(-1);
    // p1, p2 指针负责生成结果链表
    ListNode p1 = dummy1, p2 = dummy2;
    // p 负责遍历原链表，类似合并两个有序链表的逻辑
    // 这里是将一个链表分解成两个链表
    ListNode p = head;
    while (p != null) {
        if (p.val >= x) {
            p2.next = p;
            p2 = p2.next;
        } else {
            p1.next = p;
            p1 = p1.next;
        }
        // 不能直接让 p 指针前进，
        // p = p.next
        // 断开原链表中的每个节点的 next 指针
        ListNode temp = p.next;
        p.next = null;
        p = temp;
    }
    // 连接两个链表
    p1.next = dummy2.next;

    return dummy1.next;
}

```

三、合并 k 个有序链表
======

力扣第 23 题「[合并K个升序链表](https://leetcode.cn/problems/merge-k-sorted-lists/)」：

```java
ListNode mergeKLists(ListNode[] lists) {
    if (lists.length == 0) return null;
    // 虚拟头结点
    ListNode dummy = new ListNode(-1);
    ListNode p = dummy;
    // 优先级队列，最小堆
    PriorityQueue<ListNode> pq = new PriorityQueue<>(
        lists.length, (a, b)->(a.val - b.val));
    // 将 k 个链表的头结点加入最小堆
    for (ListNode head : lists) {
        if (head != null)
            pq.add(head);
    }

    while (!pq.isEmpty()) {
        // 获取最小节点，接到结果链表中
        ListNode node = pq.poll();
        p.next = node;
        if (node.next != null) {
            pq.add(node.next);
        }
        // p 指针不断前进
        p = p.next;
    }
    return dummy.next;
}
```


四、单链表的倒数第 k 个节点
======

<img src="https://s2.loli.net/2024/01/23/7kpdeJ4wZTAQMhl.jpg" alt="img" style="zoom: 33%;" />

<img src="https://s2.loli.net/2024/01/23/2lRZOsL8GqnbUSm.jpg" alt="img" style="zoom: 33%;" />

```java
// 返回链表的倒数第 k 个节点
ListNode findFromEnd(ListNode head, int k) {
    ListNode p1 = head;
    // p1 先走 k 步
    for (int i = 0; i < k; i++) {
        p1 = p1.next;
    }
    ListNode p2 = head;
    // p1 和 p2 同时走 n - k 步
    while (p1 != null) {
        p2 = p2.next;
        p1 = p1.next;
    }
    // p2 现在指向第 n - k + 1 个节点，即倒数第 k 个节点
    return p2;
}
```

力扣第 19 题「[删除链表的倒数第 N 个结点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)」

```java
// 主函数
public ListNode removeNthFromEnd(ListNode head, int n) {
    // 虚拟头结点
    ListNode dummy = new ListNode(-1);
    dummy.next = head;
    // 删除倒数第 n 个，要先找倒数第 n + 1 个节点
    ListNode x = findFromEnd(dummy, n + 1);
    // 删掉倒数第 n 个节点
    x.next = x.next.next;
    return dummy.next;
}
    
private ListNode findFromEnd(ListNode head, int k) {
    // 代码见上文
}
```

五、单链表的中点
======

力扣第 876 题「[链表的中间结点](https://leetcode.cn/problems/middle-of-the-linked-list/)」

```java
ListNode middleNode(ListNode head) {
    // 快慢指针初始化指向 head
    ListNode slow = head, fast = head;
    // 快指针走到末尾时停止
    while (fast != null && fast.next != null) {
        // 慢指针走一步，快指针走两步
        slow = slow.next;
        fast = fast.next.next;
    }
    // 慢指针指向中点
    return slow;
}
```

六、判断链表是否包含环
======

[141. 环形链表](https://leetcode.cn/problems/linked-list-cycle/)

```java
boolean hasCycle(ListNode head) {
    // 快慢指针初始化指向 head
    ListNode slow = head, fast = head;
    // 快指针走到末尾时停止
    while (fast != null && fast.next != null) {
        // 慢指针走一步，快指针走两步
        slow = slow.next;
        fast = fast.next.next;
        // 快慢指针相遇，说明含有环
        if (slow == fast) {
            return true;
        }
    }
    // 不包含环
    return false;
}
```

力扣第 142 题「[环形链表 II](https://leetcode.cn/problems/linked-list-cycle-ii/)」

![img](https://s2.loli.net/2024/01/23/lBAaP9fO3xHsJd1.png)

```java
class Solution {
    public ListNode detectCycle(ListNode head) {
        ListNode fast, slow;
        fast = slow = head;
        while (fast != null && fast.next != null) {
            fast = fast.next.next;
            slow = slow.next;
            if (fast == slow) break;
        }
        // 上面的代码类似 hasCycle 函数
        if (fast == null || fast.next == null) {
            // fast 遇到空指针说明没有环
            return null;
        }

        // 重新指向头结点
        slow = head;
        // 快慢指针同步前进，相交点就是环起点
        while (slow != fast) {
            fast = fast.next;
            slow = slow.next;
        }
        return slow;
    }
}
```

<img src="https://s2.loli.net/2024/01/23/O7F4sPCMWvUSwya.png" alt="image-20240123133506853" style="zoom: 33%;" />

<img src="https://s2.loli.net/2024/01/23/who1J3I5N9alVLe.png" alt="image-20240123133523486" style="zoom: 33%;" />

七、两个链表是否相交
======

力扣第 160 题「[相交链表](https://leetcode.cn/problems/intersection-of-two-linked-lists/)」

<img src="https://s2.loli.net/2024/01/23/uSjVlRgbtixhJzD.png" alt="img" style="zoom:50%;" />

<img src="https://s2.loli.net/2024/01/23/E3aze8LHRnw1uXY.jpg" alt="img" style="zoom: 33%;" />

<img src="https://s2.loli.net/2024/01/23/MDlutSx2Wk4UPKO.jpg" alt="img" style="zoom: 33%;" />

```java
ListNode getIntersectionNode(ListNode headA, ListNode headB) {
    // p1 指向 A 链表头结点，p2 指向 B 链表头结点
    ListNode p1 = headA, p2 = headB;
    while (p1 != p2) {
        // p1 走一步，如果走到 A 链表末尾，转到 B 链表
        if (p1 == null) p1 = headB;
        else            p1 = p1.next;
        // p2 走一步，如果走到 B 链表末尾，转到 A 链表
        if (p2 == null) p2 = headA;
        else            p2 = p2.next;
    }
    return p1;
}
```

<img src="https://s2.loli.net/2024/01/23/BLyWX4eZKn3SRzr.png" alt="img" style="zoom: 33%;" />

```java
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
    int lenA = 0, lenB = 0;
    // 计算两条链表的长度
    for (ListNode p1 = headA; p1 != null; p1 = p1.next) {
        lenA++;
    }
    for (ListNode p2 = headB; p2 != null; p2 = p2.next) {
        lenB++;
    }
    // 让 p1 和 p2 到达尾部的距离相同
    ListNode p1 = headA, p2 = headB;
    if (lenA > lenB) {
        for (int i = 0; i < lenA - lenB; i++) {
            p1 = p1.next;
        }
    } else {
        for (int i = 0; i < lenB - lenA; i++) {
            p2 = p2.next;
        }
    }
    // 看两个指针是否会相同，p1 == p2 时有两种情况：
    // 1. 要么是两条链表不相交，他俩同时走到尾部空指针
    // 2. 要么是两条链表相交，他俩走到两条链表的相交点
    while (p1 != p2) {
        p1 = p1.next;
        p2 = p2.next;
    }
    return p1;
}
```

|                             力扣                             |      |
| :----------------------------------------------------------: | ---- |
| [109. 有序链表转换二叉搜索树](https://leetcode.cn/problems/convert-sorted-list-to-binary-search-tree/?show=1) |      |
| [1257. 最小公共区域](https://leetcode.cn/problems/smallest-common-region/?show=1)🔒 |      |
| [1650. 二叉树的最近公共祖先 III](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree-iii/?show=1)🔒 |      |
| [1836. 从未排序的链表中移除重复元素](https://leetcode.cn/problems/remove-duplicates-from-an-unsorted-linked-list/?show=1)🔒 |      |
| [2. 两数相加](https://leetcode.cn/problems/add-two-numbers/?show=1) |      |
| [234. 回文链表](https://leetcode.cn/problems/palindrome-linked-list/?show=1) |      |
| [264. 丑数 II](https://leetcode.cn/problems/ugly-number-ii/?show=1) |      |
| [313. 超级丑数](https://leetcode.cn/problems/super-ugly-number/?show=1) |      |
| [355. 设计推特](https://leetcode.cn/problems/design-twitter/?show=1) |      |
| [360. 有序转化数组](https://leetcode.cn/problems/sort-transformed-array/?show=1)🔒 |      |
| [373. 查找和最小的 K 对数字](https://leetcode.cn/problems/find-k-pairs-with-smallest-sums/?show=1) |      |
| [378. 有序矩阵中第 K 小的元素](https://leetcode.cn/problems/kth-smallest-element-in-a-sorted-matrix/?show=1) |      |
| [431. 将 N 叉树编码为二叉树](https://leetcode.cn/problems/encode-n-ary-tree-to-binary-tree/?show=1)🔒 |      |
| [88. 合并两个有序数组](https://leetcode.cn/problems/merge-sorted-array/?show=1) |      |
| [97. 交错字符串](https://leetcode.cn/problems/interleaving-string/?show=1) |      |
| [977. 有序数组的平方](https://leetcode.cn/problems/squares-of-a-sorted-array/?show=1) |      |
| [剑指 Offer 18. 删除链表的节点](https://leetcode.cn/problems/shan-chu-lian-biao-de-jie-dian-lcof/?show=1) |      |
| [剑指 Offer 25. 合并两个排序的链表](https://leetcode.cn/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof/?show=1) |      |
| [剑指 Offer 49. 丑数](https://leetcode.cn/problems/chou-shu-lcof/?show=1) |      |
| [剑指 Offer 52. 两个链表的第一个公共节点](https://leetcode.cn/problems/liang-ge-lian-biao-de-di-yi-ge-gong-gong-jie-dian-lcof/?show=1) |      |
| [剑指 Offer II 021. 删除链表的倒数第 n 个结点](https://leetcode.cn/problems/SLwz0R/?show=1) |      |
| [剑指 Offer II 022. 链表中环的入口节点](https://leetcode.cn/problems/c32eOV/?show=1) |      |
| [剑指 Offer II 023. 两个链表的第一个重合节点](https://leetcode.cn/problems/3u1WK4/?show=1) |      |
| [剑指 Offer II 027. 回文链表](https://leetcode.cn/problems/aMhZSa/?show=1) |      |
| [剑指 Offer II 061. 和最小的 k 个数对](https://leetcode.cn/problems/qn8gGX/?show=1) |      |
| [剑指 Offer II 078. 合并排序链表](https://leetcode.cn/problems/vvXgSW/?show=1) |      |

