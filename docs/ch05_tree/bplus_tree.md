---
title: B+树
date: 2023-01-16 04:39:30
categories: [数据结构与算法, 数据结构]
tags: [B+树, 数据结构, 算法]
thumbnail: /img/structure.jpg
top: true
---

# B+树

## 一、什么是 B+ 树？

B+ 树是 B-树的变种。

其结构类似这样：

```
                                             ____ ___ ___
                                            | 24 |   |   |
                                                   |
                             --------------------------------------------------------
                             |                                                      |
                       ____ ____ ___                                          ____ ___ ___
                      | 15 | 20 |   |                                        | 33 |   |   | 
                             |                                                      |
        --------------------------------------------                     -----------------------
        |                    |                     |                     |                     |
 ____ ____ ___         ____ ____ ___         ____ ____ ___         ____ ____ ___         ____ ____ ___
| 10 | 12 |   |  <=>  | 15 | 18 |   |  <=>  | 20 | 21 |   |  <=>  | 30 | 31 |   |  <=>  | 34 | 38 |   |
```

B+ 树和 B-树的主要区别在于：

<!--more-->

- 内部节点只是索引，只有叶子节点才保存数据
- 叶子节点之间构成了一条双向链表
- 叶子节点包含了所有关键码值，而内部节点仅包含用于索引的关键码值

B+ 树和 B-树的区别主要在于叶子节点，它们的内部节点基本是一样的。


## 二、为什么用 B+ 树？

- B+ 树特别适合范围查询，因为叶子节点之间是双向连接，可以顺着双向链表寻找数据
- B+ 树只会在叶子节点上插入和删除数据，相对于 B-树来说，实现会简单一些


## 三、如何实现 B+ 树？

和 B-树差不多，B+ 树也包含了几种操作：

- 检索元素：在树中搜索指定的元素
- 插入元素：往树中插入新的元素
- 删除元素：从树中删除指定的元素

其中，插入和删除会对 B+ 树的结构产生影响，稍微麻烦。

### 3.1 检索元素

B+ 树的检索和 B-树的差不多：

- 基于多路查找，找到对应的节点
- 需要一直找到叶子节点，因为只有叶子节点有数据

比如，上面的 B+ 树中，要寻找元素 `20`，它的搜索路线就是：

```
                                             ____ ___ ___
                                            | 24 |   |   |
                                                   |
                             --------------------------------------------------------
                             |                                                      |
                       ____ ____ ___                                          ____ ___ ___
                      |    | 20 |   |                                        |    |   |   | 
                             |                                                      |
        --------------------------------------------                     -----------------------
        |                    |                     |                     |                     |
 ____ ____ ___         ____ ____ ___         ____ ____ ___         ____ ____ ___         ____ ____ ___
|    |    |   |  <=>  |    |    |   |  <=>  | 20 |    |   |  <=>  |    |    |   |  <=>  |    |    |   |
```

注意：虽然内部节点也有 `20` 这个关键码值，但是它没有保存数据，仅仅是作为索引，所以还是要一直搜索到叶子节点。

这个检索过程对应的伪代码：

```java
/**- 检索元素 -*/
private V get(BPTNode<K, V> root, K key) {
    int index = root.findIndex(key);
    if (root.isLeaf()) {
        // 叶子节点
        if (key.equals(root.getKey(index))) {
            // 找到 key 对应的节点
            BPTLeaf<K, V> leaf = (BPTLeaf<K, V>) root;
            return leaf.getValue(index);
        } else {
            // 没有找到对应的节点
            return null;
        }
    } else {
        // 内部节点，往子树继续找 key 对应的节点
        return get(root.getChild(index), key);
    }
}
```

B+ 树的检索要一直到达叶子节点才行。

### 3.2 插入元素

B+ 树和 B-树一样：

- 插入节点总是在叶子节点插入

B-树节点的分裂实际只有 2 种情况，但是 B+ 树的叶子节点分裂会有点特殊：

- 叶子节点分裂后，新的左右叶子节点会保留所有的关键码值
- 分裂出来的父节点的关键码值，等于右叶子节点中的最小关键码值

这是因为，B+ 树的数据都在叶子中保存，不能将其上溢到父节点中，要保持这种规则。

由于 B+ 树叶子节点和内部节点的分裂不同，所以会分为 4 种情况：

1. 叶子节点空间足够，直接插入元素
2. 叶子节点空间不足，1 个节点分裂成 3 个节点，执行上溢操作
3. 内部节点空间足够，直接插入元素
4. 内部节点空间不足，1 个节点分裂成 3 个节点，执行上溢操作

这 4 种情况的效果如下。

