---
title: 跳表
date: 2022-09-18 22:31:54
categories: [数据结构与算法, 数据结构]
tags: [算法, 数据结构, 跳表]
thumbnail: /img/structure.jpg
top: true
---

# 跳表

## 一、什么是跳表？

跳表，又叫做跳跃表、跳跃列表。

- 是一种对有序链式线性表的优化
- 在原始链表的基础上添加了多级索引链表
- 分为多层，从下往上分别是原始链表、一级索引、二级索引...
- 搜索时从上往下，实现了类似“二分查找”的功能

<!--more-->

它的结构大概是这样的：

```
 ___                                 ___
| 1 | ----------------------------> | 5 |                -- 二级索引
  |                                   |
 ___               ___               ___
| 1 | ----------> | 3 | ----------> | 5 |                -- 一级索引
  |                 |                 |
 ___      ___      ___      ___      ___      ___
| 1 | -> | 2 | -> | 3 | -> | 4 | -> | 5 | -> | 6 |       -- 原始链表
```

跳表就是对有序链表的一种优化，通过添加多级索引来提升性能。

跳表本质上是一种利用空间来换时间的数据结构。


## 二、为什么要用跳表？

### 2.1 引入背景

对于链表来说，查找某个值，只能通过遍历的方式实现，时间复杂度是 `O(n)`。

即使链表是有序的，依旧也只能是从头到尾遍历，完全没有用到数据的有序性。

```
 ___      ___      ___      ___      ___      ___
| 1 | -> | 2 | -> | 3 | -> | 4 | -> | 5 | -> | 6 |
```

在这种情况下，有没有办法用到链表的有序性？

- 给链表建立索引，方便快速定位数据的区间范围

比如说，链表的每2个节点就提取出一个索引节点：

```
 ___               ___               ___
| 1 | ----------> | 3 | ----------> | 5 |                -- 索引
  |                 |                 |
 ___      ___      ___      ___      ___      ___
| 1 | -> | 2 | -> | 3 | -> | 4 | -> | 5 | -> | 6 |       -- 原始链表
```

通过索引先快速定位到区间 `[1, 3]`、`[3, 5]` 以及 `[5, +]`，缩小范围，再小范围遍历搜索。

比如说，查找数值 4：

1. 首先在索引层遍历判断，发现 4 是在区间 `[3, 5]` 内的
2. 然后找到了原始链表的 3，再从 3 开始往右遍历，直到找到 4

4 的搜索链路就是：`1 -> 3 -> 4`，这定位速度就比遍历快多了。

有时候建第一层索引后，遍历范围还是很大（第一层索引的节点数量是 n/2）。

这个时候还可以再往上多建几层索引：

```
 ___                                 ___
| 1 | ----------------------------> | 5 |                -- 二级索引
  |                                   |
 ___               ___               ___
| 1 | ----------> | 3 | ----------> | 5 |                -- 一级索引
  |                 |                 |
 ___      ___      ___      ___      ___      ___
| 1 | -> | 2 | -> | 3 | -> | 4 | -> | 5 | -> | 6 |       -- 原始链表
```

这种在原始链表上建多层索引，实现快速查找的数据结构，就是跳表。


### 2.2 算法分析

原始有序链表的算法复杂度是：

- 搜索：时间 `O(n)`，空间 `O(1)`
- 删除：时间 `O(n)`，空间 `O(1)`
- 插入：时间 `O(n)`，空间 `O(1)`

跳表的算法复杂度如何呢？以 2 个节点 1 个索引的跳表为例。

1）空间复杂度

跳表的多层索引，从下往上数的话，节点数量分别是：

```
n/2, n/4, n/8, ..., 2
```

总共是 `n/2 + n/4 + ... + 2 = n - 2`，也就是说：

- 跳表结构本身的空间复杂度是 `O(n)`

2）时间复杂度

根据上面的结构，跳表的层级数量很容易知道：

- 跳表的索引层级数量是 `logn`

链表的插入和删除，实际上是依赖于搜索，所以只要分析搜索的时间复杂度即可。

因为每 2 个节点对应 1 个索引节点，所以每一层的搜索节点数量不会超过 3。

比如上一层索引确定了区间 `[3, 5]`，那么往下一层搜索，搜索只能是 `3,4,5`。

因此跳表的时间复杂度，依赖于每层遍历的节点数量：

