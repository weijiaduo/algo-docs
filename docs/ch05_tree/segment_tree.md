# 线段树

## 一、线段树是什么？

- 线段树本质上是一种缓存，它缓存的是区间值
- 线段树一棵平衡二叉树
- 线段树的节点表示的是一个区间，节点值表示区间值

<!--more-->

## 二、为什么要用线段树？

线段树的常见用途有：

- 缓存区间值，提高多次查询区间值的性能
- 懒更新区间，减少修改的次数，提高多次修改区间的性能

简单点说就是：

- 通过缓存，提高多次区间查询和区间修改的性能
- 重点在于多次，多次，多次！！！

下面说明线段树如何通过缓存提高性能。

一般情况下，当需要修改一个区间内的所有值时，只能通过遍历的方式实现：

```java
for (int i = start; i < end; i++) {
    nums[i] = val;
}
```

包括获取查询某个区间内的最大值、最小值、区间和时，也只能遍历整个区间：

```java
int sum = 0;
for (int i = start; i < end; i++) {
    sum += nums[i];
}
```

像这种区间查询和区间修改，没有其他办法，也只能通过遍历来实现了。

但是，如果说会多次执行区间查询，每次都要遍历一遍的话，性能会比较差。

- 能不能把区间查询的结果缓存下来，下次查询时直接返回缓存？

比如，在区间之上缓存一个区间和结果：

``` 
             ____
            | 28 |               -- 缓存区间值
              |        
   _______________________
  |                       |
 ___ ___ ___ ___ ___ ___ ___
| 1 | 6 | 7 | 2 | 5 | 4 | 3 |    -- 原始数组（已排除索引 0，从 1 开始）
```

下次查询 `[1, 7]` 的区间和时，就可以直接从缓存节点返回结果 28。

这样貌似可行，但是只缓存一个够了吗？不够，如果要查询区间 `[2 ,4]`，还是要遍历。

所以需要把各个区间的结果都缓存了，最终缓存结构就变成了一棵二叉树：

``` 
                                  ____
                                 | 28 | ([1, 7])
                                   |
                  ____________________________________
                 |                                    |
                ____                                 ____
               | 16 | ([1, 4])                      | 12 | ([5, 7])
                 |                                    |
        ___________________                     ______________
       |                   |                   |              |
      ___                 ___                 ___             |
     | 7 | ([1, 2])      | 9 | ([3, 4])      | 9 | ([5, 6])   |
       |                   |                   |              |
   _________           _________           _________          |
  |         |         |         |         |         |         |
 ___       ___       ___       ___       ___       ___       ___
| 1 |     | 6 |     | 7 |     | 2 |     | 5 |     | 4 |     | 3 |
```

这个二叉树结构就是线段树，每个节点表示一个区间，节点值就是区间值。

线段树实际就是一个二叉树缓存结构，上一层缓存着下一层节点的区间值。

有了线段树，区间的查询就变简单了，比如查询区间 `[3, 7]`：

1. 从根节点出发，利用二分法逐层往下寻找查询区间
2. 最后找到了满足要求的区间节点 `[3, 4]` 和 `[5, 7]`
3. 区间节点 `[3, 4]` 的值是 9，区间节点 `[5, 7]` 的值是 12
4. 所以 `[3, 7]` 的区间和等于 `9 + 12 = 21`

有了线段树以后，无需再遍历所有节点，而是通过上层的缓存节点就能拿到结果。

- 区间查询的复杂度从 `O(n)` 降为了 `O(logn)`

线段树就是利用空间换时间，通过缓存来提高区间查询的性能。

- 线段树本质就是缓存，父节点就是子节点的缓存

至于父节点缓存的是什么数据，视情况而定。比如说：

- 区间最大值
- 区间最小值
- 区间和
- ...

使用线段树前，应该想清楚，缓存的数据究竟是什么？


## 三、线段树如何实现？

线段树的实现方式主要分为 2 种类型：

- 静态线段树：所有节点一开始就构建好了，和区间范围有关
- 动态线段树：节点是动态创建的，随数据变化而变化

静态线段树包括：

- 简单线段树：基于数组实现的满二叉树结构

动态线段树包括：

- 动态估点线段树：基于数组实现的动态二叉树
- 动态开点线段树：基于指针实现的动态二叉树

下面分别介绍这几种线段树。

### 3.1 简单线段树

#### 3.1.1 基本思路

- 采用数组存储线段树节点
- 数组存储的是一棵满二叉树，其中有些节点是多余的
- 根节点的数组索引是 `1`
- 左子节点索引 `2 * i`；右子节点索引 `2 * i + 1`
- 总节点数量，一般取区间 `[1, n]` 的 4 倍，即 `4n`

#### 3.1.2 存储结构

线段树是一棵平衡二叉树，也就是左右子树节点数量相差不超过 1。