#### 3.2.1 叶子节点-直接插入

当叶子节点空间足够时，可以直接插入新元素：

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

此时的叶子节点空间足够时，直接插入元素就行了。

#### 3.2.2 叶子节点-分裂上溢

当叶子节点空间不足时，再插入新元素，会导致节点分裂，产生上溢：

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
   ___ ____ ___         ____ ____ ____ 
  | 9 | 12 |   |  <=>  | 20 | 25 |    | 
```

节点空间不足以插入新元素，所以 1 个节点会分裂成 3 个节点。其中，

- 叶子分裂后，新的叶子节点会保留所有的关键码值 `[9, 12]`、`[20, 25]`
- 父节点的关键码值，等于右叶子节点的最小关键码值 `[20]`
- 叶子节点之间，还需要补充上双向链接

在叶子节点分裂上溢这一点上，B+ 树和 B-树有所不同。

#### 3.2.3 内部节点-直接插入

为了保持 B+ 树的高度平衡，叶子节点会将分裂后的父节点，往上插入之前的父节点：

```
                          ____ ___ ___ 
                         | 20 |   |   |
                                |
                    ------------------------
                    |                      |
               ___ ____ ____         ____ ____ ____ 
              | 9 | 12 | 18 |  <=>  | 20 | 25 |    |


                               |  插入 10
                               v 

                         ____ ____ ___ 
                        | 12 | 20 |   | 
                               |
        -----------------------------------------------
        |                      |                      |
   ___ ____ ____         ____ ____ ____         ____ ____ ____ 
  | 9 | 10 |    |  <=>  | 12 | 18 |    |  <=>  | 20 | 25 |    |
```

左子节点 `[9, 12, 18]` 插入新元素 `10` 后会产生上溢，`12` 会上溢父节点 `[20]` 中。

因为内部节点 `[20]` 的空间足够，所以 `12` 直接插入即可。

#### 3.2.4 内部节点-分裂上溢

节点上溢后，如果父节点空间也满了，那么父节点也会出现分裂上溢：

```
                                               ____ ____ ____ 
                                              | 12 | 20 | 45 | 
                                                     |
                      ------------------------------------------------------------------
                      |                   |                     |                      |
                 ___ ___ ____       ____ ____ ____        ____ ____ ____         ____ ____ ____ 
                | 8 | 9 | 10 |     | 12 | 18 |    |      | 20 | 25 |    |       | 45 | 85 |    | 


                                                     |  插入 11
                                                     v 

                                               ____ ____ ____ 
                                              | 20 |    |    | 
                                                     |
                               ------------------------------------------------------------
                               |                                                          |
                         ____ ____ ___                                              ____ ___ ___ 
                        | 10 | 12 |   |                                            | 45 |   |   | 
                               |                                                          |
        ----------------------------------------------                      -------------------------
        |                      |                     |                      |                       |
   ___ ___ ____         ____ ____ ____         ____ ____ ____         ____ ____ ____          ____ ____ ____
  | 8 | 9 |    |  <=>  | 10 | 11 |    |  <=>  | 12 | 18 |    |  <=>  | 20 | 25 |    |   <=>  | 45 | 85 |    | 