- 搜索：时间 `O(m * logn)`
- 删除：时间 `O(m * logn)`
- 插入：时间 `O(m * logn)`

其中 m 是每层最多遍历的节点数量。

跳表将原始链表的时间复杂度从 `O(n)` 提升到了 `O(logn)` 级别。

当然，这是利用空间换时间才得到的，空间消耗变多了。


## 三、跳表如何实现？

跳表的实现方式可以分为2种：

- 数组实现：由一个数组实现，数组内部包含多层索引指针
- 链表实现：有多条链表实现，每条链表一层索引

它们的区别只在于底层存储不同，对外是一致的。

下面以链表实现为例，说明跳表的实现。

### 3.1 跳表的定义

#### 3.1.1 结构定义

为了减少代码中的判空，采用了一个哨兵头节点，所以实际的跳表结构是这样的：

```
head
 ___
|   | ---------------------------------------------------------> null
  |
 ___      ___                                 ___
|   | -> | 1 | ----------------------------> | 5 | ------------> null
  |        |                                   |
 ___      ___               ___               ___
|   | -> | 1 | ----------> | 3 | ----------> | 5 | ------------> null
  |        |                 |                 |
 ___      ___      ___      ___      ___      ___      ___
|   | -> | 1 | -> | 2 | -> | 3 | -> | 4 | -> | 5 | -> | 6 | ---> null
```

顶层的 `head` 节点单独一层，并且这一层只有它一个非空节点。


#### 3.1.2 接口定义

```java
public interface SkipList<T extends Comparable<T>> {

    /**
     * 搜索指定值是否存在
     *
     * @param value 指定值
     * @return true存在/false不存在
     */
    boolean search(T value);

    /**
     * 添加新值
     *
     * @param value 新值
     */
    void add(T value);

    /**
     * 删除指定值
     *
     * @param value 指定值
     * @return true删除成功/false删除失败
     */
    boolean erase(T value);

}
```

#### 3.1.3 链式跳表的定义

```java
public class LinkedSkipList<T extends Comparable<T>> implements SkipList<T> {

    class Node {
        // 节点值
        T value;
        // 同一层的右侧节点指针
        Node right;
        // 下一层的同值节点指针
        Node down;

        public Node(T value, Node right, Node down) {
            this.value = value;
            this.right = right;
            this.down = down;
        }
    }

}
```


### 3.2 跳表的搜索

#### 3.2.1 代码实现

跳表的搜索，就是从顶层的索引开始，一层一层往下找，直到找对对应的节点为止。

```java
public boolean search(T value) {
    Node p = head, q = null;
    // 从上往下一直找，直到最底层的原始链表为止
    while (p != null) {
        // 在当前层遍历搜索，直到找到 value 所在区间
        while (p.right != null && value.compareTo(p.right.value) > 0) {
            p = p.right;
        }
        // 搜索下一层
        q = p;
        p = p.down;
    }
    // 验证节点值是否相等
    return q != null && q.right != null && q.right.value.equals(value);
}
```

#### 3.2.2 结构示意

搜索过程就类似下面这样（暂时忽略其他线）：

```
head
 ___
|   |
  |
 ___      ___                                 ___
|   | -> | 1 |                               | 5 |
           |                                    
 ___      ___               ___               ___
|   |    | 1 | ----------> | 3 |             | 5 |
                             |                  
 ___      ___      ___      ___      ___      ___      ___
|   |    | 1 |    | 2 |    | 3 | -> | 4 |    | 5 |    | 6 |
```

很简单，就是一层一层找符合范围的区间，直到最底层的原始链表。


### 3.3 跳表的插入

#### 3.3.1 代码实现

插入节点，除了要知道插入点的位置，还要知道上一层跳到下一层的转折节点。

因为插入节点，不仅仅是插入最底层，上面的索引层也需要一起更新。

```java
/**
 * 查找指定值的前置路径
 *
 * @param value 指定值
 * @return 前置路径
 */
private List<Node> findProcessors(T value) {
    Deque<Node> stack = new ArrayDeque<>();
    Node p = head;
    // 从上往下一直找，直到最底层的原始链表为止
    while (p != null) {
        // 在当前层遍历搜索，直到找到 value 所在区间
        while (p.right != null && value.compareTo(p.right.value) > 0) {
            p = p.right;
        }
        // 记录上一层节点
        stack.push(p);
        // 搜索下一层
        p = p.down;
    }

    // 路径是倒序摆放的，最接近值 value 的在索引 0 的位置
    List<Node> path = new ArrayList<>(stack.size());
    while (!stack.isEmpty()) {
        path.add(stack.pop());
    }
    return path;
}
```