但是平衡二叉树在数组上的存储、构造和取数都不方便，所以数组存储的是满二叉树结构。

满二叉树在数组中的父子节点索引关系刚好是：

- 根节点索引是 `1`
- 左子节点索引 `2 * i`
- 右子节点索引 `2 * i + 1`

利用满二叉树的性质，很容易在数组上实现线段树。

#### 3.1.3 节点数量

二叉树的节点数量，从上往下统计的话：

```
2 ^ 0           -- 第1层
2 ^ 1           -- 第2层
2 ^ 2           -- 第3层
2 ^ 3           -- 第4层
...
2 ^ (h - 1)     -- 第h层
...
```

如果区间范围 `n = 2 ^ k` 的话，那么刚好是满二叉树，节点总数就是 `S = 2n - 1`。

但如果区间范围是 `n = 2 ^ k + c` 的话，多出了几个，二叉树的最后一层就没有满。

- 满二叉树有一个特点，下一层的节点数量等于前面所有层级的节点数量总和 + 1

所以，如果填满了最后一层构成满二叉树的话，总节点数量就满足：

- `2n - 1 <= S < (2n - 1) + 2n = 4n - 1`

所以，在初始化线段树时：

- 总节点数量一般取区间范围的 4 倍

但里面不是所有节点都会用到，有些多余的节点。

#### 3.1.4 构建过程

简单线段树的构建步骤如下：

1. 计算出区间 `[1, n]` 的总节点数量 `4n`
2. 初始化所有树节点
3. 递归构建二叉树结构

比如构建“区间和”线段树的过程，类似这样：

(1) 初始化区间节点

```java
public SegmentTree(int n) {
    // 总节点数量 4n
    tree = new Node[4 * n];
    // 初始化所有节点
    for (int i = 0; i < tree.length; i++) {
        tree[i] = new Node();
    }
}
```

(2) 递归构建二叉树

```java
public void buildTree(int root, int start, int end) {
    // 叶子节点
    if (start == end) {
        tree[root].val = arr[start];
        return ;
    }

    // 左子节点索引
    int left = 2 * root;
    // 右子节点索引
    int right = 2 * root + 1;

    // 递归构建树
    int mid = start + (end - start) / 2;
    buildTree(left, start, mid);
    buildTree(right, mid + 1, end);

    // 向上更新父节点缓存
    pushUp(node);
}

private void pushUp(int root) {
    // 左子节点索引
    int left = 2 * root;
    // 右子节点索引
    int right = 2 * root + 1;
    // 更新父节点的区间值
    tree[root].val = tree[left].val + tree[right].val;
}
```

比如，数组 `arr = [, 1, 8, 2, 7, 4]`（注意数组 0 的位置不用）：

``` 
                                   ____
                                  | 18 | ([1, 5])
                                    |
                  _______________________________________
                 |                                       |
                ____                                    ___
               | 11 | ([1, 3])                         | 7 | ([4, 5])
                 |                                       |
        ___________________                     ___________________
       |                   |                   |                   |
      ___                 ___                 ___                 ___
     | 9 | ([1, 2])      | 2 |               | 3 |               | 4 |
       |
   _________       
  |         |     
 ___       ___
| 1 |     | 8 |
```

假设区间范围就是 `[1, 5]`，也就是 `n = 5`，所以需申请 `4 * 5 = 20` 个节点空间。

但是实际上只用了 `9` 个空间，不过为了按照满二叉树结构存储数据，也需要 `15` 个节点了。

至于其他还多出来的节点，是由于 `4n` 的误差引起的，毕竟 `4n` 并不是精确的节点数量值。


### 3.2 动态估点线段树

#### 3.2.1 基本思路

- 根据数据情况，预估可能会访问到的区间范围，从而估计节点数量
- 根节点的数组索引是 1
- 子节点索引不确定，父节点内维护有左右子节点的索引 l 和 r
- 维护有一个 size 大小，表示当前节点数量，也可用于计算下一个节点的索引
- 动态开点时，新节点的索引就等于 `size + 1`，比如  `node.lp = ++size`
- 不算是完全动态，数组一开始就已经申请好空间了，动态的只有创建 `Node` 节点而已

#### 3.2.2 区间估点

静态线段树的数组长度是按照满二叉树的节点数量来设置的。

但是在实际上未必会用到这么多节点，所以可能会采用一种估点的方式来简化。

- 估点：就是估计线段树会用到实际区间范围，来替代理论上的区间范围

比如说，线段树的区间范围定义是 `[1, 1000]`，但是实际查询数据范围只用到了 `[1, 200]`。

在这种情况下，完全没必要为线段树创建 `4 * 1000` 个节点，因为很多节点实际上根本访问不到。

- 可以根据实际数据的查询范围，估计一个合理的节点数量
- 在线段树初始化数组时，就采用估点值来初始化数组长度