```

插入元素 `11` 之后，会传递性地导致节点 `[8, 9, 10]` 和节点 `[12, 20, 45]` 都产生分裂上溢。

注意，内部节点的分裂和叶子节点是不同的：

- 内部节点分裂后，子节点不会出现和父节点一样的关键码值
- 新分裂父节点的关键码值，等于插入新元素后的中间关键码值

这种分裂上溢可以一直持续到根节点，最后可能会导致 B+ 树提升一层（如上）。

上述的 4 种情况，对应的伪代码差不多是这样：

```java
/**- 插入新元素 -*/
private BPTNode<K, V> put(BPTNode<K, V> root, K key, V value) {
    int index = root.findIndex(key);
    if (root.isLeaf()) {
        // 叶子节点，有可能发生了分裂
        BPTLeaf<K, V> leaf = (BPTLeaf<K, V>) root;
        return leaf.add(key, value);
    } else {
        // 内部节点，添加到子树的叶子节点中
        BPTInternal<K, V> node = (BPTInternal<K, V>) root;
        BPTNode<K, V> child = node.getChild(index);
        child = put(child, key, value);

        // 添加到子树中，有可能发生了分裂
        return node.overflow(index, child);
    }
}
```

```java
/**- 叶子节点的上溢 -*/
public BPTNode<K, V> add(K key, V value) {
    int index = findIndex(key);

    // 1. 空间足够，可以直接插入新元素
    if (!this.isFull()) {
        insertEntry(index + 1, new Entry<>(key, value));
        return this;
    }

    // 2. 空间不足，需要分裂节点
    return splitNodes(index, new Entry<>(key, value));
}
```

```java
/**- 内部节点的上溢 -*/
public BPTNode<K, V> overflow(int index, BPTNode<K, V> node) {
    // 1. 当前空间足够，可以直接插入新节点
    if (size + node.size() < m) {
        for (Entry<K, V> entry : node.entries()) {
            insertEntry(++index, entry);
        }
        return this;
    }

    // 2. 当前空间不够，需要分裂成 3 个节点
    return splitNodes(index, node);
}
```

### 3.3 删除元素

删除元素，和插入一样，由于叶子节点的特殊性，所以实际上会有 6 种情况：

1. 叶子节点元素足够，直接删除
2. 叶子节点元素不足，看兄弟节点是否可以借一个过来（和内部节点不同）
3. 叶子节点兄弟节点不能借，那就和兄弟合并，删除父节点（和内部节点不同）
4. 内部节点元素足够，直接删除
5. 内部节点元素不足，看兄弟节点是否可以借一个过来（和叶子节点不同）
6. 内部节点兄弟节点不能借，那就和兄弟合并，删除父节点（和叶子节点不同）

这几种情况如下。

#### 3.3.1 叶子节点-直接删除

如果是叶子节点，且删除元素后，元素数量仍然处于半满状态，则可以直接删除：

```
             ____ ___ ___ 
            | 20 |   |   | 
                   |
        -----------------------
        |                     |
   ___ ____ ___         ____ ____ ____ 
  | 9 | 12 |   |  <=>  | 20 | 25 |    | 

                   |  删除 20
                   v 

             ____ ___ ___ 
            | 20 |   |   | 
                   |
        -----------------------
        |                     |
   ___ ____ ___         ____ ____ ____ 
  | 9 | 12 |   |  <=>  | 25 |    |    | 
```

叶子节点 `[20, 25]` 删除 `20` 以后，元素数量依旧满足半满状态，则可以直接删除并返回。

注意：直接删除叶子节点元素 `20` 后，并不会改变父节点的关键码值 `20`。

#### 3.3.2 叶子节点-从兄弟借

如果删除元素后，元素数量少于一半，不满足要求，则先看一下兄弟节点能不能借个元素过来：

```
              ____ ___ ___ 
             | 20 |   |   | 
                    |
         -----------------------
         |                     |
    ___ ____ ___         ____ ____ ____ 
   | 9 | 12 |   |  <=>  | 25 |    |    | 

                    |  删除 25
                    v 

              ____ ___ ___ 
             | 12 |   |   | 
                    |
         -----------------------
         |                     |
    ___ ____ ___         ____ ____ ____ 
   | 9 |    |   |  <=>  | 12 |    |    | 
```

右子节点 `[25]` 删除 `25` 以后，元素数量不满足要求了，找兄弟节点 `[9, 12]` 借一个元素。

和 B-树的旋转不同，B+ 树叶子节点的借用是这样的：

- 叶子节点直接从兄弟节点借一个元素过来
- 同时还要更新叶子节点对应的父节点关键码值

删除 `25` 以后，除了借 `12` 过来，还要更新父节点的关键码值为 `12`。

上面是从左兄弟借，从右兄弟借也是一样的，要更新父节点的关键码值：

```
              ____ ___ ___ 
             | 20 |   |   | 
                    |
         -----------------------
         |                     |
    ___ ____ ___         ____ ____ ____ 
   | 9 |    |   |  <=>  | 20 | 25 |    | 

                    |  删除 9
                    v 

              ____ ___ ___ 
             | 25 |   |   | 
                    |
          -----------------------
          |                     |
    ____ ___ ___         ____ ____ ____ 
   | 20 |   |   |  <=>  | 25 |    |    | 
```

#### 3.3.3 叶子节点-合并下溢

如果兄弟节点借不了元素，那就将`左子节点 + 右子节点`合并在一起，并删除`父节点元素`：

```
              ____ ___ ___ 
             | 25 |   |   |
                    |
          -----------------------
          |                     |
    ____ ___ ___         ____ ____ ____ 
   | 20 |   |   |  <=>  | 30 |    |    |

                    |  删除 30
                    v

              ____ ____ ___ 
             | 20 |    |   |
