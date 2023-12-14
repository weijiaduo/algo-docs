---
title: 左倾堆
date: 2023-09-27 19:22:30
categories: [数据结构与算法, 数据结构]
tags: [数据结构, 算法, 堆, 左倾堆]
thumbnail: /img/structure.jpg
top: true
---

# 左倾堆

## 一、什么是左倾堆？

左倾堆（Leftist Heap），也称为左偏树（Leftist Tree）、左偏堆，最左堆等。

和二叉堆（即常见的堆，基于数组实现的完全堆）一样，左倾堆也是优先队列的一种实现方式。

只不过左倾堆具有一些特殊的性质：

<!--more-->

1. 左倾性：左倾堆中左子树的深度（或高度）都不小于右子树的深度
2. 最小堆性：左倾堆通常用于实现最小堆，所以根节点通常都是最小值

为了实现左倾性，左倾堆的节点都有一个 NPL（Null Path Length）属性：

- NPL：节点到一个“最近的不满节点”的最短路径的长度。空节点的 NPL = -1

其中，“不满节点”是指左右子节点中至少有一个为 null 的节点。

而左倾堆的结构中，也有几个基本性质：

1. 节点的键值 <= 它的左右子节点的键值
2. 节点的左孩子的 NPL >= 节点右孩子的 NPL
3. 节点的 NPL = 它的右孩子的 NPL + 1

下面是左倾堆的一个例子：

```
        2(1)
       /    \
    3(1)     4(0)
   /    \
5(0)    7(0)
       /     
    9(0)
```

其中，括号中的数字表示 NPL。

像 4，5，7，9 这几个节点都有 null 子节点，所以它们的 NPL 都是 0。

而 2，3 的 NPL 则等于它们右孩子的 NPL + 1。

另外，有一点需要说明的是：根节点的 NPL 不一定比子节点的大。

## 二、为什么用左倾堆？

左倾堆和二叉堆的作用是一样的，只是在结构上有所区别：

- 二叉堆是用数组实现，堆结构是一棵完全二叉树
- 左倾堆是用树实现，结构就是一棵普通的二叉树

二叉堆和左倾堆都能用于优先队列的实现，区别不是很大。

- 但是在涉及“合并操作”时，二叉堆的效率会比较低，而左倾堆的性能更好
- 而左倾堆也有自己的缺点，就是在删除最小元素操作上可能表现略差一些

所以，左倾堆适用于某些特殊场景下，特别是需要频繁执行合并操作的情况。


## 三、怎么实现左倾堆？

合并操作是左倾堆的重点，合并两个堆的步骤如下：

1. 如果两个堆中有一个为空，则直接返回另一个堆
2. 如果两个堆都不为空，则比较两个堆的根节点，将较小的根节点作为新堆的根节点
3. 将较小根节点的右子树和较大根节点合并，然后将合并后的树作为新堆的右子树
4. 如果新堆的右子树的 NPL > 左子树的 NPL，则交换左右子树
5. 新堆的 NPL = 右子树的 NPL + 1

步骤看起来挺多的，但是其实逻辑很简单，伪代码如下：

```java
merge(h1, h2) {
    // 1. 返回非空堆
    if (h1 == null) {
        return h2;
    }
    if (h2 == null) {
        return h1;
    }

    // 2. 以小根节点作为新堆的根节点
    min = h1.val < h2.val ? h1 : h2;
    max = h1.val > h2.val ? h1 : h2;

    // 3. 小根节点的右子节点和大根节点合并
    min.right = merge(min.right, max);

    // 4. 交换左右子节点
    if (npl(min.right) > npl(min.left)) {
        swap(min.left, min.right);
    }

    // 5. 根节点距离 = 右子节点距离 + 1
    min.npl = npl(min.right) + 1;

    return min;
}
```

下面给出一个合并操作的例子：

```
        堆1                     堆2

        2(1)                   1(1)
       /    \                 /    \
    6(0)    4(0)           3(0)    5(0)
```

1）首先，取小根节点 1(1) 作为新堆的根节点，然后合并小根右孩子与大根堆

```
        新堆1                          待合并
           
        1(1)                      5(0)     2(1)     
       /                                  /    \
    3(0)                               6(0)    4(0)
```

2）同样地，取小根节点 2(1) 作为新堆的根节点，合并小根右孩子与大根堆

```
        新堆1       新堆2              待合并
                                     
        1(1)        2(1)           5(0)    4(0)
       /           /  
    3(0)         6(0)
```

3）同上，取小根节点 4(0) 作为新堆根节点，合并小根右孩子（null 节点）与大根堆


```
        新堆1       新堆2        新堆3
                                     
        1(1)        2(1)        4(0)
       /           /               \
    3(0)         6(0)              5(0)
```

4）此时 `4` 的右孩子 `5(0)` 大于左孩子 `NULL(-1)` 的 NPL，需要交换左右孩子

```
        新堆1       新堆2        新堆3
                                     
        1(1)        2(1)        4(0)
       /           /           /
    3(0)         6(0)        5(0)
```

同时更新 `4` 的 NPL = NPL(null) + 1 = -1 + 1 = 0。

5）接着，完成后开始递归往上合并，4 返回给 2 的右孩子

```
        新堆1       新堆2
                                     
        1(1)        2(1)
       /           /    \
    3(0)         6(0)   4(0)
                       /
                    5(0)
```

由于 `4(0)` 并不比 `6(0)` 的 NPL 大，所以不需要交换左右子节点。

同时更新 `2` 的 NPL = NPL(4) + 1 = 0 + 1 = 1。

6）继续合并完成后开始递归往上，2 返回给 1 的右孩子

```
        新堆
                                     
        1(1)
       /    \
    3(0)    2(1)
           /    \
        6(0)    4(0)
               /
            5(0)
```

