# 斜堆

## 一、什么是斜堆？

斜堆（Skew Heap），也称斜树（Skew Tree），自适应堆(Self-Adjusting Heap)，是一种自平衡二叉堆数据结构。

斜堆是左倾堆（Leftist Heap）的一个变种，只是斜堆的节点中没有 NPL 这个属性而已。

斜堆的主要特点包括：

<!--more-->

- 不平衡性：整体是不平衡的，并且在任何操作中都不会显式执行旋转操作来维持平衡
- 自平衡性：每次操作后，仅通过互换左右子树来调整结构，从而达到自平衡的效果
- 简单性：因为不需要执行显式的平衡操作，所以实现起来相对简单

斜堆最主要的特点就是，不执行显式的平衡操作，而是通过交换左右子树来维护自平衡性。

下面是斜堆的一个例子：

```
       1 
      / \
     2   3
    / \
   4   6
  /
 5
```

单从结构上看，斜堆和普通堆差不多，感受不到区别。

不过实际上它和左倾堆是类似的，整体结构具有左倾性。


## 二、为什么用斜堆？

斜堆在某些场景下，具有一些特定的应用和优点：

- 实现简单：没有复杂的平衡操作，实现简单
- 合并高效：合并操作非常高效，没有额外的平衡操作，仅需要互换左右子树
- 自适应性：经常被访问的元素会被提升到根节点，从而提高了对这些元素的访问速度
- 空间效率：节点不需要保存额外的平衡信息，比如 左倾堆、AVL、红黑树等

尽管斜堆具有上述优点，但它也有一些缺点，最坏情况下的性能不如其他一些堆结构。

斜堆通常适用于需要高效的合并操作和自适应性的场景，且不太依赖最坏情况性能保证的情况。

比如 Prim 算法、Kruskal 算法、优先级队列等需要频繁合并操作的场景。


## 三、怎么实现斜堆？

合并操作是斜堆的重点，合并两个堆的步骤如下：

1. 如果两个堆中有一个为空，则直接返回另一个堆
2. 如果两个堆都不为空，则比较两个堆的根节点，将较小的根节点作为新堆的根节点
3. 将较小根节点的右子树和较大根节点合并，然后将合并后的树作为新堆的右子树
4. 最后交换左右子树

其实和左倾堆差不多，就是少了 NPL 属性判断而已，伪代码如下：

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
    swap(min.left, min.right);

    return min;
}
```

下面给出一个合并操作的例子：

```
      堆1              堆2

       2                1
      / \              / \
     6   4            3   5
```

1）首先，取小根节点 1 作为新堆的根节点，然后合并小根右孩子与大根堆

```
      新堆1                     待合并
           
       1                       5      2     
      /                              / \
     3                              6   4
```

2）同样地，取小根节点 2 作为新堆的根节点，合并小根右孩子与大根堆

```
      新堆1     新堆2            待合并
                                     
       1         2             5      4
      /         /  
     3         6
```

3）同上，取小根节点 4 作为新堆根节点，合并小根右孩子（null 节点）与大根堆


```
      新堆1     新堆2     新堆3
                                     
       1         2        4
      /         /          \
     3         6            5
```

4）接着交换 4 的左右孩子

```
      新堆1     新堆2     新堆3
                                     
       1         2        4
      /         /        /
     3         6        5
```

5）接着开始递归往上合并，4 返回给 2 的右孩子

```
      新堆1     新堆2
                                     
       1         2
      /         / \
     3         6   4
                  /
                 5
```

6）接着交换 2 的左右孩子

```
      新堆1     新堆2
                                     
       1         2
      /         / \
     3         4   6
              /
             5
```

7）继续往上合并，2 返回给 1 的右孩子

```
      新堆1
                                     
       1 
      / \
     3   2
        / \
       4   6
      /
     5
```

8）接着交换 1 的左右孩子

```
      新堆1
                                     
       1 
      / \
     2   3
    / \
   4   6
  /
 5
```

至此，合并操作就完成了。

至于其他的操作，比如删除和插入，实际上都是通过合并操作实现的：

1. 删除最小元素：相当于删除根节点，然后合并左右子树
2. 插入新元素：相当于将新元素作为一个新的斜堆，然后两个堆合并

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

总的来说，斜堆和左倾堆差不多，实现也是比较简单的。

## 参考

> https://www.cnblogs.com/skywang12345/p/3638552.html

## 附录

### 斜堆接口

```java
/**
 * 斜堆（Skew Heap）
 * <p>
 * 左倾堆（Leftist Heap）的变种，只是斜堆没有 NPL 这个属性
 *
 * @author weijiaduo
 * @since 2023/9/26
 */
public interface SkewHeap<T> {

    /**
     * 合并另外一个堆
     *
     * @param other 另外一个堆
     */
    void merge(SkewHeap<T> other);

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

### 斜堆实现

```java
/**
 * 斜堆（Skew Heap）实现类
 * <p>
 * 左倾堆（Leftist Heap）的变种，只是斜堆没有 NPL 这个属性
 *
 * @author weijiaduo
 * @since 2023/9/26
 */
public class SkewHeapImpl<T extends Comparable<T>> implements SkewHeap<T> {

    /**
     * 堆根节点
     */
    private SkewHeapNode<T> root;
    /**
     * 节点数量
     */
    private int size;

    @Override
    public void merge(SkewHeap<T> other) {
        if (!(other instanceof SkewHeapImpl<T>)) {
            return;
        }

        root = merge(root, ((SkewHeapImpl<T>) other).root);
        size += other.size();
    }

    @Override
    public void insert(T val) {
        root = merge(root, new SkewHeapNode<>(val));
        size++;
    }

    @Override
    public T removeFirst() {
        if (size <= 0) {
            throw new IllegalStateException("Heap is Empty!");
        }

        SkewHeapNode<T> node = root;
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
    private SkewHeapNode<T> merge(SkewHeapNode<T> h1, SkewHeapNode<T> h2) {
        if (h1 == null) {
            return h2;
        }
        if (h2 == null) {
            return h1;
        }

        // 以小根节点作为新堆的根节点
        SkewHeapNode<T> min, max;
        if (h1.val.compareTo(h2.val) <= 0) {
            min = h1;
            max = h2;
        } else {
            min = h2;
            max = h1;
        }

        // 小根节点的右子节点和大根节点合并
        min.right = merge(min.right, max);

        // 交换左右子节点
        SkewHeapNode<T> tmp = min.left;
        min.left = min.right;
        min.right = tmp;

        return min;
    }

}
```