```

注意：叶子节点的合并下溢，并不包括父节点元素值，只是左右叶子兄弟合并，因为内部节点不是数据节点。

#### 3.3.4 内部节点-直接删除

叶子节点合并下溢后，父节点需要删除对应的父节点元素，如果父节点空间足够，就可以直接删除：

```
                         ____ ____ ___ 
                        | 12 | 20 |   | 
                               |
        -----------------------------------------------
        |                      |                      |
   ___ ____ ____         ____ ____ ____         ____ ____ ____ 
  | 9 | 10 |    |  <=>  | 12 | 18 |    |  <=>  | 20 |    |    |

                               |  删除 20
                               v

                         ____ ____ ___ 
                        | 12 |    |   | 
                               |
                   ------------------------
                   |                      |
              ___ ____ ____         ____ ____ ___
             | 9 | 10 |    |  <=>  | 12 | 18 |   |
```

删除 `20` 以后，会造成叶子节点合并下溢，父节点元素 `20` 被删除。

由于父节点 `[12, 20]` 空间足够，删除 `20` 后依旧满足要求，所以直接删除即可。

#### 3.3.5 内部节点-从兄弟借

如果内部节点删除时，发现空间不足，则先考虑从兄弟节点借一个元素：

```
                                               ____ ____ ____ 
                                              | 20 |    |    | 
                                                     |
                               ------------------------------------------------------------
                               |                                                          |
                         ____ ____ ___                                              ____ ___ ___ 
                        | 10 | 12 |   |                                            | 45 |   |   | 
                               |                                                          |
        ----------------------------------------------                      -------------------------
        |                      |                     |                      |                       |
   ___ ___ ____         ____ ____ ____         ____ ____ ____         ____ ____ ____          ____ ____ ____
  | 8 | 9 |    |  <=>  | 10 |    |    |  <=>  | 12 | 18 |    |  <=>  | 20 | 25 |    |   <=>  | 85 |    |    | 

                                                     |  删除 85
                                                     v

                                               ____ ____ ____ 
                                              | 12 |    |    | 
                                                     |
                              -----------------------------------------------
                              |                                             |
                        ____ ____ ___                                 ____ ___ ___ 
                       | 10 |    |   |                               | 20 |   |   | 
                              |                                             |
                   ------------------------                     ------------------------
                   |                      |                     |                      |
              ___ ___ ____         ____ ____ ____         ____ ____ ____         ____ ____ ____
             | 8 | 9 |    |  <=>  | 10 |    |    |  <=>  | 12 | 18 |    |  <=>  | 20 | 25 |    |
```

删除元素 `85`，会造成叶子节点 `[20, 25]`、`[85]` 合并，并删除父节点元素 `45`。

但是内部节点 `[45]` 由于空间不足，就需要从兄弟节点 `[10, 12]` 借一个元素过来才行。

不过，内部节点的借用和叶子节点的借用不同：

- 内部节点借用，实际上是旋转操作，将兄弟节点旋到父节点，父节点旋到当前节点

上面借用的元素 `12`，实际会旋转到父节点，然后父节点 `20` 再旋转到右子节点中。 

#### 3.3.6 内部节点-合并下溢

如果内部节点的兄弟节点借不了元素，只能将父子节点进行合并，产生下溢：

```
                                               ____ ____ ____ 
                                              | 12 |    |    | 
                                                     |
                              -----------------------------------------------
                              |                                             |
                        ____ ____ ___                                 ____ ___ ___ 
                       | 10 |    |   |                               | 20 |   |   | 
                              |                                             |
                   ------------------------                     ------------------------
                   |                      |                     |                      |
              ___ ___ ____         ____ ____ ____         ____ ____ ____         ____ ____ ____
             | 8 | 9 |    |  <=>  | 10 |    |    |  <=>  | 12 | 18 |    |  <=>  | 20 |    |    |

                                                     |  删除 20
                                                     v

                                               ____ ____ ___
                                              | 10 | 12 |   |
                                                     |
                                  -------------------------------------------
                               |                     |                      |
                          ___ ___ ____         ____ ____ ____         ____ ____ ____
                         | 8 | 9 |    |  <=>  | 10 |    |    |  <=>  | 12 | 18 |    |