7）由于 `2(1)` 比 `3(0)` 的 NPL 大，所以需要交换左右子节点

```
        新堆
                                     
        1(1)
       /    \
    2(1)    3(0)
   /    \
6(0)    4(0)
       /
    5(0)
```

同时更新 `1` 的 NPL = NPL(3) + 1 = 0 + 1 = 1。

至此，合并操作就完成了。

至于其他的操作，比如删除和插入，实际上都是通过合并操作实现的：

1. 删除最小元素：相当于删除根节点，然后合并左右子树
2. 插入新元素：相当于将新元素作为一个新的左倾堆，然后两个堆合并

下面是它们的伪代码：

```java
removeFirst() {
    temp = root;
    root = merge(root.left, root.right);
    temp.left = temp.right = null;
    return temp.val;
}
```
```java
insert(val) {
    root = merge(root, new Node<>(val));
}
```

总体来说，左倾堆的实现还是比较简单的。


## 参考

> https://www.cnblogs.com/skywang12345/p/3638384.html

## 附录

### 左倾堆接口

```java
/**
 * 左倾堆（Leftist Heap）
 * <p>
 * 基本性质：
 * <p>
 * 1. 节点的键值小于或等于它的左右子节点的键值
 * <p>
 * 2. 节点的左孩子的NPL >= 右孩子的NPL
 * <p>
 * 3. 节点的NPL = 它的右孩子的NPL + 1
 *
 * @author weijiaduo
 * @since 2023/9/26
 */
public interface LeftistHeap<T> {

    /**
     * 合并另外一个堆
     *
     * @param other 另外一个堆
     */
    void merge(LeftistHeap<T> other);

    /**
     * 插入新值
     *
     * @param val 新值
     */
    void insert(T val);

    /**
     * 移除根节点
     *
     * @return 根节点值
     */
    T removeFirst();

    /**
     * 根节点值
     *
     * @return 根节点值
     */
    T first();

    /**
     * 堆大小
     *
     * @return 堆大小
     */
    int size();

    /**
     * 是否为空
     *
     * @return true/false
     */
    boolean isEmpty();

}
```

### 左倾堆实现

```java
/**
 * 左倾堆节点
 *
 * @author weijiaduo
 * @since 2023/9/26
 */
public class LeftistHeapNode<T> {

    /**
     * 节点值
     */
    public T val;
    /**
     * 零距离（Null Path Length）
     */
    public int npl;
    /**
     * 左子节点
     */
    public LeftistHeapNode<T> left;
    /**
     * 右子节点
     */
    public LeftistHeapNode<T> right;

    public LeftistHeapNode(T val) {
        this.val = val;
    }

    @Override
    public String toString() {
        return val.toString() + "(" + npl + ")";
    }

}
```

```java
/**
 * 左倾堆（Leftist Heap）实现类
 * <p>
 * 基本性质：
 * <p>
 * 1. 节点的键值小于或等于它的左右子节点的键值
 * <p>
 * 2. 节点的左孩子的NPL >= 右孩子的NPL
 * <p>
 * 3. 节点的NPL = 它的右孩子的NPL + 1
 *
 * @author weijiaduo
 * @since 2023/9/26
 */
public class LeftistHeapImpl<T extends Comparable<T>> implements LeftistHeap<T> {

    /**
     * 根节点
     */
    private LeftistHeapNode<T> root;

    /**
     * 节点数量
     */
    private int size;

    @Override
    public void merge(LeftistHeap<T> other) {
        if (!(other instanceof LeftistHeapImpl<T>)) {
            return;
        }

        root = merge(root, ((LeftistHeapImpl<T>) other).root);
        size += other.size();
    }

    @Override
    public void insert(T val) {
        root = merge(root, new LeftistHeapNode<>(val));
        size++;
    }

    @Override
    public T removeFirst() {
        if (size <= 0) {
            throw new IllegalStateException("Heap is Empty!");
        }

        LeftistHeapNode<T> node = root;
        root = merge(root.left, root.right);
        node.left = node.right = null;
        size--;
        return node.val;
    }

    @Override
    public T first() {
        if (size <= 0) {
            throw new IllegalStateException("Heap is Empty!");
        }

        return root.val;
    }

    @Override
    public int size() {
        return size;
    }

    @Override
    public boolean isEmpty() {
        return size <= 0;
    }

    /**
     * 合并 2 个堆，并返回新堆的根节点
     *
     * @param h1 堆1 根节点
     * @param h2 堆2 根节点
     * @return 新堆的根节点
     */
    private LeftistHeapNode<T> merge(LeftistHeapNode<T> h1, LeftistHeapNode<T> h2) {
        if (h1 == null) {
            return h2;
        }
        if (h2 == null) {
            return h1;
        }

        // 以小根节点作为新堆的根节点
        LeftistHeapNode<T> min, max;
        if (h1.val.compareTo(h2.val) <= 0) {
            min = h1;
            max = h2;
        } else {
            min = h2;
            max = h1;
        }

        // 小根节点的右子节点和大根节点合并
        min.right = merge(min.right, max);

        // 右子节点距离 > 左子节点距离，交换左右子节点
        if (npl(min.right) > npl(min.left)) {
            LeftistHeapNode<T> tmp = min.left;
            min.left = min.right;
            min.right = tmp;
        }

        // 根节点距离 = 右子节点距离 + 1
        min.npl = npl(min.right) + 1;

        return min;
    }

    /**
     * 节点的 NPL 距离
     *
     * @param node 节点
     * @return NPL 距离
     */
    private int npl(LeftistHeapNode<T> node) {
        // null 节点的距离为 -1
        return node != null ? node.npl : -1;
    }

}
```
