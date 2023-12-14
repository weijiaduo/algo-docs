---
title: B-树
date: 2023-01-12 02:31:30
categories: [数据结构与算法, 数据结构]
tags: [B-树, 数据结构, 算法]
thumbnail: /img/structure.jpg
top: true
---

# B-树

## 一、B-树是什么？

B-树是一棵所有叶子节点高度都相同的平衡多叉树。

其结构类似这样：

```
                                    ____ ___ ___
                                   | 24 |   |   |  
                                          |
                         ---------------------------------------------
                         |                                           |
                   ____ ____ ___                               ____ ___ ___
                  | 15 | 20 |   |                             | 33 |   |   | 
                         |                                           |
        -----------------------------------                 -------------------
        |                |                |                 |                 |
 ____ ____ ___     ____ ___ ___     ____ ____ ___     ____ ____ ___     ____ ___ ___
| 10 | 12 |   |   | 18 |   |   |   | 21 | 23 |   |   | 30 | 31 |   |   | 38 |   |   |
```

B-树的一些性质：

<!--more-->

- 树节点中，子节点数量的最大值称为B-树的阶
- m 阶B-树的节点，最多拥有 m - 1 个元素
- B-树的所有叶子节点都在同一层，是高度平衡的
- 除根节点外，所有节点的元素个数不少于 Math.ceil(m/2) - 1，也即子节点数量不少于 Math.ceil(m/2）
- 节点的大小，始终满足 `左子节点 < 父节点 < 右子节点`，即和二叉查找树类似

简单来说，B-树的结构是节点的子节点数量要半满，叶子节点的高度要始终一样。

## 二、为什么要用B-树？

- B-树或B-树变体被广泛应用于需要进行插入、删除、范围检索等的应用程序，比如 MySQL 数据库索引
- 更新和检索操作只影响部分节点，对于基于磁盘的检索，可以减少 I/O 请求，提升性能
- B-树保证至少有一定比例的节点是满的，能改进空间利用率

## 三、怎么实现一棵B-树？

B-树主要包含几种操作：

- 检索元素：在树中搜索指定的元素
- 插入元素：往树中插入新的元素
- 删除元素：从树中删除指定的元素

其中插入和删除会对B-树的结构产生影响，稍微麻烦亿点。

### 3.1 检索元素

B-树和二叉搜索树类似，二叉搜索树使用的是二分查找，而B-树也是类似的：

- B-树检索元素采用的是二分查找的升级版本 —— 多路查找

比如说，上面的B-树中，要寻找元素 `23`，那么它的搜索路线就是：

- `24 -> 20 -> 23`

在B-树中查找的话，就是这样子：

```
                                    ____ ___ ___
                                   | 24 |   |   |
                                          |
                         ---------------------------------------------
                         |                                           |
                   ____ ____ ___                               ____ ___ ___
                  |    | 20 |   |                             |    |   |   |
                         |                                           |
        -----------------------------------                 -------------------
        |                |                |                 |                 |
 ____ ____ ___     ____ ___ ___     ____ ____ ___     ____ ____ ___     ____ ___ ___
|    |    |   |   |    |   |   |   |    | 23 |   |   |    |    |   |   |    |   |   |
```

对应到伪代码的话，差不多是这样的：

```java
private V get(BTNode<K, V> root, K key) {
    int index = root.findIndex(key);
    if (key.equals(root.getKey(index))) {
        // 找到 key 对应的节点
        return root.getValue(index);
    } else {
        // 往子树继续找 key 对应的节点
        return get(root.getChild(index), key);
    }
}
```

实际就是从上往下多路递归查找元素。

### 3.2 插入元素

B-树和二叉搜索树的插入类似：

- B-树插入新元素，始终是在叶子节点插入

插入元素，可以分为 2 种情况处理：

1. 节点空间足够，直接插入元素
2. 节点空间不足，1 个节点分裂成 3 个节点，执行上溢操作

这 2 种情况的效果如下。

#### 3.2.1 叶子节点-直接插入

当叶子节点空间足够时，可以直接插入新元素，而且不会影响到B-树的高度平衡：

```
 ____ ____ ___ 
| 12 |    |   | 

       | 插入 20
       v

 ____ ____ ___ 
| 12 | 20 |   | 

       | 插入 9
       v

 ____ ____ ____ 
| 9  | 12 | 20 | 
```

空间足够时，直接插入元素就行了。

#### 3.2.2 叶子节点-上溢分裂

当叶子节点空间不足时，此时再插入新元素，会导致节点空间上溢，然后分裂：

```
             ___ ____ ____ 
            | 9 | 12 | 20 | 

                  | 插入 25
                  v

            ____ ___ ___ 
           | 20 |   |   | 
                  |
        ---------------------
        |                   |
   ___ ____ ___       ____ ____ ____ 
  | 9 | 12 |   |     | 25 |    |    | 
```

节点元素上溢后，1 个节点会分裂成 3 个节点。

#### 3.2.3 内部节点-直接插入

为了保持B-树的高度平衡，叶子节点会将分裂后的父节点，往上插入之前的父节点：

```
                       ____ ___ ___ 
                      | 20 |   |   |
                             |
                  ----------------------
                  |                    |
             ___ ____ ____       ____ ____ ____ 
            | 9 | 12 | 18 |     | 25 |    |    |


                             |  插入 10
                             v 

                       ____ ____ ___ 
                      | 12 | 20 |   |
                             |
        --------------------------------------------
        |                    |                     |
   ___ ____ ____       ____ ____ ____        ____ ____ ____ 
  | 9 | 10 |    |     | 18 |    |    |      | 25 |    |    |
```

左子节点 `[9, 12, 18]` 插入新节点 `10` 会产生上溢，节点 `12` 会上溢父节点中。

因为内部节点 `[20]` 空间足够，所以 `12` 直接插入即可。

#### 3.2.4 内部节点-分裂上溢

子节点上溢后，如果父节点空间也满了，那么父节点也会出现分裂上溢：

```
                                             ____ ____ ____ 
                                            | 20 | 30 | 45 | 
                                                   |
                  -------------------------------------------------------------------
                  |                    |                     |                      |
             ___ ____ ____       ____ ____ ____        ____ ____ ____         ____ ____ ____ 
            | 9 | 12 | 18 |     | 25 |    |    |      | 37 | 39 |    |       | 62 | 85 |    | 


                                                   |  插入 10
                                                   v 

                                             ____ ____ ____ 
                                            | 30 |    |    | 
                                                   |
                             ----------------------------------------------------------
                             |                                                        |
                       ____ ____ ___                                            ____ ___ ___ 
                      | 12 | 20 |   |                                          | 45 |   |   | 
                             |                                                        |
        --------------------------------------------                      ------------------------
        |                    |                     |                      |                      |
   ___ ____ ____       ____ ____ ____        ____ ____ ____         ____ ____ ____         ____ ____ ____
  | 9 | 10 |    |     | 18 |    |    |      | 25 |    |    |       | 37 | 39 |    |       | 62 | 85 |    | 
```

插入元素 `10` 之后，会传递性地导致节点 `[9, 12, 18]` 和节点 `[20, 30, 45]` 都产生分裂上溢。

这种分裂上溢可以一直持续到根节点，最后可能会导致B-树提升一层（如上）。

上述的 4 种情况，实际就 2 种：

1. 空间足够时，直接插入
2. 空间不足时，分裂上溢

对应的伪代码类似这样：

```java
/**- 插入新元素 -*/
private BTNode<K, V> put(BTNode<K, V> root, K key, V value) {
    int index = root.findIndex(key);
    
    // 叶子节点
    if (root.isLeaf()) {
        // 叶子节点添加元素后，可能发生了分裂
        return root.add(key, value);
    }

    // 内部节点
    BTNode<K, V> child = root.getChild(index);
    child = put(child, key, value);
    // 子树添加元素后，可能发生了分裂
    return root.overflow(index, child);
}
```

```java
/**- 直接插入和分裂上溢（叶子节点和内部节点的插入逻辑一样） -*/
public BTNode<K, V> overflow(int index, BTNode<K, V> node) {
    // 子节点没上溢
    if (getChild(index) == node) {
        return this;
    }

    // 子节点上溢了
    // 1. 空间足够，直接插入
    if (size + node.size() < m) {
        return insertNode(index, node);
    }

    // 2. 空间不足，分裂节点
    return splitNodes(index, node);
}
```

### 3.3 删除元素

删除元素，可以分为 3 种情况：

1. 删除后元素数量仍然满足要求，直接删除
2. 删除后元素数量不满足要求，看兄弟节点是否可以借一个过来
3. 兄弟节点不能借，那就将父子节点合并成 1 个节点，产生下溢

这几种情况如下。

#### 3.3.1 直接删除

如果节点删除元素后，元素数量仍然处于半满状态，则直接返回即可：

```
                       ____ ___ ___ 
                      | 20 |   |   | 
                             |
                  ----------------------
                  |                    |
             ___ ____ ____       ____ ____ ____ 
            | 9 | 12 | 18 |     | 25 |    |    | 


                             |  删除 12
                             v 

                       ____ ___ ___ 
                      | 20 |   |   | 
                             |
                  ----------------------
                  |                    |
             ___ ____ ____       ____ ____ ____ 
            | 9 | 18 |    |     | 25 |    |    | 
```

叶子节点 `[9, 12, 18]` 删除 `12` 以后，元素数量依旧满足半满状态，则可以直接删除并返回。

#### 3.3.2 从兄弟借

如果删除元素后，元素数量少于一半，不满足要求，则看一下兄弟节点能不能借一下：

```
                       ____ ___ ___ 
                      | 20 |   |   | 
                             |
                  ----------------------
                  |                    |
             ___ ____ ____       ____ ____ ____ 
            | 9 | 12 | 18 |     | 25 |    |    | 


                             |  删除 25
                             v 

                       ____ ___ ___ 
                      | 18 |   |   | 
                             |
                  ----------------------
                  |                    |
             ___ ____ ____       ____ ____ ____ 
            | 9 | 12 |    |     | 20 |    |    | 
```

右子节点 `[25]` 删除 `25` 以后，元素数量不满足要求了，找兄弟节点 `[9, 12, 18]` 借一个。

但是不能直接把左子节点的元素移到右子节点中，因为B-树是要求满足 `左子节点 < 父节点 < 右子节点`。

所以“借用”实际上是一种旋转操作，父节点转到需要借用元素的节点，而兄弟节点元素则转到父节点中。

#### 3.3.3 父子合并

如果兄弟节点借不了元素，那就将`父节点 + 左子节点 + 右子节点`合并在一起：

```
                       ____ ___ ___ 
                      | 20 |   |   | 
                             |
                  ----------------------
                  |                    |
             ___ ____ ____       ____ ____ ____ 
            | 9 |    |    |     | 25 |    |    | 


                             |  删除 25
                             v 

                        ___ ____ ___ 
                       | 9 | 20 |   | 
```

删除元素 `25` 后，由于兄弟节点没办法借用，所以将父子节点都合并成 1 个节点，这个合并称为下溢。

下溢和上溢一样，也可能会持续下溢，一直持续到根节点，最终导致树的层级下降。

上述情况的伪代码如下：

```java
/**- 删除元素 -*/
private BTNode<K, V> remove(BTNode<K, V> root, K key) {
    int index = root.findIndex(key);

    // 找到对应节点后移除元素
    if (key.equals(root.getKey(index))) {
        // 直接删除元素后，可能导致父节点下溢
        return root.delete(index);
    }

    // 在子树内递归移除元素
    remove(root.getChild(index), key);
    // 子树删除元素后，可能导致父节点下溢
    return root.underflow(index);
}
```
```java
/** 从兄弟借和父子合并（叶子节点和内部节点的下溢逻辑是一样的） */
public BTNode<K, V> underflow(int index) {
    // 子节点合法，无需下溢
    if (isLegal(getChild(index))) {
        return this;
    }

    // 子节点不合法，需修正
    // 1. 从兄弟节点借一个元素
    if (borrowLeft(index) || borrowRight(index)) {
        return this;
    }

    // 2. 合并父元素 + 左右子节点
    return mergeNodes(index);
}
```

## 总结

- 性质
    - m 阶B-树的节点，最多拥有 m - 1 个元素
    - B-树的所有叶子节点都在同一层，是高度平衡的
    - 除根节点外，所有节点的元素个数不少于 Math.ceil(m/2) - 1，也即子节点数量不少于 Math.ceil(m/2）
    - 节点的大小，始终满足 `左子节点 < 父节点 < 右子节点`
- 检索
    - 多路查找：二分查找的升级版本
- 插入
    1. 空间足够，直接插入
    2. 空间不足，分裂上溢
- 删除
    1. 元素够多，直接删除
    2. 元素不够，从兄弟借
    3. 兄弟节点无法借用，合并父子节点，执行下溢


## 参考

> 《数据结构与算法分析（第三版）》
>
> https://www.cnblogs.com/nullzx/p/8729425.html
>
> https://github.com/LiuXianghai-coder/Test-Repo/blob/master/DataStructure/BTree.java
>
> https://blog.csdn.net/u011711997/article/details/80420392
>
> https://blog.csdn.net/c630843901/article/details/121423196
>
> https://blog.csdn.net/oooooorz/article/details/112297554
>
>https://blog.csdn.net/cdnight/article/details/11772621
>

## 附录

### B-树接口

```java
/**
 * B-树
 *
 * @author weijiaduo
 * @since 2023/1/2
 */
public interface BTree<K extends Comparable<K>, V> {

    /**
     * 查找指定 key 的值
     *
     * @param key key
     * @return value
     */
    V get(K key);

    /**
     * 添加指定 key-value 的节点
     *
     * @param key   key
     * @param value value
     */
    void put(K key, V value);

    /**
     * 删除指定 key 的节点
     *
     * @param key key
     */
    void remove(K key);

}
```

### B-树实现

```java
/**
 * B-树实现类
 *
 * @author weijiaduo
 * @since 2023/1/2
 */
public class BTreeImpl<K extends Comparable<K>, V> implements BTree<K, V> {

    /**
     * 根节点
     */
    private BTNode<K, V> root;
    /**
     * m 阶 B 树
     */
    private final int m;

    public BTreeImpl(int m) {
       this.m = m;
    }

    @Override
    public V get(K key) {
        return get(root, key);
    }

    /**
     * 从当前节点开始找指定 key 的值
     *
     * @param root 当前根节点
     * @param key  key
     * @return value
     */
    private V get(BTNode<K, V> root, K key) {
        if (root == null) {
            return null;
        }

        // 查找当前 key 所在的位置
        int index = root.findIndex(key);
        if (key.equals(root.getKey(index))) {
            // 找到 key 对应的节点
            return root.getValue(index);
        } else {
            // 往子树继续找 key 对应的节点
            return get(root.getChild(index), key);
        }
    }

    @Override
    public void put(K key, V value) {
        root = put(root, key, value);
    }

    /**
     * 在当前树内添加指定的 key-value
     *
     * @param root  当前根节点
     * @param key   key
     * @param value value
     * @return 新的根节点
     */
    private BTNode<K, V> put(BTNode<K, V> root, K key, V value) {
        // 1. 整棵树为空时，插入新节点
        if (root == null) {
            BTNode<K, V> node = new BTNode<>(m);
            return node.add(key, value);
        }

        // 验证是否已存在，存在则只需更新
        int index = root.findIndex(key);
        if (key.equals(root.getKey(index))) {
            // 更新节点
            root.setValue(index, value);
            return root;
        }

        // 2. 叶子节点，直接添加
        if (root.isLeaf()) {
            // 添加到叶子节点中，有可能发生分裂
            return root.add(key, value);
        }

        // 3. 内部节点，添加到子树的叶子节点中
        BTNode<K, V> child = root.getChild(index);
        child = put(child, key, value);

        // 添加元素后子树可能发生了分裂
        return root.overflow(index, child);
    }

    @Override
    public void remove(K key) {
        root = remove(root, key);
        if (root != null && root.isEmpty()) {
            // 树的高度下降
            root = root.firstChild();
        }
    }

    /**
     * 在当前树内删除指定 key 的节点
     *
     * @param root 当前根节点
     * @param key  key
     * @return 新的根节点
     */
    private BTNode<K, V> remove(BTNode<K, V> root, K key) {
        if (root == null) {
            return null;
        }

        // 找到对应节点后移除元素
        int index = root.findIndex(key);
        if (key.equals(root.getKey(index))) {
            return root.delete(index);
        }

        // 在子树内递归移除元素
        remove(root.getChild(index), key);

        // 移除后可能需要父节点下溢
        return root.underflow(index);
    }

}
```

### B-树节点

```java
/**
 * B-树节点
 *
 * @author weijiaduo
 * @since 2023/1/2
 */
public class BTNode<K extends Comparable<K>, V> {

    /**
     * m 阶 B 树
     */
    private final int m;
    /**
     * 节点元素阈值
     */
    private final int threshold;
    /**
     * 节点元素数组
     * <p>
     * 元素的第 0 位始终是一个占位元素，专门用于存放最左子节点
     * <p>
     * 所以一个节点能拥有的元素个数最多是 m - 1
     */
    private final Object[] elements;
    /**
     * 实际元素数量
     * <p>
     * 取值范围是 [0, m - 1]
     */
    private int size;

    public BTNode(int m) {
        this.m = m;
        this.threshold = half(m);
        elements = new Object[m];
        // 最左元素占位，用于存放最左子节点指针
        elements[0] = new Entry<K, V>(null, null);
        size = 0;
    }

    /**
     * @return 元素数量
     */
    public int size() {
        return size;
    }

    /**
     * 半数阈值 ceil(m / 2) - 1
     *
     * @param m 当前值
     * @return 半数阈值
     */
    private int half(int m) {
        return (m + 1) / 2 - 1;
    }

    /**
     * 是否已满
     *
     * @return true已满/false未满
     */
    public boolean isFull() {
        return size == m - 1;
    }

    /**
     * 是否为空
     *
     * @return true为空/false非空
     */
    public boolean isEmpty() {
        return size == 0;
    }

    /**
     * 节点是否合法
     * <p>
     * 节点的元素数量要求大于等于阈值
     *
     * @return true合法/false非法
     */
    public boolean isLegal() {
        return size >= threshold;
    }

    /**
     * 是否可借用元素给别的节点
     * <p>
     * 当节点的元素数量大于阈值时，才可以外借元素
     *
     * @return true可借用/false不可借用
     */
    private boolean canBorrow() {
        return size > threshold;
    }

    /**
     * 是否是叶子节点
     * <p>
     * 没有任何一个非空子节点时才是叶子节点
     *
     * @return true叶子节点/false内部节点
     */
    public boolean isLeaf() {
        for (int i = 0; i <= size; i++) {
            if (getChild(i) != null) {
                return false;
            }
        }
        return true;
    }

    /**
     * 获取所有的元素
     *
     * @return 元素集合
     */
    public List<Entry<K, V>> entries() {
        List<Entry<K, V>> entries = new ArrayList<>(size);
        // 元素从索引 1 开始
        for (int i = 1; i <= size; i++) {
            entries.add(getEntry(i));
        }
        return entries;
    }

    /**
     * 获取所有的 key
     *
     * @return key 集合
     */
    public List<K> keys() {
        List<K> keys = new ArrayList<>(size);
        // 元素从索引 1 开始
        for (int i = 1; i <= size; i++) {
            keys.add(getKey(i));
        }
        return keys;
    }

    /**
     * 获取所有的 value
     *
     * @return value 集合
     */
    public List<V> values() {
        List<V> values = new ArrayList<>(size);
        // 元素从索引 1 开始
        for (int i = 1; i <= size; i++) {
            values.add(getValue(i));
        }
        return values;
    }

    /**
     * 获取指定位置的 key
     *
     * @param index 索引
     * @return key
     */
    public K getKey(int index) {
        return getEntry(index).key;
    }

    /**
     * 获取指定位置的 value
     *
     * @param index 索引
     * @return value
     */
    public V getValue(int index) {
        return getEntry(index).value;
    }

    /**
     * 设置指定位置的 value
     *
     * @param index 索引
     * @param value 值
     */
    public void setValue(int index, V value) {
        getEntry(index).value = value;
    }

    /**
     * 获取所有的子节点
     * <p>
     * 注意：是所有可能的子节点，也包括了 null 子节点
     *
     * @return 子节点集合
     */
    public List<BTNode<K, V>> children() {
        List<BTNode<K, V>> children = new ArrayList<>(size + 1);
        // 子节点从索引 0 开始
        for (int i = 0; i <= size; i++) {
            children.add(getChild(i));
        }
        return children;
    }

    /**
     * 返回第一个子节点
     * <p>
     * 注意：不是第一个非空子节点，而是第 1 个元素的左子节点
     *
     * @return 第一个子节点
     */
    public BTNode<K, V> firstChild() {
        return getChild(0);
    }

    /**
     * 返回最后一个子节点
     * <p>
     * 注意：不是最后一个非空子节点，而是最后 1 个元素的右子节点
     *
     * @return 最后一个子节点
     */
    public BTNode<K, V> lastChild() {
        return getChild(size);
    }

    /**
     * 获取指定位置的下一层子节点
     *
     * @param index 指定位置
     * @return 下一层子树根节点
     */
    public BTNode<K, V> getChild(int index) {
        return getEntry(index).pointer;
    }

    /**
     * 更新指定位置的子节点
     *
     * @param index 索引
     * @param node  子节点
     */
    public void setChild(int index, BTNode<K, V> node) {
        getEntry(index).pointer = node;
    }

    /**
     * 查找指定 key 的位置
     * <p>
     * 返回最后一个小于等于 key 的位置
     *
     * @param key key
     * @return key 的位置
     */
    public int findIndex(K key) {
        // 第 0 位是占位元素，还没算在内
        return 1 + binaryle(keys(), key);
    }

    /**
     * 二分查找，返回最后一个小于等于 key 的位置
     *
     * @param keys key 集合
     * @param key  指定 key
     * @return 最后一个小于等于 key 的位置/-1
     */
    private int binaryle(List<K> keys, K key) {
        int size = keys.size();
        int l = 0, r = size - 1;
        while (l <= r) {
            int m = l + (r - l) / 2;
            K k = keys.get(m);
            if (k.compareTo(key) <= 0) {
                if (m == size - 1 || keys.get(m + 1).compareTo(key) > 0) {
                    return m;
                }
                l = m + 1;
            } else {
                r = m - 1;
            }
        }
        return -1;
    }

    /**
     * 添加新元素
     *
     * @param key   key
     * @param value value
     */
    public BTNode<K, V> add(K key, V value) {
        BTNode<K, V> node = new BTNode<>(m);
        node.addEntry(new Entry<>(key, value));
        int index = findIndex(key);
        return overflow(index, node);
    }

    /**
     * 节点上溢处理
     * <p>
     * 子树添加新元素后，可能会分裂成新节点，替代原有的子节点
     *
     * @param index 索引
     * @param node  新节点
     * @return 添加后的当前节点
     */
    public BTNode<K, V> overflow(int index, BTNode<K, V> node) {
        if (node == null || node.isEmpty()) {
            return this;
        }

        // 还是旧节点，说明子节点没有上溢，无需处理
        BTNode<K, V> cur = getChild(index);
        if (cur == node) {
            return this;
        }

        // 1. 当前空间足够插入
        if (size + node.size() < m) {
            // 新子节点的左子节点替代旧子节点的位置
            setChild(index, node.firstChild());
            for (Entry<K, V> entry : node.entries()) {
                insertEntry(++index, entry);
            }
            return this;
        }

        // 2. 当前空间不够，需要分裂成 3 个节点
        return splitNodes(index, node);
    }

    /**
     * 子节点上溢后，父节点已满，则需要分裂节点，将 1 个节点拆分成 3 个节点
     *
     * @param index   上溢子节点的父元素索引
     * @param newNode 上溢的子节点
     * @return 分裂后新的父节点
     */
    private BTNode<K, V> splitNodes(int index, BTNode<K, V> newNode) {
        // 新子节点的左子节点替代旧子节点的位置
        setChild(index, newNode.firstChild());

        int newSize = size + newNode.size();
        List<Entry<K, V>> allEntries = new ArrayList<>(newSize);
        List<Entry<K, V>> curEntries = entries();
        for (int i = 0; i < index && i < size; i++) {
            allEntries.add(curEntries.get(i));
        }
        allEntries.addAll(newNode.entries());
        for (int i = index; i < size; i++) {
            allEntries.add(curEntries.get(i));
        }

        // 根节点
        int mid = newSize / 2;
        BTNode<K, V> root = new BTNode<>(m);
        for (int i = mid; i <= mid; i++) {
            root.addEntry(allEntries.get(i));
        }
        // 左子节点
        BTNode<K, V> left = new BTNode<>(m);
        for (int i = 0; i < mid; i++) {
            left.addEntry(allEntries.get(i));
        }
        // 右子节点
        BTNode<K, V> right = new BTNode<>(m);
        for (int i = mid + 1; i < newSize; i++) {
            right.addEntry(allEntries.get(i));
        }

        // 边界指针
        left.setChild(0, firstChild());
        right.setChild(0, root.lastChild());
        root.setChild(0, left);
        root.setChild(root.size, right);
        return root;
    }

    /**
     * 删除指定位置的元素
     * <p>
     * 删除元素后，节点可能会产生下溢，从而返回了一个空元素节点
     *
     * @param index 索引
     * @return 删除后的根节点
     */
    public BTNode<K, V> delete(int index) {
        if (isEmpty() || index == 0) {
            throw new IllegalStateException(String.format("index: %d, size: %d", index, size));
        }

        if (isLeaf()) {
            // 叶子节点
            // 直接删除，删除后可能会变成元素为空的空节点
            // 空节点将由父节点的 underflow 处理掉，或者由根节点处理掉
            removeEntry(index);
            return this;
        }

        // 内部节点
        // 使用前驱或后驱进行替换
        int rpIndex = getReplacer(this, index);
        BTNode<K, V> child = getChild(rpIndex);
        BTNode<K, V> newChild;
        if (rpIndex < index) {
            // 前驱元素替换
            Entry<K, V> max = max(child);
            newChild = removeMax(child);
            setChild(rpIndex, newChild);
            setEntry(index, max);
        } else {
            // 后驱元素替换
            Entry<K, V> min = min(child);
            newChild = removeMin(child);
            setChild(rpIndex, newChild);
            setEntry(index, min);
        }

        // 移除后可能需要父节点下溢
        return underflow(rpIndex);
    }

    /**
     * 节点下溢处理
     * <p>
     * 存在非法子节点时，可能需要对父节点进行下溢操作
     *
     * @param index 父节点索引
     * @return 根节点
     */
    public BTNode<K, V> underflow(int index) {
        // 验证子节点是否合法
        BTNode<K, V> cur = getChild(index);
        if (cur == null || cur.isLegal()) {
            return this;
        }

        // 1. 从兄弟节点借一个元素
        BTNode<K, V> left = null;
        if (index > 0) {
            left = getChild(index - 1);
            if (left != null && left.canBorrow()) {
                return borrowLeft(index);
            }
        }
        BTNode<K, V> right = null;
        if (index < size) {
            right = getChild(index + 1);
            if (right != null && right.canBorrow()) {
                return borrowRight(index);
            }
        }
        if (left == null && right == null) {
            return this;
        }

        // 2. 合并父元素 + 左右子节点
        // 始终把当前节点作为合并时的右子节点
        return mergeNodes(left != null ? index : index + 1);
    }

    /**
     * 借用左子节点的值
     * <p>
     * 实际就是右旋，将父节点转到右子节点，左子节点的元素转到父节点
     *
     * @param index 父节点索引
     */
    private BTNode<K, V> borrowLeft(int index) {
        BTNode<K, V> left = getChild(index - 1);
        BTNode<K, V> right = getChild(index);

        Entry<K, V> parentEntry = getEntry(index);
        Entry<K, V> leftEntry = left.getEntry(left.size);
        BTNode<K, V> leftRight = left.lastChild();

        // 父节点转到右子节点
        right.insertEntry(1, parentEntry);
        right.setChild(1, right.firstChild());
        // 左子节点转到父节点
        left.removeEntry(left.size);
        setEntry(index, leftEntry);
        setChild(index, right);
        right.setChild(0, leftRight);
        return this;
    }

    /**
     * 借用右子节点的值
     * <p>
     * 实际就是左旋，将父元素转到左子节点，右子节点的元素转到父节点
     *
     * @param index 父节点索引
     */
    private BTNode<K, V> borrowRight(int index) {
        BTNode<K, V> right = getChild(index + 1);
        BTNode<K, V> left = getChild(index);

        Entry<K, V> parentEntry = getEntry(index + 1);
        Entry<K, V> rightEntry = right.getEntry(1);
        BTNode<K, V> rightLeft = right.firstChild();

        // 父节点转到左子节点
        left.addEntry(parentEntry);
        // 右子节点转到父节点
        right.setChild(0, right.getChild(1));
        right.removeEntry(1);
        setEntry(index + 1, rightEntry);
        setChild(index + 1, right);
        left.setChild(left.size, rightLeft);
        return this;
    }

    /**
     * 父节点元素 + 右子节点，全都合并到左子节点
     * <p>
     * 因为父节点元素和右子节点是一一对应的，所以都是按照“父 + 右 --> 左”进行合并
     * <p>
     * 方便同时删除父节点元素和右子节点
     *
     * @param index 父元素索引
     */
    private BTNode<K, V> mergeNodes(int index) {
        BTNode<K, V> left = getChild(index - 1);
        BTNode<K, V> right = getChild(index);

        // 父节点元素合并到左子节点
        Entry<K, V> parentEntry = getEntry(index);
        left.addEntry(parentEntry);

        // 右子节点合并到左子节点
        left.setChild(left.size, right.firstChild());
        for (Entry<K, V> entry : right.entries()) {
            left.addEntry(entry);
        }

        // 移除父节点元素
        removeEntry(index);
        return this;
    }

    /**
     * 获取可替换子节点索引（前驱/后驱）
     *
     * @param root  当前根节点
     * @param index 当前索引
     * @return 可替换子节点索引
     */
    private int getReplacer(BTNode<K, V> root, int index) {
        BTNode<K, V> right = null;
        if (index <= root.size) {
            right = root.getChild(index);
        }
        if (right == null) {
            return index - 1;
        }
        BTNode<K, V> left = null;
        if (index > 0) {
            left = root.getChild(index - 1);
        }
        if (left == null) {
            return index;
        }

        if (left.canBorrow()) {
            return index - 1;
        } else {
            return index;
        }
    }

    /**
     * 获取最大值
     *
     * @param root 当前根节点
     * @return 最大值
     */
    private Entry<K, V> max(BTNode<K, V> root) {
        if (root == null) {
            return null;
        }

        if (root.isLeaf()) {
            return root.getEntry(root.size);
        }
        return max(root.lastChild());
    }

    /**
     * 移除最大值
     *
     * @param root 当前根节点
     * @return 移除后的根节点
     */
    private BTNode<K, V> removeMax(BTNode<K, V> root) {
        if (root == null) {
            return null;
        }

        if (root.isLeaf()) {
            root.removeEntry(root.size);
            return root;
        }

        int index = root.size;
        removeMax(root.getChild(index));
        return root.underflow(index);
    }

    /**
     * 获取最小值
     *
     * @param root 当前根节点
     * @return 最小值
     */
    private Entry<K, V> min(BTNode<K, V> root) {
        if (root == null) {
            return null;
        }

        if (root.isLeaf()) {
            return root.getEntry(1);
        }
        return min(root.firstChild());
    }

    /**
     * 移除最小值
     *
     * @param root 当前根节点
     * @return 移除值后的根节点
     */
    private BTNode<K, V> removeMin(BTNode<K, V> root) {
        if (root == null) {
            return null;
        }

        if (root.isLeaf()) {
            root.removeEntry(1);
            return root;
        }

        int index = 0;
        removeMin(root.getChild(index));
        return root.underflow(index);
    }

    /**
     * 获取指定位置的元素
     *
     * @param index 索引
     * @return 元素
     */
    private Entry<K, V> getEntry(int index) {
        return (Entry<K, V>) elements[index];
    }

    /**
     * 更新指定位置的元素
     * <p>
     * 要求原始元素必须存在，否则应该使用 {@code insertEntry()}
     *
     * @param index 索引
     * @param entry 元素
     */
    private void setEntry(int index, Entry<K, V> entry) {
        Entry<K, V> oldEntry = getEntry(index);
        if (oldEntry == null || index == 0) {
            throw new IllegalStateException(String.format("index: %d, size: %d", index, size));
        }

        // 保留子节点，只替换元素的 key-value
        elements[index] = entry;
        entry.pointer = oldEntry.pointer;
    }

    /**
     * 追加新元素
     * <p>
     * 此方法不会产生节点分裂，需要分裂时应调用 {@code overflow()} 方法
     *
     * @param entry 元素
     */
    private void addEntry(Entry<K, V> entry) {
        if (isFull()) {
            throw new IllegalStateException(String.format("size: %d", size));
        }

        elements[++size] = entry;
    }

    /**
     * 插入新元素
     * <p>
     * 此方法不会产生节点分裂，需要分裂时应调用 {@code overflow()} 方法
     *
     * @param index 插入索引
     * @param entry 元素
     */
    private void insertEntry(int index, Entry<K, V> entry) {
        if (isFull() || index == 0) {
            throw new IllegalStateException(String.format("index: %d, size: %d", index, size));
        }

        if (size >= index) {
            System.arraycopy(elements, index, elements, index + 1, size - index + 1);
            elements[index] = entry;
            size++;
        } else {
            elements[index] = entry;
            size = index;
        }
    }

    /**
     * 移除指定位置的元素
     *
     * @param index 索引
     */
    private void removeEntry(int index) {
        if (isEmpty() || index == 0) {
            throw new IllegalStateException(String.format("index: %d, size: %d", index, size));
        }

        if (size > index) {
            System.arraycopy(elements, index + 1, elements, index, size - index);
        }
        elements[size] = null;
        size--;
    }

    @Override
    public String toString() {
        String[] s = new String[size];
        for (int i = 1; i <= size; i++) {
            s[i - 1] = elements[i].toString();
        }
        return "[" + String.join(", ", s) + "]";
    }

    /**
     * 单个元素
     */
    static class Entry<K extends Comparable<K>, V> {
        /**
         * key
         */
        K key;
        /**
         * value
         */
        V value;
        /**
         * 右子节点指针，即大于 key 的子树
         */
        BTNode<K, V> pointer;

        public Entry(K key, V value) {
            this.key = key;
            this.value = value;
            pointer = null;
        }

        @Override
        public String toString() {
            return "{" + key + " : " + value + "}";
        }
    }

}
```