```


删除元素 `20`，会造成叶子节点 `[12, 18]`、`[20]` 合并，并删除父节点元素 `20`。

删除内部节点元素 `20` 时，由于其空间不足，且兄弟节点无法借用，只能合并父子节点，产生下溢了。

注意，内部节点的合并和叶子节点的合并不同：

- 内部节点的合并下溢，包括了 `父节点 + 左子节点 + 右子节点`

即上面的父子节点 `[12]`、`[10]`、`[20]` 会合并起来，形成一个新节点。

下溢和上溢一样，也可能会持续下溢，一直持续到根节点，最终导致树的层级下降。

上述情况的伪代码如下：

```java
/**- 删除元素 -*/
private BPTNode<K, V> remove(BPTNode<K, V> root, K key) {
    int index = root.findIndex(key);
    if (root.isLeaf()) {
        BPTLeaf<K, V> leaf = (BPTLeaf<K, V>) root;
        if (key.equals(root.getKey(index))) {
            // 找到对应节点后移除元素
            return leaf.delete(index);
        } else {
            // 没找到对应节点，直接返回
            return leaf;
        }
    } else {
        BPTInternal<K, V> node = (BPTInternal<K, V>) root;
        // 在子树内递归移除元素
        remove(node.getChild(index), key);
        // 移除后可能需要父节点下溢
        return node.underflow(index);
    }
}
```

```java
/**- 叶子节点的下溢 -*/
public BPTNode<K, V> delete(K key, V value) {
    int index = findIndex(key);

    // 1. 空间足够，可以直接删除元素
    if (this.isEnough()) {
        removeEntry(index);
        return this;
    }

    // 2. 从兄弟节点借一个元素
    if (borrowLeft(index) || borrowRight(index)) {
        return this;
    }

    // 3. 合并左右兄弟节点
    return mergeNodes(index);
}
```

```java
/**- 内部节点的下溢 -*/
public BPTNode<K, V> underflow(int index) {
    // 1. 空间足够，可以直接删除元素
    if (this.isEnough()) {
        removeEntry(index);
        return this;
    }

    // 2. 从兄弟节点借一个元素
    if (borrowLeft(index) || borrowRight(index)) {
        return this;
    }

    // 3. 合并父元素 + 左右子节点
    return mergeNodes(index);
}
```

## 总结

- 性质
    - 内部节点只是索引，只有叶子节点才保存数据
    - 叶子节点之间构成了一条双向链表
    - 叶子节点包含了所有关键码值，而内部节点仅包含用于索引的关键码值
- 检索
    - 多路查找：一直找到叶子节点
- 插入
    1. 叶子节点空间足够，直接插入
    2. 叶子节点空间不足，分裂上溢（父节点关键码值等于右叶子节点的最小关键码值）
    3. 内部节点空间足够，直接插入
    4. 内部节点空间不足，分裂上溢（节点的关键码值都不一样）
- 删除
    1. 叶子节点空间足够，直接删除
    2. 叶子节点空间不足，从兄弟借（移动）
    3. 叶子节点兄弟节点无法借用，合并左右叶子节点，删除父元素，执行下溢（仅合并兄弟叶子节点）
    4. 内部节点空间足够，直接删除
    5. 内部节点空间不足，从兄弟借（旋转）
    6. 内部节点兄弟节点无法借用，合并父子节点，执行下溢（合并父子节点）


## 参考

> 《数据结构与算法分析（第三版）》
>
> https://www.cnblogs.com/nullzx/p/8729425.html


## 附录

### B+ 树接口

```java
/**
 * B+ 树接口定义
 *
 * @author weijiaduo
 * @since 2023/1/9
 */