查找返回的是目标节点的前置路径：

- 前置路径是搜索过程中上一层跳到下一层的那个转折节点列表

比如上面搜索 4 时，搜索路径是 `1->3->4`，那么得到的前置路径就是 `[3, 1]`。

为什么前置路径是倒序的？

- 因为插入节点是从下往上插入的，为了方便就倒序返回了

找到前置路径后，就可以准备插入节点了：

```java
public void add(T value) {
    List<Node> processors = findProcessors(value);
    int levels = processors.size();

    Node newNode = null;
    boolean insertUp = true;
    for (int i = 0; i < levels && insertUp; i++) {
        Node processor = processors.get(i);
        // 新节点指向下一层的同值节点
        newNode = new Node(value, processor.right, newNode);
        // 在当前层插入新节点
        processor.right = newNode;

        // 是否要继续往上一层插入节点
        insertUp = random.nextDouble() < FACTOR;
    }

    // 更新层级
    if (insertUp) {
        levelUp();
    }
}
```

#### 3.3.2 结构示意

比如插入前跳表的结构是这样的：

```
head
 ___
|   | ---------------------------------------------------------> null
  |
 ___      ___                                 ___
|   | -> | 1 | ----------------------------> | 5 | ------------> null
  |        |                                   |
 ___      ___                                 ___
|   | -> | 1 | ----------------------------> | 5 | ------------> null
  |        |                                   |
 ___      ___      ___               ___      ___      ___
|   | -> | 1 | -> | 2 | ----------> | 4 | -> | 5 | -> | 6 | ---> null
```

此时插入一个 3，插入过程是从下往上的：

1）第一次插入

```
head
 ___
|   | ---------------------------------------------------------> null
  |
 ___      ___                                 ___
|   | -> | 1 | ----------------------------> | 5 | ------------> null
  |        |                                   |
 ___      ___                                 ___
|   | -> | 1 | ----------------------------> | 5 | ------------> null
  |        |                                   |
 ___      ___      ___      ___      ___      ___      ___
|   | -> | 1 | -> | 2 | -> | 3 | -> | 4 | -> | 5 | -> | 6 | ---> null
```

2）第二次插入

```
head
 ___
|   | ---------------------------------------------------------> null
  |
 ___      ___                                 ___
|   | -> | 1 | ----------------------------> | 5 | ------------> null
  |        |                                   |
 ___      ___               ___               ___
|   | -> | 1 | ----------> | 3 | ----------> | 5 | ------------> null
  |        |                 |                 |
 ___      ___      ___      ___      ___      ___      ___
|   | -> | 1 | -> | 2 | -> | 3 | -> | 4 | -> | 5 | -> | 6 | ---> null
```

至于需要插入多少层，这个是通过随机数控制的。

采用了随机数 `random.nextDouble() < FACTOR`，来判断要不要往上一层插入节点。

为什么采用随机数？

- 为了减少算法代码的复杂度
- 同时又尽量确保跳表的结构平衡

跳表本身代码就比较简单，如果为了维持结构而引入其他复杂算法，显得得不偿失。

随机数就很合适，使用不复杂，而且能大概率保证跳表结构不会严重退化。


#### 3.3.3 提升层级

如果节点被插入到了顶层，这个时候需要提升整个跳表的层级。

比如 3 节点被插入到顶层后，跳表结构变成了这样：

```
head
 ___                        ___
|   | -------------------> | 3 | ------------------------------> null
  |
 ___      ___               ___               ___
|   | -> | 1 | ----------> | 3 | ----------> | 5 | ------------> null
  |        |                 |                 |
 ___      ___               ___               ___
|   | -> | 1 | ----------> | 3 | ----------> | 5 | ------------> null
  |        |                 |                 |
 ___      ___      ___      ___      ___      ___      ___
|   | -> | 1 | -> | 2 | -> | 3 | -> | 4 | -> | 5 | -> | 6 | ---> null
```

为了维持跳表头节点在单独一层的结构，就需要提升层级：

```java
/**
 * 提升层级
 */
private void levelUp() {
    // 最上层只有一个头节点
    head = new Node(null, null, head);
}
```

