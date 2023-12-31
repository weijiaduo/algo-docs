#  二叉搜索树

## 一、结构定义

二叉搜索树（binary search tree），也称为二叉查找树，是一种二叉树，同时还满足了以下的条件：

1. 对于根节点，左子树中所有节点的值 < 根节点的值 < 右子树中所有节点的值
2. 任意节点的左、右子树也是二叉搜索树

相比于普通二叉树，二叉搜索树的节点之间带有大小关系，这使得二叉搜索树可以支持快速的查找、插入、删除操作。

<!--more-->

比如说，这就是一棵二叉搜索树：

```
    5
   / \
  3   6
 / \   \
2   4   7
```


## 二、基本操作

### 2.1 遍历

二叉搜索树满足「左子节点 < 根节点 < 右子节点」的大小关系，而中序遍历恰好是按照「左 -> 根 -> 右」的顺序遍历：

```
    5
   / \
  3   6        中序遍历：2 3 4 5 6 7
 / \   \
2   4   7
```

因此，对二叉搜索树进行中序遍历的话，可以得到一个递增的有序序列。

利用中序遍历升序的性质，可以用二叉搜索树来实现排序：先将数据插入到二叉搜索树中，再进行中序遍历，就可以得到有序的数据序列。

### 2.2 查找

二叉搜索树满足「左子节点 < 根节点 < 右子节点」的大小关系，因此查找操作与二分查找类似，每次可以将查找区间缩小一半。

比如说，要查找值为 4 的节点：

```
二叉搜索树                 查找 4

    5                       5         -- 第一轮
   / \                     /
  3   6         -->       3           -- 第二轮
 / \   \                   \
2   4   7                   4         -- 第三轮
```

查找过程就是，比较当前节点的值与目标值，根据判断结果决定向左子树还是右子树查找。

查询过程的伪代码类似这样：

```java
TreeNode search(TreeNode root, int val) {
    if (root == null || root.val == val) {
        return root;
    }
    if (root.val < val) {
        return search(root.right, val);
    } else {
        return search(root.left, val);
    }
}
```

复杂度分析：

- 时间复杂度：O(logn)，查找次数最多是树的高度，平衡的二叉搜索树高度是 logn。
- 空间复杂度：O(1)。

### 2.3 插入

插入操作需要满足几个条件：

1. 插入后必须能保持二叉搜索树「左子树 < 根节点 < 右子树」的性质
2. 新插入节点必须是叶子节点

比如说，要插入值为 1 的节点：

```
二叉搜索树                 插入 1

    5                       5
   / \                     / \
  3   6         -->       3   6
 / \   \                 / \   \
2   4   7               2   4   7
                       /
                      1
```

插入的新节点必须是放在叶子节点，因此首先要找到插入位置，找的过程和查找操作类似。

插入过程的伪代码类似这样：

```java
TreeNode insert(TreeNode root, int val) {
    if (root == null) {
        return new TreeNode(val);
    }
    if (root.val < val) {
        root.right = insert(root.right, val);
    } else {
        root.left = insert(root.left, val);
    }
    return root;
}
```

复杂度分析：

- 时间复杂度：O(logn)，插入需要查找，与查找操作的复杂度相同。
- 空间复杂度：O(1)。

### 2.4 删除

删除操作也需要满足：删除后必须能保持二叉搜索树「左子树 < 根节点 < 右子树」的性质。

因此，删除需要分为 3 种情况：

1. 要删除的节点是叶子节点，可以直接删除
2. 要删除的节点只有一个子节点，那么将子节点提升到要删除的节点的位置即可
3. 要删除的节点有两个子节点，需要找到左子树中最大的节点，或者右子树中最小的节点来接替要删除的节点

#### (1) 删除叶子节点

比如说，要删除值为 4 的节点：

```
二叉搜索树                 删除 4

    5                       5
   / \                     / \
  3   6         -->       3   6
 / \   \                 /     \
2   4   7               2       7
```

删除叶子节点的过程就是：

1. 找到要删除的节点（4）。
2. 然后将父节点（3）指向它（4）的指针置为 null。

#### (2) 删除只有一个子节点的节点

比如说，要删除值为 6 的节点：

```
二叉搜索树                 删除 6

    5                       5
   / \                     / \
  3   6         -->       3   7
 / \   \                 / \
2   4   7               2   4  
```

删除只有一个子节点的节点的过程就是：

1. 找到要删除的节点（6）。
2. 然后将父节点（5）指向它（6）的指针指向它的子节点（7）。

#### (3) 删除有两个子节点的节点

比如说，要删除值为 5 的节点：

```
二叉搜索树                删除 5（替换节点4）         删除 5（替换节点6）

    5                          4                          6    
   / \                        / \                        / \  
  3   6         -->          3   6          或          3   7  
 / \   \                    /     \                    / \   
2   4   7                  2       7                  2   4  
```

删除有两个子节点的节点的过程就是：

1. 找到要删除的节点（5）。
2. 然后找到删除节点左子树中最大的节点（4），或者右子树中最小的节点（6），作为替换节点。
3. 将替换节点（4/6）与要删除的节点（5）交换，然后删除要删除的节点（5）。

删除过程的伪代码类似这样（以右子树中最小的节点作为替换节点）：

```java
TreeNode delete(TreeNode root, int val) {
    if (root == null) {
        return null;
    }
    if (root.val == val) {
        // 找到要删除的节点
        // 1. 删除叶子节点
        if (root.left == null && root.right == null) {
            return null;
        }

        // 2. 删除只有一个子节点的节点
        if (root.left == null) {
            return root.right;
        }
        if (root.right == null) {
            return root.left;
        }
        
        // 3. 删除有两个子节点的节点
        // 找到右子树中最小的节点，作为替换节点
        TreeNode minNode = getMin(root.right);
        // 删除 minNode（等价于交换 root 和 minNode，并删除 root）
        root.right = delete(root.right, minNode.val);
        // 用 minNode 替换 root
        minNode.left = root.left;
        minNode.right = root.right;
        root = minNode;
    } else if (root.val < val) {
        // 去右子树删除
        root.right = delete(root.right, val);
    } else if (root.val > val) {
        // 去左子树删除
        root.left = delete(root.left, val);
    }
    return root;
}
```

因为删除涉及到了二叉搜索树的内部节点，为了保持它的性质，所以操作会稍微有点复杂。

复杂度分析：

- 时间复杂度：O(logn)，删除需要查找，与查找操作的复杂度相同。
- 空间复杂度：O(1)。


## 三、典型应用

- 索引：用作系统中的多级索引，实现高效的查找、插入、删除操作。
- 搜索算法：作为某些搜索算法的底层数据结构。
- 有序数据存储：用于存储数据流，以保持其有序状态。
- 范围查询：BST可用于执行范围查询，例如查找在给定范围内的所有元素。
- 自动补全和拼写检查：BST可用于实现自动补全和拼写检查功能，其中树中的节点存储单词或短语。


## 参考

> https://www.hello-algo.com/chapter_tree/binary_search_tree/
>
> https://time.geekbang.org/column/article/68334