public interface BPTree<K extends Comparable<K>, V>  {

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

### B+ 树实现

```java
/**
 * B+ 树实现类
 *
 * @author weijiaduo
 * @since 2023/1/9
 */
public class BPTreeImpl<K extends Comparable<K>, V> implements BPTree<K, V> {

    /**
     * 根节点
     */
    private BPTNode<K, V> root;
    /**
     * m 阶 B+ 树
     */
    private final int m;

    /**
     * B+树构造器
     *
     * @param m 指定 m 阶 B+ 树
     */
    public BPTreeImpl(int m) {
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
    private V get(BPTNode<K, V> root, K key) {
        if (root == null) {
            return null;
        }

        // 查找当前 key 所在的位置
        int index = root.findIndex(key);
        if (root.isLeaf()) {
            if (key.equals(root.getKey(index))) {
                BPTLeaf<K, V> leaf = (BPTLeaf<K, V>) root;
                return leaf.getValue(index);
            } else {
                return null;
            }
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
    private BPTNode<K, V> put(BPTNode<K, V> root, K key, V value) {
        // 1. 整棵树为空时，插入新节点
        if (root == null) {
            BPTLeaf<K, V> leaf = new BPTLeaf<>(m);
            return leaf.add(key, value);
        }

        int index = root.findIndex(key);
        if (root.isLeaf()) {
            // 2. 叶子节点
            BPTLeaf<K, V> leaf = (BPTLeaf<K, V>) root;

            // 验证是否已存在，存在则只需更新值
            if (key.equals(leaf.getKey(index))) {
                leaf.setValue(index, value);
                return leaf;
            }

            // 添加到叶子节点中，有可能发生了分裂
            return leaf.add(key, value);
        } else {
            // 3. 内部节点，添加到子树的叶子节点中
            BPTInternal<K, V> node = (BPTInternal<K, V>) root;
            BPTNode<K, V> child = node.getChild(index);
            child = put(child, key, value);

            // 添加到子树中，有可能发生了分裂
            return node.overflow(index, child);
        }
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
    private BPTNode<K, V> remove(BPTNode<K, V> root, K key) {
        if (root == null) {
            return null;
        }

        int index = root.findIndex(key);
        if (root.isLeaf()) {
            BPTLeaf<K, V> leaf = (BPTLeaf<K, V>) root;
            if (key.equals(root.getKey(index))) {
                // 找到对应节点后移除元素
                return leaf.delete(index);
            } else {
                // 没找到对应节点，直接返回
                return leaf;
            }
        } else {
            BPTInternal<K, V> node = (BPTInternal<K, V>) root;
            // 在子树内递归移除元素
            remove(node.getChild(index), key);
            // 移除后可能需要父节点下溢
            return node.underflow(index);
        }
    }

}
```

### B+ 节点定义

```java
/**
 * B+树基类节点
 *
 * @author weijiaduo
 * @since 2023/1/10
 */
public abstract class BPTNode<K extends Comparable<K>, V> {

    /**
     * m 阶 B 树
     */
    protected final int m;
    /**
     * 节点元素阈值
     */
    protected final int threshold;
    /**
     * 节点元素数组
     * <p>
     * 元素的第 0 位始终是一个占位元素，专门用于存放最左子节点
     * <p>
     * 所以一个节点能拥有的元素个数最多是 m - 1
     */
    protected final Object[] elements;
    /**
     * 实际元素数量
     * <p>
     * 取值范围是 [0, m - 1]
     */
    protected int size;

    public BPTNode(int m) {
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
    protected int half(int m) {
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
    protected boolean canBorrow() {
        return size > threshold;
    }

    /**
     * 是否是叶子节点
     * <p>
     * 没有任何一个非空子节点时才是叶子节点
     *
     * @return true叶子节点/false内部节点
     */
    public abstract boolean isLeaf();

    /**
     * 获取所有的元素
     *
     * @return 元素集合
     */
    protected List<Entry<K, V>> entries() {
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
     * 获取指定位置的 key
     *
     * @param index 索引
     * @return key
     */
    public K getKey(int index) {
        return getEntry(index).key;
    }

    /**
     * 获取所有的子节点
     * <p>
     * 注意：是所有可能的子节点，也包括了 null 子节点
     *
     * @return 子节点集合
     */
    public List<BPTNode<K, V>> children() {
        List<BPTNode<K, V>> children = new ArrayList<>(size + 1);
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
    public BPTNode<K, V> firstChild() {
        return getChild(0);
    }

    /**
     * 返回最后一个子节点
     * <p>
     * 注意：不是最后一个非空子节点，而是最后 1 个元素的右子节点
     *
     * @return 最后一个子节点
     */
    public BPTNode<K, V> lastChild() {
        return getChild(size);
    }

    /**
     * 获取指定位置的下一层子节点
     *
     * @param index 指定位置
     * @return 下一层子树根节点
     */
    public BPTNode<K, V> getChild(int index) {
        return getEntry(index).pointer;
    }

    /**
     * 更新指定位置的子节点
     *
     * @param index 索引
     * @param node  子节点
     */
    public void setChild(int index, BPTNode<K, V> node) {
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
    protected int binaryle(List<K> keys, K key) {
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
     * 获取指定位置的元素
     *
     * @param index 索引
     * @return 元素
     */
    protected Entry<K, V> getEntry(int index) {
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
    protected void setEntry(int index, Entry<K, V> entry) {
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
    protected void addEntry(Entry<K, V> entry) {
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
    protected void insertEntry(int index, Entry<K, V> entry) {
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
    protected void removeEntry(int index) {
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
    public static class Entry<K extends Comparable<K>, V> {
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
        BPTNode<K, V> pointer;

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

### B+ 树内部节点

```java
/**
 * B+树内部节点
 *
 * @author weijiaduo
 * @since 2023/1/10
 */
public class BPTInternal<K extends Comparable<K>, V> extends BPTNode<K, V> {

    public BPTInternal(int m) {
        super(m);
    }