提升层级后，跳表的结构就恢复正常了：

```
head
 ___
|   | ---------------------------------------------------------> null
  |
 ___                        ___
|   | -------------------> | 3 | ------------------------------> null
  |
 ___      ___               ___               ___
|   | -> | 1 | ----------> | 3 | ----------> | 5 | ------------> null
  |        |                 |                 |
 ___      ___               ___               ___
|   | -> | 1 | ----------> | 3 | ----------> | 5 | ------------> null
  |        |                 |                 |
 ___      ___      ___      ___      ___      ___      ___
|   | -> | 1 | -> | 2 | -> | 3 | -> | 4 | -> | 5 | -> | 6 | ---> null
```


### 3.4 跳表的删除

#### 3.4.1 代码实现

删除和插入过程是一样的，都是先找到前置路径后，再处理节点：

```java
public boolean erase(T value) {
    List<Node> processors = findProcessors(value);

    // 验证删除节点是否存在，没有找到则直接返回
    Node target = processors.get(0).right;
    if (target == null || !value.equals(target.value)) {
        return false;
    }

    // 从下往上，逐层删除指定节点
    for (Node processor : processors) {
        // 指定 value 值的节点在每一层都删完了，就跳出循环
        Node delNode = processor.right;
        if (delNode == null || !delNode.value.equals(value)) {
            break;
        }
        processor.right = delNode.right;
    }

    // 更新层级
    levelDown();

    return true;
}
```

#### 3.4.2 结构示意

删除也和插入一样，是从下往上的。比如删除 3：

1）第一次删除

```
head
 ___
|   | ---------------------------------------------------------> null
  |
 ___      ___                                 ___
|   | -> | 1 | ----------------------------> | 5 | ------------> null
  |        |                                   |
 ___      ___               ___               ___
|   | -> | 1 | ----------> | 3 | ----------> | 5 | ------------> null
  |        |                                   |
 ___      ___      ___               ___      ___      ___
|   | -> | 1 | -> | 2 | ----------> | 4 | -> | 5 | -> | 6 | ---> null
```

2）第二次删除

```
head
 ___
|   | ---------------------------------------------------------> null
  |
 ___      ___                                 ___
|   | -> | 1 | ----------------------------> | 5 | ------------> null
  |        |                                   |
 ___      ___                                 ___
|   | -> | 1 | ----------------------------> | 5 | ------------> null
  |        |                                   |
 ___      ___      ___               ___      ___      ___
|   | -> | 1 | -> | 2 | ----------> | 4 | -> | 5 | -> | 6 | ---> null
```

#### 3.4.3 降低层级

删除节点后，有可能导致跳表的上层节点变得比较少，此时需要降低层级。

比如把 1 和 5 都删掉后：

```
head
 ___
|   | ---------------------------------------------------------> null
  |
 ___                                          
|   | ---------------------------------------------------------> null
  |                                             
 ___                                      
|   | ---------------------------------------------------------> null
  |                                             
 ___               ___               ___               ___
|   | ----------> | 2 | ----------> | 4 | ----------> | 6 | ---> null
```

上面几层链表都空了，浪费空间，这个时候需要将这些节点很少的层级删除：

```java
/**
 * 降低层级
 */
private void levelDown() {
    while (head.down != null) {
        // 连续 2 层为空时，才降低层级，因为最上层只有 1 个头节点
        if (head.right != null || head.down.right != null) {
            break;
        }
        Node p = head;
        head = head.down;
        p.down = null;
    }
}
```

删除后，跳表又恢复成正常的结构了：

```
head
 ___
|   | ---------------------------------------------------------> null
  |                                             
 ___               ___               ___               ___
|   | ----------> | 2 | ----------> | 4 | ----------> | 6 | ---> null
```


## 四、跳表的应用场景

跳表设计的目的，是为了快速查找。所以它比较适合这些场景：

- 是有序链表，无序链表用不了
- 多次查询链表，注意是多次，只查一次的话不如直接遍历
- 较少的插入和删除，虽然跳表的插入删除性能不差，但是其目的不在于此

其实本质上，

- 跳表就是二分查找的链表实现版本

所以有序数组二分查找适用的场景，基本适用于有序链表的跳表。


## 参考