网上提供了一些公式，比如 `6 * m * logn`，其中，m 是查询次数，n 是区间值域大小。

#### 3.2.3 动态建点

动态线段树的节点是动态创建的，所以不再维护满二叉树的结构了。

- 新创建的节点都放在数组末尾
- 父节点内保存指向左右子节点的索引

节点的结构定义如下：

```java
class Node {
    int val;   // 区间值
    int left;  // 左子节点索引
    int right; // 右子节点索引
}
```

新节点的创建就类似这样，添加到数组末尾：

```java
// 更新父节点中的左子节点索引
node.left = ++size;
// 创建左子节点
tree[node.left] = new Node();
```

#### 3.2.4 构建过程

动态估点线段树的构建步骤如下：

1. 估计区间节点范围
2. 根据估点值初始化数组长度
3. 初始化根节点
4. 访问时动态创建其他节点

比如构建“区间和”线段树的过程，类似这样：

(1) 区间估点 + 初始化

```java
public SegmentTree(int low, int high) {
    // 理论上的区间范围
    this.low = low;
    this.high = high;

    // 估点区间范围 6 * m * logn
    int n = (int) (6 * 10000 * Math.log(high));
    // 按照估点创建数组
    tree = new Node[n];

    // 初始化根节点
    tree[1] = new Node();
}
```

(2) 访问节点时动态建点

```java
public int query(Node node, int start, int end, int l, int r) {
    ...

    // 访问节点前，先向下推送更新
    pushDown(node, start, end);

    ...
}

public int update(Node node, int start, int end, int l, int r) {
    ...

    // 访问节点前，先向下推送更新
    pushDown(node, start, end);

    ...
}

private void pushDown(Node node, int start, int end) {
    // 动态开点，新节点总是添加到数组末尾
    if (node.left == 0) {
        node.left = ++size;
        tree[node.left] = new Node();
    }
    if (node.right == 0) {
        node.right = ++size;
        tree[node.right] = new Node();
    }

    ...
}
```


### 3.3 动态开点线段树

#### 3.3.1 基本思路

- 父子节点间采用指针进行链接
- 维护有一个根节点
- 每个节点维护有左右子节点的指针 `left` 和 `right`
- 动态开点时，直接创建新节点，比如 `node.left = new Node()`

#### 3.3.2 动态建点

基于指针的动态建点再简单不过了，就是平常的树结构：

```java
class Node {
    int val;     // 区间值
    Node left;   // 左子节点
    Node right;  // 右子节点
}
```

在访问到节点前，动态创建节点：

```java
// 创建左子节点
node.left = new Node();
```

#### 3.3.3 构建过程

动态开点线段树的构建步骤如下：

1. 初始化根节点
2. 访问时动态创建其他节点

比如构建“区间和”线段树的过程，类似这样：

(1) 初始化根节点 + 初始化区间

```java
public SegmentTree(int low, int high) {
    this.root = new Node();
    this.low = low;
    this.high = high;
}
```

(2) 访问时动态创建其他节点


```java
public int query(Node node, int start, int end, int l, int r) {
    ...

    // 动态向下更新
    pushDown(node, start, end);

    ...
}

public int update(Node node, int start, int end, int l, int r) {
    ...

    // 动态向下更新
    pushDown(node, start, end);

    ...
}

private void pushDown(Node node, int start, int end) {
    // 动态开点
    if (node.left == null) {
        node.left = new Node();
    }
    if (node.right == null) {
        node.right = new Node();
    }

    ...
}
```

### 3.4 小结

静态线段树的特点：

- 一开始就要申请全部空间，以及构建好线段树结构
- 占用空间比较大，初始化会比较慢
- 有些节点未必会访问到，空间利用率低

所以又引申出了动态线段树：

- 用到这个节点时，再申请节点空间以及初始化

动态线段树的实现方式有 2 种：

- 动态估点：基于数组
- 动态开点：基于指针

视不同的情况可以选择不同的线段树。


## 四、线段树的优化

### 4.1 懒标记更新

#### 4.1.1 懒更新引入

线段树本质是一棵缓存树，在数据没有修改的情况下，查询性能确实高。

但是如果频繁对区间数据进行修改，就可能会对性能造成影响。

比如说，要把区间 `[1, 5]` 内的值都更新为 1，且此时在线段树中找到了 `[1, 3]` 和 `[4, 5]` 这 2 个区间。

按正常逻辑，需要把以 `[1, 3]` 和 `[4, 5]` 为根节点的 2 棵子树的所有节点值都更新为 1。

但是子树里面包括了很多节点，比如 `[1, 1]`、`[2, 2]`、`[1, 2]` ... `[5, 5]` 等等。

- 这些节点不一定都会被访问到，也许永远没人访问，白更新了
- 每次更新区间都把所有相关子节点都更新一遍的话，势必会影响性能