    @Override
    public boolean isLeaf() {
        return false;
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
    public BPTNode<K, V> overflow(int index, BPTNode<K, V> node) {
        if (node == null || node.isEmpty()) {
            return this;
        }

        // 还是旧节点，说明子节点没有上溢，无需处理
        BPTNode<K, V> cur = getChild(index);
        if (cur == node) {
            return this;
        }

        // 1. 当前空间足够，可以直接插入新节点
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
    private BPTNode<K, V> splitNodes(int index, BPTNode<K, V> newNode) {
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
        BPTInternal<K, V> root = new BPTInternal<>(m);
        for (int i = mid; i <= mid; i++) {
            root.addEntry(allEntries.get(i));
        }
        // 左子节点
        BPTInternal<K, V> left = new BPTInternal<>(m);
        for (int i = 0; i < mid; i++) {
            left.addEntry(allEntries.get(i));
        }
        // 右子节点（和叶子节点不同，右子节点不包括根节点元素）
        BPTInternal<K, V> right = new BPTInternal<>(m);
        for (int i = mid + 1; i < newSize; i++) {
            right.addEntry(allEntries.get(i));
        }

        // 更新边界指针
        left.setChild(0, firstChild());
        right.setChild(0, root.lastChild());
        root.setChild(0, left);
        root.setChild(root.size, right);
        return root;
    }

    /**
     * 节点下溢处理
     * <p>
     * 存在非法子节点时，可能需要对父节点进行下溢操作
     *
     * @param index 父节点索引
     * @return 根节点
     */
    public BPTNode<K, V> underflow(int index) {
        // 验证子节点是否合法，合法就不用下溢
        BPTNode<K, V> cur = getChild(index);
        if (cur == null || cur.isLegal()) {
            return this;
        }

        // 1. 从兄弟节点借一个元素
        BPTNode<K, V> left = null;
        if (index > 0) {
            left = getChild(index - 1);
            if (left != null && left.canBorrow()) {
                return borrowLeft(index);
            }
        }
        BPTNode<K, V> right = null;
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
    private BPTNode<K, V> borrowLeft(int index) {
        BPTNode<K, V> left = getChild(index - 1);
        BPTNode<K, V> right = getChild(index);
        if (left.isLeaf()) {
            // 叶子节点，父元素不是数据，不能旋转到叶子节点里面
            // 左叶子节点转移到右叶子节点（叶子节点的子节点都为空，不需要更新）
            Entry<K, V> leftEntry = left.getEntry(left.size);
            left.removeEntry(left.size);
            right.insertEntry(1, leftEntry);
            // 更新父节点索引
            setEntry(index, new Entry<>(leftEntry.key, null));
            setChild(index, right);
        } else {
            // 内部节点，都是索引元素，直接旋转
            Entry<K, V> parentEntry = getEntry(index);
            Entry<K, V> leftEntry = left.getEntry(left.size);
            BPTNode<K, V> leftRight = left.lastChild();

            // 父节点转到右子节点
            right.insertEntry(1, parentEntry);
            right.setChild(1, right.firstChild());
            // 左子节点转到父节点
            left.removeEntry(left.size);
            setEntry(index, leftEntry);
            setChild(index, right);
            right.setChild(0, leftRight);
        }
        return this;
    }

    /**
     * 借用右子节点的值
     * <p>
     * 实际就是左旋，将父元素转到左子节点，右子节点的元素转到父节点
     *
     * @param index 父节点索引
     */
    private BPTNode<K, V> borrowRight(int index) {
        BPTNode<K, V> right = getChild(index + 1);
        BPTNode<K, V> left = getChild(index);
        if (right.isLeaf()) {
            // 叶子节点，父元素不是数据，不能旋转到叶子节点里面
            // 右叶子节点转移到左叶子子节点（叶子节点的子节点都为空，不需要更新）
            Entry<K, V> rightEntry = right.getEntry(1);
            right.removeEntry(1);
            left.addEntry(rightEntry);
            // 更新父节点索引
            Entry<K, V> newRightEntry = right.getEntry(1);
            setEntry(index + 1, new Entry<>(newRightEntry.key, null));
            setChild(index + 1, right);
        } else {
            // 内部节点，都是索引元素，直接旋转
            Entry<K, V> parentEntry = getEntry(index + 1);
            Entry<K, V> rightEntry = right.getEntry(1);
            BPTNode<K, V> rightLeft = right.firstChild();

            // 父节点转到左子节点
            left.addEntry(parentEntry);
            // 右子节点转到父节点
            right.setChild(0, right.getChild(1));
            right.removeEntry(1);
            setEntry(index + 1, rightEntry);
            setChild(index + 1, right);
            left.setChild(left.size, rightLeft);
        }
        return this;
    }

    /**
     * 父节点元素 + 右子节点，全都合并到左子节点
     * <p>
     * 因为父节点元素和右子节点是一一对应的，所以都是按照“父 + 右 --> 左”进行合并
     * <p>
     * 方便同时删除父节点元素和右子节点
     *
     * @param index 父节点索引
     */
    private BPTNode<K, V> mergeNodes(int index) {
        BPTNode<K, V> left = getChild(index - 1);
        BPTNode<K, V> right = getChild(index);

        // 父节点元素合并到左子节点
        if (!left.isLeaf()) {
            // 子节点是叶子节点时，合并不要加上父元素
            Entry<K, V> parentEntry = getEntry(index);
            left.addEntry(parentEntry);
        }

        // 右子节点合并到左子节点
        left.setChild(left.size, right.firstChild());
        for (Entry<K, V> entry : right.entries()) {
            left.addEntry(entry);
        }

        // 移除父节点元素
        removeEntry(index);

        // 更新叶子节点双向链表的指针
        if (left.isLeaf()) {
            BPTLeaf<K, V> leftLeaf = (BPTLeaf<K, V>) left;
            BPTLeaf<K, V> rightLeaf = (BPTLeaf<K, V>) right;
            BPTLeaf<K, V> rightNext = rightLeaf.getNext();
            if (rightNext != null) {
                rightNext.setPrev(leftLeaf);
            }
            leftLeaf.setNext(rightNext);
        }
        return this;
    }

}
```

### B+ 树叶子节点

```java
/**
 * B+树叶子节点
 *
 * @author weijiaduo
 * @since 2023/1/10
 */
public class BPTLeaf<K extends Comparable<K>, V> extends BPTNode<K, V> {

    /**
     * 双向链表，上一个节点
     */
    private BPTLeaf<K, V> prev;
    /**
     * 双向链表，下一个节点
     */
    private BPTLeaf<K, V> next;

    public BPTLeaf(int m) {
        super(m);
    }

    @Override
    public boolean isLeaf() {
        return true;
    }

    /**
     * 添加新元素
     *
     * @param key   key
     * @param value value
     */
    public BPTNode<K, V> add(K key, V value) {
        int index = findIndex(key);

        // 1. 空间足够，可以直接插入新元素
        if (!this.isFull()) {
            insertEntry(index + 1, new Entry<>(key, value));
            return this;
        }

        // 2. 空间不足，需要分裂节点
        return splitNodes(index, new Entry<>(key, value));
    }

    /**
     * 叶子节点已满，无法插入新元素时，需分裂成 3 个节点
     *
     * @param index    元素插入位置
     * @param newEntry 新元素
     * @return 分裂后的父节点
     */
    private BPTNode<K, V> splitNodes(int index, Entry<K, V> newEntry) {
        List<Entry<K, V>> curEntries = entries();
        curEntries.add(index, newEntry);
        int newSize = curEntries.size();
        int mid = newSize / 2;

        // 根节点（内部节点只需要索引就够了，不需要数据）
        BPTNode<K, V> root = new BPTInternal<>(m);
        Entry<K, V> entry = curEntries.get(mid);
        root.addEntry(new Entry<>(entry.key, null));
        // 左子节点
        BPTLeaf<K, V> left = new BPTLeaf<>(m);
        for (int i = 0; i < mid; i++) {
            left.addEntry(curEntries.get(i));
        }
        // 右子节点（和内部节点不同，右子节点包括了根节点元素）
        BPTLeaf<K, V> right = new BPTLeaf<>(m);
        for (int i = mid; i < newSize; i++) {
            right.addEntry(curEntries.get(i));
        }
        // 设置父子节点关系
        root.setChild(0, left);
        root.setChild(root.size, right);

        // 更新叶子节点双向链表指针
        left.prev = this.prev;
        if (this.prev != null) {
            this.prev.next = left;
        }
        left.next = right;
        right.prev = left;
        if (this.next != null) {
            this.next.prev = right;
        }
        right.next = this.next;
        this.prev = this.next = null;
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
    public BPTNode<K, V> delete(int index) {
        // 叶子节点直接删除元素，删除后可能会变成元素为空的空节点
        // 空节点会由内部节点处理掉，或者由根节点处理掉
        removeEntry(index);
        return this;
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
     * @return 上一个叶子节点
     */
    public BPTLeaf<K, V> getPrev() {
        return prev;
    }

    /**
     * @return 下一个叶子节点
     */
    public BPTLeaf<K, V> getNext() {
        return next;
    }

    /**
     * 更新上一个叶子节点
     *
     * @param prev 上一个叶子节点
     */
    protected void setPrev(BPTLeaf<K, V> prev) {
        this.prev = prev;
    }

    /**
     * 更新下一个叶子节点
     *
     * @param next 下一个叶子节点
     */
    protected void setNext(BPTLeaf<K, V> next) {
        this.next = next;
    }

}
```