> https://leetcode.cn/problems/design-skiplist/
>
> https://blog.csdn.net/yjw123456/article/details/105159817/
>
> https://blog.csdn.net/weixin_45480785/article/details/116293416
>
> https://baijiahao.baidu.com/s?id=1710441201075985657&wfr=spider&for=pc

## 附录

### 跳表接口

```java
/**
 * 跳表接口
 *
 * @author weijiaduo
 * @since 2022/7/28
 */
public interface SkipList<T extends Comparable<T>> {

    /**
     * 搜索指定值是否存在
     *
     * @param value 指定值
     * @return true存在/false不存在
     */
    boolean search(T value);

    /**
     * 添加新值
     *
     * @param value 新值
     */
    void add(T value);

    /**
     * 删除指定值
     *
     * @param value 指定值
     * @return true删除成功/false删除失败
     */
    boolean erase(T value);

}
```

### 链表实现

```java
/**
 * 跳表（链表）
 *
 * @author weijiaduo
 * @since 2022/7/28
 */
public class LinkedSkipList<T extends Comparable<T>> implements SkipList<T> {

    class Node {
        T value;
        Node right;
        Node down;

        public Node(T value, Node right, Node down) {
            this.value = value;
            this.right = right;
            this.down = down;
        }
    }

    /**
     * 默认最大层级
     */
    static final int DEFAULT_MAX_LEVEL = 32;
    /**
     * 随机因子
     */
    static final double FACTOR = 0.25;

    /**
     * 哨兵头节点
     */
    Node head;
    /**
     * 最大层级
     */
    int maxLevels;
    /**
     * 随机数
     */
    Random random = new Random();

    public LinkedSkipList() {
        this(DEFAULT_MAX_LEVEL);
    }

    public LinkedSkipList(int maxLevels) {
        this.maxLevels = maxLevels;
        this.head = new Node(null, null, null);
    }

    @Override
    public boolean search(T value) {
        Node p = head, q = null;
        // 从上往下一直找，直到最底层的原始链表为止
        while (p != null) {
            // 在当前层遍历，直到找到 value 所在区间
            while (p.right != null && value.compareTo(p.right.value) > 0) {
                p = p.right;
            }
            // 记录上一层的转折点
            q = p;
            // 搜索下一层
            p = p.down;
        }
        return q != null && q.right != null && q.right.value.equals(value);
    }

    @Override
    public void add(T value) {
        List<Node> processors = findProcessors(value);
        int levels = processors.size();

        Node newNode = null;
        boolean insertUp = true;
        for (int i = 0; i < levels && insertUp; i++) {
            Node processor = processors.get(i);
            // 新节点指向下一层的同值节点
            newNode = new Node(value, processor.right, newNode);
            // 在当前层插入新节点
            processor.right = newNode;

            // 是否要继续往上一层插入节点
            insertUp = random.nextDouble() < FACTOR;
        }

        // 更新层级
        if (insertUp) {
            levelUp();
        }
    }

    @Override
    public boolean erase(T value) {
        List<Node> processors = findProcessors(value);

        // 验证删除节点是否存在，没有找到则直接返回
        Node target = processors.get(0).right;
        if (target == null || !value.equals(target.value)) {
            return false;
        }

        // 从下往上，逐层删除指定节点
        for (Node processor : processors) {
            // 指定 value 值的节点在每一层都删完了，就跳出循环
            Node delNode = processor.right;
            if (delNode == null || !delNode.value.equals(value)) {
                break;
            }
            processor.right = delNode.right;
        }

        // 更新层级
        levelDown();

        return true;
    }

    /**
     * 提升层级
     */
    private void levelUp() {
        // 最上层只有一个头节点
        head = new Node(null, null, head);
    }

    /**
     * 降低层级
     */
    private void levelDown() {
        while (head.down != null) {
            // 连续 2 层为空时，才降低层级，因为最上层只有 1 个头节点
            if (head.right != null || head.down.right != null) {
                break;
            }
            Node p = head;
            head = head.down;
            p.down = null;
        }
    }

    /**
     * 查找指定值的前置路径
     *
     * @param value 指定值
     * @return 前置路径
     */
    private List<Node> findProcessors(T value) {
        Deque<Node> stack = new ArrayDeque<>();
        Node p = head;
        // 从上往下一直找，直到最底层的原始链表为止
        while (p != null) {
            // 在当前层遍历，直到找到 value 所在区间
            while (p.right != null && value.compareTo(p.right.value) > 0) {
                p = p.right;
            }
            // 记录上一层的转折点
            stack.push(p);
            // 搜索下一层
            p = p.down;
        }

        // 路径是倒序摆放的，最接近值 value 的在索引 0 的位置
        List<Node> path = new ArrayList<>(stack.size());
        while (!stack.isEmpty()) {
            path.add(stack.pop());
        }
        return path;
    }

}
```