为此，提出了一种懒更新的方式：

- 更新区间时，先标记根节点被更新了，但是暂时不要同步到子节点里面
- 下次访问子节点前，先从父节点上把上次的数据同步到子节点里面

使用懒更新后，只需要更新 `[1, 3]` 和 `[4, 5]` 这 2 个节点就够了，速度就很快。

等到下次查询区间 `[1, 2]` 时，才会从 `[1, 3]` 区间上同步之前的更新下来。

#### 4.1.2 懒标记实现

在节点中引入懒标记，同时用于保存更新值：

```java
class Node {
    int val;     // 区间值
    Node left;   // 左子节点
    Node right;  // 右子节点
    int add;     // 懒标记，0 时表示无更新，其他表示有更新
}
```

然后在更新区间时，只需要更新父节点就行了。

等下次访问子节点前，再从父节点同步数据到子节点中：

```java
public void update(Node node, int start, int end, int l, int r, int val) {
    if (l <= start && end <= r) {
        // 更新区间节点值
        node.val = val;
        // 更新懒标记
        node.add = val;
        return;
    }

    // 动态向下更新
    pushDown(node, start, end);

    ...
}

private void pushDown(Node node, int start, int end) {
    ...

    // 父节点没有更新
    if (node.add == 0) {
        return;
    }

    // 同步到子节点
    node.left.val = node.add;
    node.left.add = node.add;
    node.right.val = node.add;
    node.right.add = node.add;

    // 重置父节点的懒标记
    node.add = 0;
}
```

原理很简单，就是尽量不往下面更新，减少更新时间。


## 五、线段树模板

线段树有 2 个主要操作：

- 区间查询
- 区间更新

因此接口只定义了这 2 个。

### 5.1 接口定义

```java
public interface SegmentTree {

    /**
     * 查询指定区间的值
     *
     * @param l 目标区间[l, r]的左边界
     * @param r 目标区间[l, r]的右边界
     * @return 区间值
     */
    int query(int l, int r);

    /**
     * 更新指定区间的值
     *
     * @param l   目标区间[l, r]的左边界
     * @param r   目标区间[l, r]的右边界
     * @param val 更新的值
     */
    void update(int l, int r, int val);

}
```

### 5.1 动态估点（数组）

```java
public abstract class ArraySegmentTree implements SegmentTree {

    public static class Node {
        /**
         * 节点值
         */
        public int val;
        /**
         * 懒惰标记
         */
        public int add;
        /**
         * 左右节点数组下标
         */
        public int left, right;
    }

    /**
     * 树节点数组
     */
    protected final Node[] tree;
    /**
     * 区间最小值
     */
    protected final int low;
    /**
     * 区间最大值
     */
    protected final int high;
    /**
     * 当前节点数量
     */
    protected int size;

    public ArraySegmentTree(int low, int high) {
        this.low = low;
        this.high = high;

        // 区间估点 4n
        int n = 4 * high;
        tree = new Node[n];

        // 初始化根节点
        tree[1] = new Node();
        size = 1;
    }

    @Override
    public int query(int l, int r) {
        return query(tree[1], low, high, l, r);
    }

    /**
     * 查询指定区间的值
     *
     * @param node  当前节点
     * @param start 当前区间[start, end]的左边界
     * @param end   当前区间[start, end]的右边界
     * @param l     目标区间[l, r]的左边界
     * @param r     目标区间[l, r]的右边界
     * @return 区间值
     */
    protected int query(Node node, int start, int end, int l, int r) {
        if (l <= start && end <= r) {
            return node.val;
        }

        // 访问节点前，先向下推送更新
        pushDown(node, start, end);

        // 分别取左右子区间的值
        Integer lResult = null, rResult = null;
        int mid = middle(start, end);
        if (l <= mid) {
            lResult = query(tree[node.left], start, mid, l, r);
        }
        if (r > mid) {
            rResult = query(tree[node.right], mid + 1, end, l, r);
        }

        // 合并子区间的查询结果
        return mergeQuery(node, start, end, lResult, rResult);
    }

    @Override
    public void update(int l, int r, int val) {
        update(tree[1], low, high, l, r, val);
    }

    /**
     * 更新指定区间的值
     *
     * @param node  当前节点
     * @param start 当前区间[start, end]的左边界
     * @param end   当前区间[start, end]的右边界
     * @param l     目标区间[l, r]的左边界
     * @param r     目标区间[l, r]的右边界
     * @param val   更新的值
     */
    protected void update(Node node, int start, int end, int l, int r, int val) {
        if (l <= start && end <= r) {
            // 更新符合区间要求的节点值
            writeDown(node, start, end, val);
            return;
        }

        // 访问节点前，先向下推送更新
        pushDown(node, start, end);

        // 递归更新左右子区间
        int mid = middle(start, end);
        if (l <= mid) {
            update(tree[node.left], start, mid, l, r, val);
        }
        if (r > mid) {
            update(tree[node.right], mid + 1, end, l, r, val);
        }

        // 子节点更新后，向上推送更新
        pushUp(node, start, end);
    }

    /**
     * 子节点的值上推到父节点
     *
     * @param node  父节点
     * @param start 当前区间[start, end]的左边界
     * @param end   当前区间[start, end]的右边界
     */
    protected void pushUp(Node node, int start, int end) {
        writeUp(node, start, end);
    }

    /**
     * 父节点的值下推到子节点
     *
     * @param node  当前节点
     * @param start 当前区间[start, end]的左边界
     * @param end   当前区间[start, end]的右边界
     */
    protected void pushDown(Node node, int start, int end) {
        // 动态开点，新节点总是添加到数组末尾
        if (node.left == 0) {
            node.left = ++size;
            tree[node.left] = new Node();
        }
        if (node.right == 0) {
            node.right = ++size;
            tree[node.right] = new Node();
        }

        // 没有懒标记，无需再往下推
        if (node.add == 0) {
            return;
        }

        // 把懒标记下推给子节点
        int mid = middle(start, end);
        writeDown(tree[node.left], start, mid, node.add);
        writeDown(tree[node.right], mid + 1, end, node.add);

        // 懒标记已处理
        node.add = 0;
    }

    /**
     * 合并指定区间的查询结果
     *
     * @param node  当前节点
     * @param start 当前区间[start, end]的左边界
     * @param end   当前区间[start, end]的右边界
     * @param lVal  左区间的查询结果
     * @param rVal  右区间的查询结果
     * @return 区间值
     */
    protected abstract int mergeQuery(Node node, int start, int end, Integer lVal, Integer rVal);

    /**
     * 向上更新节点的值
     *
     * @param node  当前节点
     * @param start 当前区间[start, end]的左边界
     * @param end   当前区间[start, end]的右边界
     */
    protected abstract void writeUp(Node node, int start, int end);

    /**
     * 向下更新节点的值
     *
     * @param node  当前节点
     * @param start 当前区间[start, end]的左边界
     * @param end   当前区间[start, end]的右边界
     * @param val   更新的值
     */
    protected abstract void writeDown(Node node, int start, int end, int val);

    /**
     * 划分左右区间的中点
     *
     * @param start 左边界[start, end]
     * @param end   右边界[start, end]
     * @return 中点
     */
    protected int middle(int start, int end) {
        return start + (end - start) / 2;
    }

}
```

