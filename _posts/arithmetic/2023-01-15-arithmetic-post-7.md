---
layout:       post
title: '二叉树（纲领篇）'
date: 2023-01-15
header-style: text
catalog:      true
mathjax:      true
tags:
  - 数据结构
  - 二叉树
  - 分解问题的思路
  - 遍历的思路
  - 核心框架
---

读完本文，你不仅学会了算法套路，还可以顺便解决如下题目：

原文：[东哥带你刷二叉树（纲领篇） labuladong 的算法笔记 (gitee.io)](https://labuladong.gitee.io/algo/di-yi-zhan-da78c/shou-ba-sh-66994/dong-ge-da-334dd/)

视频：[【labuladong】二叉树/递归的框架思维（纲领篇）_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1nG411x77H/?spm_id_from=333.788&vd_source=305afb3d9183503c497178f9d3f3284c)

|                             力扣                             | 难度 |      |
| :----------------------------------------------------------: | :--: | ---- |
| [104. 二叉树的最大深度](https://leetcode.cn/problems/maximum-depth-of-binary-tree/) |  🟢   | 完成 |
| [144. 二叉树的前序遍历](https://leetcode.cn/problems/binary-tree-preorder-traversal/) |  🟢   | 完成 |
| [543. 二叉树的直径](https://leetcode.cn/problems/diameter-of-binary-tree/) |  🟢   | 完成 |
| [剑指 Offer 55 - I. 二叉树的深度](https://leetcode.cn/problems/er-cha-shu-de-shen-du-lcof/) |  🟢   |      |

二叉树的重要性
======

快速排序的代码框架如下：

```java
void sort(int[] nums, int lo, int hi) {
    /****** 前序遍历位置 ******/
    // 通过交换元素构建分界点 p
    int p = partition(nums, lo, hi);
    /************************/

    sort(nums, lo, p - 1);
    sort(nums, p + 1, hi);
}
```

归并排序的代码框架如下：

```java
// 定义：排序 nums[lo..hi]
void sort(int[] nums, int lo, int hi) {
    int mid = (lo + hi) / 2;
    // 排序 nums[lo..mid]
    sort(nums, lo, mid);
    // 排序 nums[mid+1..hi]
    sort(nums, mid + 1, hi);

    /****** 后序位置 ******/
    // 合并 nums[lo..mid] 和 nums[mid+1..hi]
    merge(nums, lo, mid, hi);
    /*********************/
}
```

深入理解前中后序
======

```java
void traverse(TreeNode root) {
    if (root == null) {
        return;
    }
    // 前序位置
    traverse(root.left);
    // 中序位置
    traverse(root.right);
    // 后序位置
}
```

```java
/* 迭代遍历数组 */
void traverse(int[] arr) {
    for (int i = 0; i < arr.length; i++) {

    }
}

/* 递归遍历数组 */
void traverse(int[] arr, int i) {
    if (i == arr.length) {
        return;
    }
    // 前序位置
    traverse(arr, i + 1);
    // 后序位置
}

/* 迭代遍历单链表 */
void traverse(ListNode head) {
    for (ListNode p = head; p != null; p = p.next) {

    }
}

/* 递归遍历单链表 */
void traverse(ListNode head) {
    if (head == null) {
        return;
    }
    // 前序位置
    traverse(head.next);
    // 后序位置
}
```

<img src="https://s2.loli.net/2024/01/23/tEJeMKWYbAogTUz.jpg" alt="img" style="zoom:33%;" />

```java
/* 递归遍历单链表，倒序打印链表元素 */
void traverse(ListNode head) {
    if (head == null) {
        return;
    }
    traverse(head.next);
    // 后序位置
    print(head.val);
}
```

<img src="https://s2.loli.net/2024/01/23/vVLtoFMH57JWwk4.jpg" alt="img" style="zoom:33%;" />

两种解题思路
======

**二叉树题目的递归解法可以分两类思路，第一类是遍历一遍二叉树得出答案，第二类是通过分解问题计算出答案，这两类思路分别对应着 [回溯算法核心框架](https://labuladong.gitee.io/algo/di-ling-zh-bfe1b/hui-su-sua-c26da/) 和 [动态规划核心框架](https://labuladong.gitee.io/algo/di-ling-zh-bfe1b/dong-tai-g-1e688/)**。

力扣第 104 题「[二叉树的最大深度](https://leetcode.cn/problems/maximum-depth-of-binary-tree/)」

**这就是遍历二叉树计算答案的思路**解法代码如下：

```java
// 记录最大深度
int res = 0;
// 记录遍历到的节点的深度
int depth = 0;

// 主函数
int maxDepth(TreeNode root) {
	traverse(root);
	return res;
}

// 二叉树遍历框架
void traverse(TreeNode root) {
	if (root == null) {
		return;
	}
	// 前序位置
	depth++;
    if (root.left == null && root.right == null) {
        // 到达叶子节点，更新最大深度
		res = Math.max(res, depth);
    }
	traverse(root.left);
	traverse(root.right);
	// 后序位置
	depth--;
}
```

**分解问题计算答案的思路**解法代码如下：

```java
// 定义：输入根节点，返回这棵二叉树的最大深度
int maxDepth(TreeNode root) {
	if (root == null) {
		return 0;
	}
	// 利用定义，计算左右子树的最大深度
	int leftMax = maxDepth(root.left);
	int rightMax = maxDepth(root.right);
	// 整棵树的最大深度等于左右子树的最大深度取最大值，
    // 然后再加上根节点自己
	int res = Math.max(leftMax, rightMax) + 1;

	return res;
}
```

**那么我们再回头看看最基本的二叉树前中后序遍历**

```java
List<Integer> res = new LinkedList<>();

// 返回前序遍历结果
List<Integer> preorderTraverse(TreeNode root) {
    traverse(root);
    return res;
}

// 二叉树遍历函数
void traverse(TreeNode root) {
    if (root == null) {
        return;
    }
    // 前序位置
    res.add(root.val);
    traverse(root.left);
    traverse(root.right);
}
```

<img src="https://s2.loli.net/2024/01/23/qKjh4bZrOkszfGQ.jpg" alt="img" style="zoom: 33%;" />

```java
// 定义：输入一棵二叉树的根节点，返回这棵树的前序遍历结果
List<Integer> preorderTraverse(TreeNode root) {
    List<Integer> res = new LinkedList<>();
    if (root == null) {
        return res;
    }
    // 前序遍历的结果，root.val 在第一个
    res.add(root.val);
    // 利用函数定义，后面接着左子树的前序遍历结果
    res.addAll(preorderTraverse(root.left));
    // 利用函数定义，最后接着右子树的前序遍历结果
    res.addAll(preorderTraverse(root.right));
    return res;
}
```

<img src="https://s2.loli.net/2024/01/23/vVLtoFMH57JWwk4.jpg" alt="img" style="zoom:33%;" />

1、如果把根节点看做第 1 层，如何打印出每一个节点所在的层数？

```java
// 二叉树遍历函数
void traverse(TreeNode root, int level) {
    if (root == null) {
        return;
    }
    // 前序位置
    printf("节点 %s 在第 %d 层", root, level);
    traverse(root.left, level + 1);
    traverse(root.right, level + 1);
}

// 这样调用
traverse(root, 1);
```

2、如何打印出每个节点的左右子树各有多少节点？

```java
// 定义：输入一棵二叉树，返回这棵二叉树的节点总数
int count(TreeNode root) {
    if (root == null) {
        return 0;
    }
    int leftCount = count(root.left);
    int rightCount = count(root.right);
    // 后序位置
    printf("节点 %s 的左子树有 %d 个节点，右子树有 %d 个节点",
            root, leftCount, rightCount);

    return leftCount + rightCount + 1;
}
```

**这两个问题的根本区别在于：一个节点在第几层，你从根节点遍历过来的过程就能顺带记录，用递归函数的参数就能传递下去；而以一个节点为根的整棵子树有多少个节点，你需要遍历完子树之后才能数清楚，然后通过递归函数的返回值拿到答案**。

<img src="https://s2.loli.net/2024/01/23/xruY4sNUF9nWJ5V.png" alt="img" style="zoom: 50%;" />

```java
class Solution {
    // 记录最大直径的长度
    int maxDiameter = 0;

    public int diameterOfBinaryTree(TreeNode root) {
        maxDepth(root);
        return maxDiameter;
    }

    int maxDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }
        int leftMax = maxDepth(root.left);
        int rightMax = maxDepth(root.right);
        // 后序位置，顺便计算最大直径
        int myDiameter = leftMax + rightMax;
        maxDiameter = Math.max(maxDiameter, myDiameter);

        return 1 + Math.max(leftMax, rightMax);
    }
}
```

以树的视角看动归/回溯/DFS算法的区别和联系
======

**动归/DFS/回溯算法都可以看做二叉树问题的扩展，只是它们的关注点不同**：

- **动态规划算法属于分解问题的思路，它的关注点在整棵「子树」**。
- **回溯算法属于遍历的思路，它的关注点在节点间的「树枝」**。
- **DFS 算法属于遍历的思路，它的关注点在单个「节点」**。

**第一个例子**，动态规划算法属于分解问题的思路，它的关注点在整棵「子树」。

```java
// 定义：输入一棵二叉树，返回这棵二叉树的节点总数
int count(TreeNode root) {
    if (root == null) {
        return 0;
    }
    // 我这个节点关心的是我的两个子树的节点总数分别是多少
    int leftCount = count(root.left);
    int rightCount = count(root.right);
    // 后序位置，左右子树节点数加上自己就是整棵树的节点数
    return leftCount + rightCount + 1;
}
```

```java
int fib(int N) {
    if (N == 1 || N == 2) return 1;
    return fib(N - 1) + fib(N - 2);
}
```

<img src="https://s2.loli.net/2024/01/23/njTJ29IxYwVr61P.jpg" alt="img" style="zoom:33%;" />

**第二个例子**，回溯算法属于遍历的思路，它的关注点在节点间的「树枝」。

```java
void traverse(TreeNode root) {
    if (root == null) return;
    printf("从节点 %s 进入节点 %s", root, root.left);
    traverse(root.left);
    printf("从节点 %s 回到节点 %s", root.left, root);

    printf("从节点 %s 进入节点 %s", root, root.right);
    traverse(root.right);
    printf("从节点 %s 回到节点 %s", root.right, root);
}
```

```java
// 多叉树节点
class Node {
    int val;
    Node[] children;
}

void traverse(Node root) {
    if (root == null) return;
    for (Node child : root.children) {
        printf("从节点 %s 进入节点 %s", root, child);
        traverse(child);
        printf("从节点 %s 回到节点 %s", child, root);
    }
}
```

```java
void backtrack(...) {
    for (int i = 0; i < ...; i++) {
        // 做选择
        ...

        // 进入下一层决策树
        backtrack(...);

        // 撤销刚才做的选择
        ...
    }
}
```

 [回溯算法秒杀排列组合子集的九种题型](https://labuladong.gitee.io/algo/di-ling-zh-bfe1b/hui-su-sua-56e11/) 中讲到的全排列，我们的关注点在一条条树枝上：

```java
// 回溯算法核心部分代码
void backtrack(int[] nums) {
    // 回溯算法框架
    for (int i = 0; i < nums.length; i++) {
        // 做选择
        used[i] = true;
        track.addLast(nums[i]);

        // 进入下一层回溯树
        backtrack(nums);

        // 取消选择
        track.removeLast();
        used[i] = false;
    }
}
```

<img src="https://s2.loli.net/2024/01/23/eAb546DycwrgmT7.jpg" alt="img" style="zoom:33%;" />

**第三个例子**，DFS 算法属于遍历的思路，它的关注点在单个「节点」。

```java
void traverse(TreeNode root) {
    if (root == null) return;
    // 遍历过的每个节点的值加一
    root.val++;
    traverse(root.left);
    traverse(root.right);
}
```

```java
// DFS 算法核心逻辑
void dfs(int[][] grid, int i, int j) {
    int m = grid.length, n = grid[0].length;
    if (i < 0 || j < 0 || i >= m || j >= n) {
        return;
    }
    if (grid[i][j] == 0) {
        return;
    }
    // 遍历过的每个格子标记为 0
    grid[i][j] = 0;
    dfs(grid, i + 1, j);
    dfs(grid, i, j + 1);
    dfs(grid, i - 1, j);
    dfs(grid, i, j - 1);
}
```

<img src="https://s2.loli.net/2024/01/23/oVTxq5WnadYEQOU.png" alt="image-20240123232656963" style="zoom:33%;" />

好，请你仔细品一下上面三个简单的例子，是不是像我说的：动态规划关注整棵「子树」，回溯算法关注节点间的「树枝」，DFS 算法关注单个「节点」。

```java
// DFS 算法把「做选择」「撤销选择」的逻辑放在 for 循环外面
void dfs(Node root) {
    if (root == null) return;
    // 做选择
    print("我已经进入节点 %s 啦", root)
    for (Node child : root.children) {
        dfs(child);
    }
    // 撤销选择
    print("我将要离开节点 %s 啦", root)
}

// 回溯算法把「做选择」「撤销选择」的逻辑放在 for 循环里面
void backtrack(Node root) {
    if (root == null) return;
    for (Node child : root.children) {
        // 做选择
        print("我站在节点 %s 到节点 %s 的树枝上", root, child)
        backtrack(child);
        // 撤销选择
        print("我将要离开节点 %s 到节点 %s 的树枝上", child, root)
    }
}
```

层序遍历
======

```java
// 输入一棵二叉树的根节点，层序遍历这棵二叉树
void levelTraverse(TreeNode root) {
    if (root == null) return;
    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root);

    // 从上到下遍历二叉树的每一层
    while (!q.isEmpty()) {
        int sz = q.size();
        // 从左到右遍历每一层的每个节点
        for (int i = 0; i < sz; i++) {
            TreeNode cur = q.poll();
            // 将下一层节点放入队列
            if (cur.left != null) {
                q.offer(cur.left);
            }
            if (cur.right != null) {
                q.offer(cur.right);
            }
        }
    }
}
```

<img src="https://s2.loli.net/2024/01/23/Rb1azqfkGei9QgO.jpg" alt="img" style="zoom:33%;" />

关于层序遍历（以及其扩展出的 [BFS 算法框架](https://labuladong.gitee.io/algo/di-ling-zh-bfe1b/bfs-suan-f-463fd/)），我在最后多说几句。

如果你对二叉树足够熟悉，可以想到很多方式通过递归函数得到层序遍历结果，比如下面这种写法：

```java
class Solution {
    List<List<Integer>> res = new ArrayList<>();

    List<List<Integer>> levelTraverse(TreeNode root) {
        if (root == null) {
            return res;
        }
        // root 视为第 0 层
        traverse(root, 0);
        return res;
    }

    void traverse(TreeNode root, int depth) {
        if (root == null) {
            return;
        }
        // 前序位置，看看是否已经存储 depth 层的节点了
        if (res.size() <= depth) {
            // 第一次进入 depth 层
            res.add(new LinkedList<>());
        }
        // 前序位置，在 depth 层添加 root 节点的值
        res.get(depth).add(root.val);
        traverse(root.left, depth + 1);
        traverse(root.right, depth + 1);
    }
}
```

这种思路从结果上说确实可以得到层序遍历结果，但其本质还是二叉树的前序遍历，或者说 DFS 的思路，而不是层序遍历，或者说 BFS 的思路。因为这个解法是依赖前序遍历自顶向下、自左向右的顺序特点得到了正确的结果。

**抽象点说，这个解法更像是从左到右的「列序遍历」，而不是自顶向下的「层序遍历」**。所以对于计算最小距离的场景，这个解法完全等同于 DFS 算法，没有 BFS 算法的性能的优势。

还有优秀读者评论了这样一种递归进行层序遍历的思路：

```java
class Solution {

    List<List<Integer>> res = new LinkedList<>();

    List<List<Integer>> levelTraverse(TreeNode root) {
        if (root == null) {
            return res;
        }
        List<TreeNode> nodes = new LinkedList<>();
        nodes.add(root);
        traverse(nodes);
        return res;
    }

    void traverse(List<TreeNode> curLevelNodes) {
        // base case
        if (curLevelNodes.isEmpty()) {
            return;
        }
        // 前序位置，计算当前层的值和下一层的节点列表
        List<Integer> nodeValues = new LinkedList<>();
        List<TreeNode> nextLevelNodes = new LinkedList<>();
        for (TreeNode node : curLevelNodes) {
            nodeValues.add(node.val);
            if (node.left != null) {
                nextLevelNodes.add(node.left);
            }
            if (node.right != null) {
                nextLevelNodes.add(node.right);
            }
        }
        // 前序位置添加结果，可以得到自顶向下的层序遍历
        res.add(nodeValues);
        traverse(nextLevelNodes);
        // 后序位置添加结果，可以得到自底向上的层序遍历结果
        // res.add(nodeValues);
    }
}
```

|                             力扣                             |      |
| :----------------------------------------------------------: | ---- |
| [100. 相同的树](https://leetcode.cn/problems/same-tree/?show=1) |      |
| [1008. 前序遍历构造二叉搜索树](https://leetcode.cn/problems/construct-binary-search-tree-from-preorder-traversal/?show=1) |      |
| [101. 对称二叉树](https://leetcode.cn/problems/symmetric-tree/?show=1) |      |
| [1022. 从根到叶的二进制数之和](https://leetcode.cn/problems/sum-of-root-to-leaf-binary-numbers/?show=1) |      |
| [1026. 节点与其祖先之间的最大差值](https://leetcode.cn/problems/maximum-difference-between-node-and-ancestor/?show=1) |      |
| [108. 将有序数组转换为二叉搜索树](https://leetcode.cn/problems/convert-sorted-array-to-binary-search-tree/?show=1) |      |
| [1080. 根到叶路径上的不足节点](https://leetcode.cn/problems/insufficient-nodes-in-root-to-leaf-paths/?show=1) |      |
| [110. 平衡二叉树](https://leetcode.cn/problems/balanced-binary-tree/?show=1) |      |
| [111. 二叉树的最小深度](https://leetcode.cn/problems/minimum-depth-of-binary-tree/?show=1) |      |
| [1110. 删点成林](https://leetcode.cn/problems/delete-nodes-and-return-forest/?show=1) |      |
| [1120. 子树的最大平均值](https://leetcode.cn/problems/maximum-average-subtree/?show=1)🔒 |      |
| [113. 路径总和 II](https://leetcode.cn/problems/path-sum-ii/?show=1) |      |
| [114. 二叉树展开为链表](https://leetcode.cn/problems/flatten-binary-tree-to-linked-list/?show=1) |      |
| [116. 填充每个节点的下一个右侧节点指针](https://leetcode.cn/problems/populating-next-right-pointers-in-each-node/?show=1) |      |
| [124. 二叉树中的最大路径和](https://leetcode.cn/problems/binary-tree-maximum-path-sum/?show=1) |      |
| [1245. 树的直径](https://leetcode.cn/problems/tree-diameter/?show=1)🔒 |      |
| [1261. 在受污染的二叉树中查找元素](https://leetcode.cn/problems/find-elements-in-a-contaminated-binary-tree/?show=1) |      |
| [129. 求根节点到叶节点数字之和](https://leetcode.cn/problems/sum-root-to-leaf-numbers/?show=1) |      |
| [1315. 祖父节点值为偶数的节点和](https://leetcode.cn/problems/sum-of-nodes-with-even-valued-grandparent/?show=1) |      |
| [1325. 删除给定值的叶子节点](https://leetcode.cn/problems/delete-leaves-with-a-given-value/?show=1) |      |
| [1339. 分裂二叉树的最大乘积](https://leetcode.cn/problems/maximum-product-of-splitted-binary-tree/?show=1) |      |
| [1367. 二叉树中的链表](https://leetcode.cn/problems/linked-list-in-binary-tree/?show=1) |      |
| [1372. 二叉树中的最长交错路径](https://leetcode.cn/problems/longest-zigzag-path-in-a-binary-tree/?show=1) |      |
| [1373. 二叉搜索子树的最大键值和](https://leetcode.cn/problems/maximum-sum-bst-in-binary-tree/?show=1) |      |
| [1376. 通知所有员工所需的时间](https://leetcode.cn/problems/time-needed-to-inform-all-employees/?show=1) |      |
| [1379. 找出克隆二叉树中的相同节点](https://leetcode.cn/problems/find-a-corresponding-node-of-a-binary-tree-in-a-clone-of-that-tree/?show=1) |      |
| [1430. 判断给定的序列是否是二叉树从根到叶的路径](https://leetcode.cn/problems/check-if-a-string-is-a-valid-sequence-from-root-to-leaves-path-in-a-binary-tree/?show=1)🔒 |      |
| [1443. 收集树上所有苹果的最少时间](https://leetcode.cn/problems/minimum-time-to-collect-all-apples-in-a-tree/?show=1) |      |
| [1448. 统计二叉树中好节点的数目](https://leetcode.cn/problems/count-good-nodes-in-binary-tree/?show=1) |      |
| [145. 二叉树的后序遍历](https://leetcode.cn/problems/binary-tree-postorder-traversal/?show=1) |      |
| [1457. 二叉树中的伪回文路径](https://leetcode.cn/problems/pseudo-palindromic-paths-in-a-binary-tree/?show=1) |      |
| [1469. 寻找所有的独生节点](https://leetcode.cn/problems/find-all-the-lonely-nodes/?show=1)🔒 |      |
| [1485. 克隆含随机指针的二叉树](https://leetcode.cn/problems/clone-binary-tree-with-random-pointer/?show=1)🔒 |      |
| [1490. 克隆 N 叉树](https://leetcode.cn/problems/clone-n-ary-tree/?show=1)🔒 |      |
| [1602. 找到二叉树中最近的右侧节点](https://leetcode.cn/problems/find-nearest-right-node-in-binary-tree/?show=1)🔒 |      |
| [1612. 检查两棵二叉表达式树是否等价](https://leetcode.cn/problems/check-if-two-expression-trees-are-equivalent/?show=1)🔒 |      |
| [1740. 找到二叉树中的距离](https://leetcode.cn/problems/find-distance-in-a-binary-tree/?show=1)🔒 |      |
| [2049. 统计最高分的节点数目](https://leetcode.cn/problems/count-nodes-with-the-highest-score/?show=1) |      |
| [226. 翻转二叉树](https://leetcode.cn/problems/invert-binary-tree/?show=1) |      |
| [250. 统计同值子树](https://leetcode.cn/problems/count-univalue-subtrees/?show=1)🔒 |      |
| [254. 因子的组合](https://leetcode.cn/problems/factor-combinations/?show=1)🔒 |      |
| [257. 二叉树的所有路径](https://leetcode.cn/problems/binary-tree-paths/?show=1) |      |
| [270. 最接近的二叉搜索树值](https://leetcode.cn/problems/closest-binary-search-tree-value/?show=1)🔒 |      |
| [298. 二叉树最长连续序列](https://leetcode.cn/problems/binary-tree-longest-consecutive-sequence/?show=1)🔒 |      |
| [301. 删除无效的括号](https://leetcode.cn/problems/remove-invalid-parentheses/?show=1) |      |
| [332. 重新安排行程](https://leetcode.cn/problems/reconstruct-itinerary/?show=1) |      |
| [333. 最大 BST 子树](https://leetcode.cn/problems/largest-bst-subtree/?show=1)🔒 |      |
| [366. 寻找二叉树的叶子节点](https://leetcode.cn/problems/find-leaves-of-binary-tree/?show=1)🔒 |      |
| [404. 左叶子之和](https://leetcode.cn/problems/sum-of-left-leaves/?show=1) |      |
| [426. 将二叉搜索树转化为排序的双向链表](https://leetcode.cn/problems/convert-binary-search-tree-to-sorted-doubly-linked-list/?show=1)🔒 |      |
| [437. 路径总和 III](https://leetcode.cn/problems/path-sum-iii/?show=1) |      |
| [501. 二叉搜索树中的众数](https://leetcode.cn/problems/find-mode-in-binary-search-tree/?show=1) |      |
| [508. 出现次数最多的子树元素和](https://leetcode.cn/problems/most-frequent-subtree-sum/?show=1) |      |
| [513. 找树左下角的值](https://leetcode.cn/problems/find-bottom-left-tree-value/?show=1) |      |
| [515. 在每个树行中找最大值](https://leetcode.cn/problems/find-largest-value-in-each-tree-row/?show=1) |      |
| [530. 二叉搜索树的最小绝对差](https://leetcode.cn/problems/minimum-absolute-difference-in-bst/?show=1) |      |
| [538. 把二叉搜索树转换为累加树](https://leetcode.cn/problems/convert-bst-to-greater-tree/?show=1) |      |
| [549. 二叉树中最长的连续序列](https://leetcode.cn/problems/binary-tree-longest-consecutive-sequence-ii/?show=1)🔒 |      |
| [559. N 叉树的最大深度](https://leetcode.cn/problems/maximum-depth-of-n-ary-tree/?show=1) |      |
| [563. 二叉树的坡度](https://leetcode.cn/problems/binary-tree-tilt/?show=1) |      |
| [572. 另一棵树的子树](https://leetcode.cn/problems/subtree-of-another-tree/?show=1) |      |
| [582. 杀掉进程](https://leetcode.cn/problems/kill-process/?show=1)🔒 |      |
| [606. 根据二叉树创建字符串](https://leetcode.cn/problems/construct-string-from-binary-tree/?show=1) |      |
| [617. 合并二叉树](https://leetcode.cn/problems/merge-two-binary-trees/?show=1) |      |
| [623. 在二叉树中增加一行](https://leetcode.cn/problems/add-one-row-to-tree/?show=1) |      |
| [654. 最大二叉树](https://leetcode.cn/problems/maximum-binary-tree/?show=1) |      |
| [663. 均匀树划分](https://leetcode.cn/problems/equal-tree-partition/?show=1)🔒 |      |
| [666. 路径总和 IV](https://leetcode.cn/problems/path-sum-iv/?show=1)🔒 |      |
| [669. 修剪二叉搜索树](https://leetcode.cn/problems/trim-a-binary-search-tree/?show=1) |      |
| [671. 二叉树中第二小的节点](https://leetcode.cn/problems/second-minimum-node-in-a-binary-tree/?show=1) |      |
| [687. 最长同值路径](https://leetcode.cn/problems/longest-univalue-path/?show=1) |      |
| [776. 拆分二叉搜索树](https://leetcode.cn/problems/split-bst/?show=1)🔒 |      |
| [865. 具有所有最深节点的最小子树](https://leetcode.cn/problems/smallest-subtree-with-all-the-deepest-nodes/?show=1) |      |
| [894. 所有可能的真二叉树](https://leetcode.cn/problems/all-possible-full-binary-trees/?show=1) |      |
| [897. 递增顺序搜索树](https://leetcode.cn/problems/increasing-order-search-tree/?show=1) |      |
| [938. 二叉搜索树的范围和](https://leetcode.cn/problems/range-sum-of-bst/?show=1) |      |
| [951. 翻转等价二叉树](https://leetcode.cn/problems/flip-equivalent-binary-trees/?show=1) |      |
| [965. 单值二叉树](https://leetcode.cn/problems/univalued-binary-tree/?show=1) |      |
| [968. 监控二叉树](https://leetcode.cn/problems/binary-tree-cameras/?show=1) |      |
| [971. 翻转二叉树以匹配先序遍历](https://leetcode.cn/problems/flip-binary-tree-to-match-preorder-traversal/?show=1) |      |
| [979. 在二叉树中分配硬币](https://leetcode.cn/problems/distribute-coins-in-binary-tree/?show=1) |      |
| [987. 二叉树的垂序遍历](https://leetcode.cn/problems/vertical-order-traversal-of-a-binary-tree/?show=1) |      |
| [988. 从叶结点开始的最小字符串](https://leetcode.cn/problems/smallest-string-starting-from-leaf/?show=1) |      |
| [99. 恢复二叉搜索树](https://leetcode.cn/problems/recover-binary-search-tree/?show=1) |      |
| [993. 二叉树的堂兄弟节点](https://leetcode.cn/problems/cousins-in-binary-tree/?show=1) |      |
| [998. 最大二叉树 II](https://leetcode.cn/problems/maximum-binary-tree-ii/?show=1) |      |
| [剑指 Offer 06. 从尾到头打印链表](https://leetcode.cn/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/?show=1) |      |
| [剑指 Offer 26. 树的子结构](https://leetcode.cn/problems/shu-de-zi-jie-gou-lcof/?show=1) |      |
| [剑指 Offer 27. 二叉树的镜像](https://leetcode.cn/problems/er-cha-shu-de-jing-xiang-lcof/?show=1) |      |
| [剑指 Offer 28. 对称的二叉树](https://leetcode.cn/problems/dui-cheng-de-er-cha-shu-lcof/?show=1) |      |
| [剑指 Offer 33. 二叉搜索树的后序遍历序列](https://leetcode.cn/problems/er-cha-sou-suo-shu-de-hou-xu-bian-li-xu-lie-lcof/?show=1) |      |
| [剑指 Offer 34. 二叉树中和为某一值的路径](https://leetcode.cn/problems/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-jing-lcof/?show=1) |      |
| [剑指 Offer 36. 二叉搜索树与双向链表](https://leetcode.cn/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/?show=1) |      |
| [剑指 Offer 55 - I. 二叉树的深度](https://leetcode.cn/problems/er-cha-shu-de-shen-du-lcof/?show=1) |      |
| [剑指 Offer 55 - II. 平衡二叉树](https://leetcode.cn/problems/ping-heng-er-cha-shu-lcof/?show=1) |      |
| [剑指 Offer II 044. 二叉树每层的最大值](https://leetcode.cn/problems/hPov7L/?show=1) |      |
| [剑指 Offer II 045. 二叉树最底层最左边的值](https://leetcode.cn/problems/LwUNpT/?show=1) |      |
| [剑指 Offer II 049. 从根节点到叶节点的路径数字之和](https://leetcode.cn/problems/3Etpl5/?show=1) |      |
| [剑指 Offer II 050. 向下的路径节点之和](https://leetcode.cn/problems/6eUYwP/?show=1) |      |
| [剑指 Offer II 051. 节点之和最大的路径](https://leetcode.cn/problems/jC7MId/?show=1) |      |
| [剑指 Offer II 052. 展平二叉搜索树](https://leetcode.cn/problems/NYBBNL/?show=1) |      |
| [剑指 Offer II 054. 所有大于等于节点的值之和](https://leetcode.cn/problems/w6cpku/?show=1) |      |