### 数组实现

```java
/**
 * 跳表（数组实现）
 *
 * @author weijiaduo
 * @since 2022/7/27
 */
public class ArraySkipList<T extends Comparable<T>> implements SkipList<T> {

    class Node {
        T value;
        List<Node> forwards;

        Node(T value, int levels) {
            this.value = value;
            forwards = new ArrayList<>(levels);
            for (int i = 0; i < levels; i++) {
                forwards.add(null);
            }
        }
    }

    /**
     * 默认最大层级
     */
    static final int DEFAULT_MAX_LEVEL = 32;
    /**
     * 随机因子
     */
    static final double FACTOR = 0.25;

    /**
     * 哨兵头节点
     */
    Node head;
    /**
     * 当前层级
     */
    int levels = 0;
    /**
     * 最大层级
     */
    int maxLevels;
    /**
     * 随机数
     */
    Random random = new Random();

    public ArraySkipList() {
        this(DEFAULT_MAX_LEVEL);
    }

    public ArraySkipList(int maxLevels) {
        this.maxLevels = maxLevels;
        head = new Node(null, maxLevels);
    }

    /**
     * {@inheritDoc}
     */
    @Override
    public boolean search(T value) {
        Node p = head, q = null;
        // 从上往下一直找，直到最底层的原始链表为止
        for (int i = levels - 1; i >= 0; i--) {
            // 在当前层遍历，直到找到 value 所在区间
            Node right = p.forwards.get(i);
            while (right != null && value.compareTo(right.value) > 0) {
                p = right;
                right = p.forwards.get(i);
            }
            // 记录上一层的转折点
            q = p;
        }
        if (q != null) {
            Node target = q.forwards.get(0);
            return target != null && value.equals(target.value);
        }
        return false;
    }

    /**
     * {@inheritDoc}
     */
    @Override
    public void add(T value) {
        List<Node> processors = findProcessors(value);
        int lv = randomLevel();
        levels = Math.max(levels, lv);
        Node newNode = new Node(value, levels);
        for (int i = 0; i < lv; i++) {
            Node processor = processors.get(i);
            newNode.forwards.set(i, processor.forwards.get(i));
            processor.forwards.set(i, newNode);
        }
    }

    /**
     * {@inheritDoc}
     */
    @Override
    public boolean erase(T value) {
        List<Node> processors = findProcessors(value);
        Node target = processors.get(0).forwards.get(0);
        if (target == null || !value.equals(target.value)) {
            return false;
        }

        // 从下往上，逐层删除指定节点
        for (int i = 0; i < levels; i++) {
            Node processor = processors.get(i);
            Node delNode = processor.forwards.get(i);
            if (delNode == null || delNode != target) {
                break;
            }
            processor.forwards.set(i, delNode.forwards.get(i));
        }

        // 降低层级，最顶层只有头节点时
        for (; levels > 1; levels--) {
            if (head.forwards.get(levels - 1) != null) {
                break;
            }
        }

        return true;
    }

    /**
     * 查找指定值的前置路径
     *
     * @param value 指定值
     * @return 前置路径
     */
    private List<Node> findProcessors(T value) {
        List<Node> path = new ArrayList<>(maxLevels);
        for (int i = 0; i < maxLevels; i++) {
            path.add(head);
        }
        Node p = head;
        for (int i = levels - 1; i >= 0; i--) {
            // 在当前层遍历，直到找到 value 所在区间
            Node right = p.forwards.get(i);
            while (right != null && value.compareTo(right.value) > 0) {
                p = right;
                right = p.forwards.get(i);
            }
            // 记录上一层的转折点
            path.set(i, p);
        }
        return path;
    }

    /**
     * 随机层级
     * <p>
     * 这个随机有点影响时间
     *
     * @return 随即层级
     */
    private int randomLevel() {
        int lv = 1;
        while (lv < maxLevels && random.nextDouble() < FACTOR) {
            lv++;
        }
        return lv;
    }

}
```