### 5.3 动态开点（指针）

```java
public abstract class LinkSegmentTree implements SegmentTree {

    public static class Node {
        /**
         * 节点值
         */
        public int val;
        /**
         * 懒惰标记
         */
        public int add;
        /**
         * 左右节点
         */
        public Node left, right;
    }

    /**
     * 根节点
     */
    protected final Node root;
    /**
     * 区间最小值
     */
    protected final int low;
    /**
     * 区间最大值
     */
    protected final int high;

    public LinkSegmentTree(int low, int high) {
        this.low = low;
        this.high = high;
        this.root = new Node();
    }

    /**
     * 查询指定区间的值
     *
     * @param l 目标区间[l, r]的左边界
     * @param r 目标区间[l, r]的右边界
     * @return 区间值
     */
    @Override
    public int query(int l, int r) {
        return query(root, low, high, l, r);
    }

    /**
     * 查询指定区间的值
     *
     * @param node  当前节点
     * @param start 当前区间[start, end]的左边界
     * @param end   当前区间[start, end]的右边界
     * @param l     目标区间[l, r]的左边界
     * @param r     目标区间[l, r]的右边界
     * @return 区间值
     */
    protected int query(Node node, int start, int end, int l, int r) {
        if (l <= start && end <= r) {
            return node.val;
        }

        // 访问节点前，先向下推送更新
        pushDown(node, start, end);

        // 分别取左右子区间的值
        Integer lResult = null, rResult = null;
        int mid = middle(start, end);
        if (l <= mid) {
            lResult = query(node.left, start, mid, l, r);
        }
        if (r > mid) {
            rResult = query(node.right, mid + 1, end, l, r);
        }

        // 合并子区间的查询结果
        return mergeQuery(node, start, end, lResult, rResult);
    }

    /**
     * 更新指定区间的值
     *
     * @param l   目标区间[l, r]的左边界
     * @param r   目标区间[l, r]的右边界
     * @param val 更新的值
     */
    @Override
    public void update(int l, int r, int val) {
        update(root, low, high, l, r, val);
    }

    /**
     * 更新指定区间的值
     *
     * @param node  当前节点
     * @param start 当前区间[start, end]的左边界
     * @param end   当前区间[start, end]的右边界
     * @param l     目标区间[l, r]的左边界
     * @param r     目标区间[l, r]的右边界
     * @param val   更新的值
     */
    protected void update(Node node, int start, int end, int l, int r, int val) {
        if (l <= start && end <= r) {
            // 更新符合区间要求的节点值
            writeDown(node, start, end, val);
            return;
        }

        // 访问节点前，先向下推送更新
        pushDown(node, start, end);

        // 递归更新左右子区间
        int mid = middle(start, end);
        if (l <= mid) {
            update(node.left, start, mid, l, r, val);
        }
        if (r > mid) {
            update(node.right, mid + 1, end, l, r, val);
        }

        // 子节点更新后，向上推送更新
        pushUp(node, start, end);
    }

    /**
     * 子节点的值上推到父节点
     *
     * @param node  父节点
     * @param start 当前区间[start, end]的左边界
     * @param end   当前区间[start, end]的右边界
     */
    protected void pushUp(Node node, int start, int end) {
        writeUp(node, start, end);
    }

    /**
     * 父节点的值下推到子节点
     *
     * @param node  当前节点
     * @param start 当前区间[start, end]的左边界
     * @param end   当前区间[start, end]的右边界
     */
    protected void pushDown(Node node, int start, int end) {
        // 动态开点，指向动态创建的对象
        if (node.left == null) {
            node.left = new Node();
        }
        if (node.right == null) {
            node.right = new Node();
        }

        // 没有懒标记，无需再往下推
        if (node.add == 0) {
            return;
        }

        // 把懒标记下推给子节点
        int mid = middle(start, end);
        writeDown(node.left, start, mid, node.add);
        writeDown(node.right, mid + 1, end, node.add);

        // 懒标记已处理
        node.add = 0;
    }

    /**
     * 合并指定区间的查询结果
     *
     * @param node  当前节点
     * @param start 当前区间[start, end]的左边界
     * @param end   当前区间[start, end]的右边界
     * @param lVal  左区间的查询结果
     * @param rVal  右区间的查询结果
     * @return 区间值
     */
    protected abstract int mergeQuery(Node node, int start, int end, Integer lVal, Integer rVal);

    /**
     * 向上更新节点的值
     *
     * @param node  当前节点
     * @param start 当前区间[start, end]的左边界
     * @param end   当前区间[start, end]的右边界
     */
    protected abstract void writeUp(Node node, int start, int end);

    /**
     * 向下更新节点的值
     *
     * @param node  当前节点
     * @param start 当前区间[start, end]的左边界
     * @param end   当前区间[start, end]的右边界
     * @param val   更新的值
     */
    protected abstract void writeDown(Node node, int start, int end, int val);

    /**
     * 划分左右区间的中点
     *
     * @param start 左边界[start, end]
     * @param end   右边界[start, end]
     * @return 中点
     */
    protected int middle(int start, int end) {
        return start + (end - start) / 2;
    }

}
```

## 六、实际案例

### 6.1 LC 307. 区域和检索

> 题目描述

给你一个数组 nums ，请你完成两类查询。 

其中一类查询要求 更新 数组 nums 下标对应的值

另一类查询要求返回数组 nums 中索引 left 和索引 right 之间（ 包含 ）的nums元素的 和 ，其中 left <= right

实现 NumArray 类：

NumArray(int[] nums) 用整数数组 nums 初始化对象

void update(int index, int val) 将 nums[index] 的值 更新 为 val

int sumRange(int left, int right) 返回数组 nums 中索引 left 和索引 right 之间（ 包含 ）的> nums元素的 和 （即，nums[left] + nums[left + 1], ..., nums[right]

提示：

- 1 <= nums.length <= 3 * 104
- -100 <= nums[i] <= 100
- 0 <= index < nums.length
- -100 <= val <= 100
- 0 <= left <= right < nums.length
- 调用 update 和 sumRange 方法次数不大于 3 * 104 

---

> 问题分析

问题中涉及了 2 个操作：

- 单点更新
- 区间求和

单点更新，可以当成是长度为 1 区间更新，所以这 2 种操作都可认为是区间操作。

线段树正好可以用来解决这 2 个问题。

首先分析一下：

- 区间节点需要缓存什么数据？

从题目可知，区间查询的结果是区间和，所以：

- 父节点缓存的应该是子节点的区间和

确定节点缓存的数据之后，套用模板修改代码即可。

---

> 代码实现

这可能用简单线段树会好一些，因为区间范围只有 `3 * 10^4`，不是很大，直接构建线段树也很快。

不过这里是采用了动态开点线段树来作为例子演示。

```java
/**
 * 区间和线段树
 *
 * 父节点缓存的是子树所有节点的和
 */
public class SumLinkSegmentTree extends LinkSegmentTree {

    public SumLinkSegmentTree(int low, int high) {
        super(low, high);
    }

    @Override
    protected int mergeQuery(Node node, int start, int end, Integer lVal, Integer rVal) {
        // 合并子树节点的区间和
        int sum = 0;
        if (lVal != null) {
            sum += lVal;
        }
        if (rVal != null) {
            sum += rVal;
        }
        return sum;
    }

    @Override
    protected void writeUp(Node node, int start, int end) {
        // 子树的区间和发生变化后，父节点也要更新
        node.val = node.left.val + node.right.val;
    }

    @Override
    protected void writeDown(Node node, int start, int end, int val) {
        // end - start + 1 表示子树中叶子节点的数量
        // 区间内所有叶子节点都设为 val 的话
        // 那么父节点应该等于所有叶子节点的 val 总和
        node.val = (end - start + 1) * val;
        node.add = val;
    }

}
```

区间和线段树实现好了，下面直接按题目要求调用：

```java
class NumArray {

    SumSegmentTree segmentTree;
    int low = 0;
    int high = (int) 3e4;

    public NumArray(int[] nums) {
        segmentTree = new SumSegmentTree(low, high);
        for (int i = 0; i < nums.length; i++) {
            this.update(i, nums[i]);
        }
    }
    
    public void update(int index, int val) {
        segmentTree.update(index, index, val);
    }
    
    public int sumRange(int left, int right) {
        return segmentTree.query(left, right);
    }
}
```

这道题的解决方案还有其他方法，线段树的性能并不是最优的。

这里只是举例子说明一下线段树的用途。


### 6.2 LC 699. 掉落的方块

> 题目描述

在二维平面上的 x 轴上，放置着一些方块。

给你一个二维整数数组 positions ，其中 positions[i] = [lefti, sideLengthi] 表示：第 i 个方块边长为 sideLengthi ，其左侧边与 x 轴上坐标点 lefti 对齐。

每个方块都从一个比目前所有的落地方块更高的高度掉落而下。方块沿 y 轴负方向下落，直到着陆到 另一个正方形的顶边 或者是 x 轴上 。一个方块仅仅是擦过另一个方块的左侧边或右侧边不算着陆。一旦着陆，它就会固定在原地，无法移动。

在每个方块掉落后，你必须记录目前所有已经落稳的 方块堆叠的最高高度 。

返回一个整数数组 ans ，其中 ans[i] 表示在第 i 块方块掉落后堆叠的最高高度。

提示：

- 1 <= positions.length <= 1000
- 1 <= lefti <= 108
- 1 <= sideLengthi <= 106

---

> 问题分析

问题中出现的 2 个行为：

- 方块掉落在坐标轴上，占用了一个区间，这实际上就是区间更新
- 查询所有方块堆叠的最高高度，实际就是在查找区间内的最大值

所以也可以用线段树来实现区间更新和区间求最大值。

首先分析一下：

- 区间节点需要缓存什么数据？

从题目可知，查询的是区间最大值，所以：

- 父节点缓存的应该是子节点的区间最大值

确定节点缓存的数据之后，开始套用模板修改代码。

---

> 代码实现

这里的区间范围有 `10^8`，比较大，不适合用简单线段树，采用动态开点会更好一些。

```java
/**
 * 区间最大值线段树
 *
 * 父节点缓存的是子树的最大值
 */
public class MaxLinkSegmentTree extends LinkSegmentTree {

    public MaxLinkSegmentTree(int low, int high) {
        super(low, high);
    }

    @Override
    protected int mergeQuery(Node node, int start, int end, Integer lVal, Integer rVal) {
        // 取左右子树中的最大值
        int max = Integer.MIN_VALUE;
        if (lVal != null) {
            max = lVal;
        }
        if (rVal != null) {
            max = Math.max(rVal, max);
        }
        return max;
    }

    @Override
    protected void writeUp(Node node, int start, int end) {
        // 子节点更新后，父节点也要更新最大值
        node.val = Math.max(node.left.val, node.right.val);
    }

    @Override
    protected void writeDown(Node node, int start, int end, int val) {
        // 子节点都更新为 val 的话，那么所有值都会相等
        // 此时区间最大值就是 val
        node.val = val;
        node.add = val;
    }

}
```

接着按题目要求实现调用代码即可：

```java
public List<Integer> fallingSquares(int[][] positions) {
    List<Integer> ans = new ArrayList<>(positions.length);
    int low = 1, high = (int) 1e8;
    MaxLinkSegmentTree segmentTree = new MaxLinkSegmentTree(low, high);
    for (int[] position : positions) {
        int x = position[0];
        int w = position[1];

        // 找到区间内的最大值
        int maxH = segmentTree.query(x, x + w - 1);
        // 堆叠新落下的方块高度
        segmentTree.update(x, x + w - 1, maxH + w);

        ans.add(segmentTree.query(low, high));
    }
    return ans;
}
```

### 6.3 LC 715. Range 模块

> 题目描述

Range模块是跟踪数字范围的模块。

设计一个数据结构来跟踪表示为 半开区间 的范围并查询它们。

半开区间 [left, right) 表示所有 left <= x < right 的实数 x 。

实现 RangeModule 类:

RangeModule() 初始化数据结构的对象。

void addRange(int left, int right) 添加 半开区间 [left, right)，跟踪该区间中的每个实数。添加与当前跟踪的数字部分重叠的区间时，应当添加在区间 [left, right) 中尚未跟踪的任何数字到该区间中。

boolean queryRange(int left, int right) 只有在当前正在跟踪区间 [left, right) 中的每一个实数时，才返回 true ，否则返回 false 。

void removeRange(int left, int right) 停止跟踪 半开区间 [left, right) 中当前正在跟踪的每个实数。

提示：

- 1 <= left < right <= 10^9
- 在单个测试用例中，对 addRange 、queryRange 和 removeRange 的调用总数不超过 104 次

---

> 问题分析

问题已经说的很清楚了，就是区间查询和区间更新。

现在还需要确认一下：

- 区间节点需要缓存什么样的数据？

跟踪区间，可以理解为区间被标记了。那么可以这样来标记：

- 区间值为 1 时：表示被跟踪了
- 区间值小于 1 时：表示未跟踪

这个时候，只要查出区间内的最小值，就能知道区间是否被跟踪了。所以，

- 父节点缓存的应该是子节点的区间最小值

确定节点缓存的数据之后，开始套用模板修改。

---

> 代码实现

这里的区间范围有 `10^9`，比较大，不适合用简单线段树，采用动态开点会更好一些。

```java
/**
 * 区间最小值线段树
 *
 * 父节点缓存的是子树的最小值
 */
public class MaxLinkSegmentTree extends LinkSegmentTree {

    public MaxLinkSegmentTree(int low, int high) {
        super(low, high);
    }

    @Override
    protected int mergeQuery(Node node, int start, int end, Integer lVal, Integer rVal) {
        // 取左右子树中的最小值
        int min = Integer.MAX_VALUE;
        if (lVal != null) {
            min = lVal;
        }
        if (rVal != null) {
            min = Math.min(rVal, min);
        }
        return min;
    }

    @Override
    protected void writeUp(Node node, int start, int end) {
        // 子节点更新后，父节点也要更新最小值
        node.val = Math.min(node.left.val, node.right.val);
    }

    @Override
    protected void writeDown(Node node, int start, int end, int val) {
        // 子节点都更新为 val 的话，那么所有值都会相等
        // 此时区间最小值就是 val
        node.val = val;
        node.add = val;
    }

}
```

接着按照题目要求，实现其他的调用代码即可：

```java
class RangeModule {

    final int low = 1;
    final int high = (int) 1e9;
    MaxLinkSegmentTree segmentTree;

    public RangeModule() {
        segmentTree = new MinLinkSegmentTree(low, high);
    }
    
    public void addRange(int left, int right) {
        segmentTree.update(left, right - 1, 1);
    }
    
    public boolean queryRange(int left, int right) {
        int ret = segmentTree.query(left, right - 1);
        // 区间最小值大于 0 表示被跟踪了
        return ret > 0;
    }
    
    public void removeRange(int left, int right) {
        // 这里更新的值必须使用小于 0 的整数
        // 不能用 0，因为懒标志用 0 表示无更新了
        // 设成 0 的话会导致线段树不更新数据
        // 相当于 0 已经被懒标志占用了，使用 0 会造成冲突
        segmentTree.update(left, right - 1, -1);
    }
}
```

### 6.4 小结

上面列举了几种线段树的几种常见使用场景：

- 区间和
- 区间最大值
- 区间最小值

线段树本质就是区间缓存，所以对于：

- 多次区间查询
- 多次区间更新

就比较适用，不过具体用哪种，视情况而定。


## 总结

什么是线段树？

- 线段树本质上是一种缓存，它缓存的是区间值
- 父节点就是子节点的缓存

为什么要用线段树？

- 通过缓存，提高多次区间查询和区间修改的性能
- 重点在于多次，多次，多次！！！

线段树的实现方式？

- 静态线段树：一开始就创建好所有节点和构建树结构
- 动态线段树：区间访问过程中动态创建节点和初始化

静态线段树：

- 简单线段树（基于数组）

动态线段树：

- 动态估点（基于数组）
- 动态开点（基于指针）


## 参考

> https://leetcode.cn/problems/range-sum-query-mutable/solution/by-lfool-v3x9/
>
> https://leetcode.cn/problems/my-calendar-iii/solution/xian-duan-shu-by-xiaohu9527-rfzj/
>
> https://leetcode.cn/problems/falling-squares/solution/by-ac_oier-zpf0/
>
> https://www.cnblogs.com/chengmf/p/16451615.html
