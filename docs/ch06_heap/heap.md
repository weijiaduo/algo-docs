# 堆

## 一、是什么？

在逻辑意义上，堆是一棵二叉树，类似这样：

```
    大值堆                    小值堆

      30                        14    
   /      \                  /      \ 
  28      16                22      16
    \    /                 /       /  
    22  12                24      26  
```

堆的特性：

- 是一棵二叉树
- 节点是局部有序的（不像 BST 那样整体有序）

局部有序的意思是：

<!--more-->

- 大值堆：父节点的值总比子节点的值大
- 小值堆：父节点的值总比子节点的值小
- 兄弟节点之间不存在大小关系

即，局部有序是指的父子之间有序，兄弟之间无序。

## 二、为什么？

### 2.1 场景

- 每次只需要最优先的数据，而不必所有数据都排好序，比如任务调度
- 需要从大量数据中找出 TopN 的数据

### 2.2 目的

- 想要实现一种按优先级排列的数据结构
- 并且能保证较高的操作效率（插入和删除）

### 2.3 对比

- 有序表的插入和删除复杂度都是 O(n)，性能较差
- BST 的插入和删除复杂度是 O(logn)，但结构不稳定
- AVL 和红黑树可以满足条件，但是实现复杂

## 三、怎么做？

### 3.1 存储结构

堆是二叉树，按照一般实现，会使用节点链接的方式实现。

- 但是使用链接形式的二叉树来实现堆，过于浪费空间

如果把堆做成一棵完全二叉树，那就会相对简单很多：

- 以数组作为堆的底层物理存储结构
- 将堆树的节点按照层级顺序放入数组中

虽然堆的存储结构是数组，但是逻辑上是棵完全二叉树。

以大值堆为例，物理结构和逻辑结构的对应关系如下：

```
                   30     
                /      \  
               28      16                          -- 逻辑结构
              /  \    /   
             18  22  12   
  
   ___ ____ ____ ____ ____ ____ ____
  |   | 30 | 28 | 16 | 18 | 22 | 12 |              -- 物理结构
```

堆使用数组存储时的性质：

- 堆节点的位置从索引 1 开始
- 左子节点索引是 `2*i`，右子节点索引是 `2*i+1`
- 父节点索引是 `i/2`

索引从 `1` 开始是为了减少父子索引计算时的 `+/-` 次数。

### 3.2 建堆

建堆方式：

- 方式 1：将每个元素逐个插入堆中
- 方式 2：对分支节点，按照从下往上、从右往左的顺序调整

从理论上说，方案 2 的效率更高，因为它只需要处理 `n/2` 个元素。

比如，构建大值堆，构建的过程如下：

```
                            16=>30                  22=>28               12=>30=>16

      12                      12                      12                     30      
   /      \      16        /      \      22        /      \      12       /      \   
  22      16   =====>     22      30   =====>     28      30   =====>    28      16  
 /  \    /               /  \    /               /  \    /              /  \    /    
18  28  30              18  28  16              18  22  16             18  22  12    
```

这里的最后一个分支节点是 `16`，所以按照 `16 -> 22 -> 12` 的顺序来调整。

从最后一个分支节点开始，从下往上、从右往左逐个执行 `sift down`，就能构建出一个堆。

### 3.3 插入

插入过程：

- 先将插入元素放到数组最后，对应到堆树就是最后一个叶子节点
- 然后对插入元素从下往上递归调整父子节点的位置

比如，给下面例子的堆插入新元素 `40`，插入过程如下：

```
   要插入40                 插入40                   40=>16                 40=>30

      30                      30                      30                     40      
   /      \      40        /      \                /      \               /      \   
  28      16   =====>     28      16   =====>     28      40   =====>    28      30  
 /  \    /               /  \    /  \            /  \    /  \           /  \    /  \   
18  22  12              18  22  12  40          18  22  12  16         18  22  12  16  
```

插入过程是从下往上执行 `sift up`，直到结构稳定或抵达根节点为止。

### 3.4 删除

堆的删除，一般都是删除根节点。所以删除过程是这样的：

- 交换根节点和数组最后一个元素的位置
- 删除最后一个元素（原根节点）
- 对新根节点（原最后一个元素）从上往下调整位置

比如，下面例子要删除根节点 `30`，过程是这样的：

```
    要移除30               交换30-12                 移除30                调整12                   调整12

      30                      12                      12                     28                       28      
   /      \                /      \                /      \               /      \                 /      \   
  28      16   =====>     28      16   =====>     28      16   =====>    12      16     =====>    22      16  
 /  \    /               /  \    /               /  \                   /  \                     /  \       
18  22  12              18  22  30              18  22                 18  22                   18  12        
```

删除是从上往下执行 `siftdown` 方法，直到结构稳定或抵达叶子节点为止。

## 四、实际案例

问题描述：

> TopN 问题，从一个不知道长度的数据流中，统计 TopN 的数据

对应的伪代码如下：

```java
// 创建一个最小值堆
Heap<Integer> minHeap = new MinHeap<>(n + 1);
while ((num = in.readInt()) != -1) {
   minHeap.intsert(num);
   // 堆中只保留 n 个元素
   if (minHeap.size() > n) {
      // 删除最小的元素值
      minHeap.removeFirst();
   }
}
```

最终堆里只会保留 `n` 个最大值，即 `TopN` 数据。


## 总结

堆的性质：

- 堆是一棵二叉树（使用数组实现的堆才是完全二叉树）
- 节点是局部有序的（不像 BST 那样整体有序）
   - 大值堆：父节点的值总比子节点的值大
   - 小值堆：父节点的值总比子节点的值小
   - 兄弟节点之间不存在大小关系

堆的存储：

- 以数组作为堆的底层物理存储结构
- 将堆树的节点按照层级顺序放入数组中
- 使用数组存储的堆，是一棵完全二叉树
   - 堆节点的位置从索引 1 开始
   - 左子节点索引是 `2*i`，右子节点索引是 `2*i+1`
   - 父节点索引是 `i/2`
   - 索引从 `1` 开始是为了减少父子索引计算时的 `+/-` 次数

堆的操作：

- 建堆方式
   - 方式 1：将每个元素逐个插入堆中
   - 方式 2（更好）：对分支节点，按照从下往上、从右往左的顺序调整
- 插入过程
   - 先将插入元素放到数组最后，对应到堆树就是最后一个叶子节点
   - 然后对插入元素从下往上递归调整父子节点的位置
- 删除过程
   - 交换根节点和数组最后一个元素的位置
   - 删除最后一个元素（原根节点）
   - 对新根节点（原最后一个元素）从上往下调整位置

## 参考

> 《数据结构与算法设计（第三版）》
>
> 《算法（第4版）》
>

## 附录

### 堆接口

```java
/**
 * 堆接口
 *
 * @author weijiaduo
 * @since 2023/2/24
 */
public interface Heap<T> {

    /**
     * 插入新值
     *
     * @param val 新值
     */
    void insert(T val);

    /**
     * 移除第一个节点
     *
     * @return 第一个节点值
     */
    T removeFirst();

    /**
     * @return 第一个元素
     */
    T first();

    /**
     * @return 元素数量
     */
    int size();

}
```

### 堆实现

```java
/**
 * 堆实现
 *
 * @author weijiaduo
 * @since 2023/2/24
 */
public class HeapImpl<T extends Comparable<T>> implements Heap<T> {

    /**
     * 堆，从 1 开始
     */
    private final T[] elements;
    /**
     * 堆大小
     */
    private int size;
    /**
     * 值比较器
     */
    private final Comparator<T> cmp;

    public HeapImpl(int capacity) {
        //noinspection unchecked
        elements = (T[]) new Comparable[capacity + 1];
        cmp = Comparator.reverseOrder();
        size = 0;
    }

    public HeapImpl(T[] elements) {
        this(elements, Comparator.reverseOrder());
    }

    public HeapImpl(T[] elements, Comparator<T> cmp) {
        this.cmp = cmp;
        size = elements.length;
        //noinspection unchecked
        this.elements = (T[]) new Comparable[size + 1];
        System.arraycopy(elements, 0, this.elements, 1, size);
        build();
    }

    /**
     * 构建堆
     */
    private void build() {
        for (int i = size / 2; i > 0; i--) {
            siftDown(i);
        }
    }

    @Override
    public void insert(T val) {
        if (size + 1 >= elements.length) {
            throw new IllegalStateException("Heap isFull!");
        }

        elements[++size] = val;
        siftUp(size);
    }

    @Override
    public T removeFirst() {
        if (size <= 0) {
            throw new IllegalStateException("Heap isEmpty!");
        }

        T val = elements[1];
        swap(1, size--);
        siftDown(1);
        return val;
    }

    @Override
    public T first() {
        if (size <= 0) {
            throw new IllegalStateException("Heap isEmpty!");
        }
        return elements[1];
    }

    @Override
    public int size() {
        return size;
    }

    /**
     * 从上往下调整
     *
     * @param index 指定开始索引
     */
    private void siftDown(int index) {
        int i = index;
        while (i < size) {
            int m = i;
            int l = left(i);
            if (l <= size && prior(l, m)) {
                m = l;
            }
            int r = right(i);
            if (r <= size && prior(r, m)) {
                m = r;
            }
            if (m == i) {
                break;
            }
            swap(i, m);
            i = m;
        }
    }

    /**
     * 从下往上调整
     *
     * @param index 指定开始索引
     */
    private void siftUp(int index) {
        int i = index;
        while (i > 0) {
            int p = parent(i);
            if (p > 0 && prior(i, p)) {
                swap(i, p);
                i = p;
            } else {
                break;
            }
        }
    }

    /**
     * 父节点索引
     *
     * @param i 当前节点索引
     * @return 父节点索引
     */
    private int parent(int i) {
        return i / 2;
    }

    /**
     * 左子节点索引
     *
     * @param i 当前节点索引
     * @return 左子节点索引
     */
    private int left(int i) {
        return 2 * i;
    }

    /**
     * 右子节点索引
     *
     * @param i 当前节点索引
     * @return 右子节点索引
     */
    private int right(int i) {
        return 2 * i + 1;
    }

    /**
     * elements[i] 是否小于 elements[j]
     *
     * @param i 索引 i
     * @param j 索引 j
     * @return true/false
     */
    private boolean prior(int i, int j) {
        return cmp.compare(elements[i], elements[j]) < 0;
    }

    /**
     * 交换2个元素的位置
     *
     * @param i 索引 i
     * @param j 索引 j
     */
    private void swap(int i, int j) {
        if (i == j) {
            return;
        }
        T t = elements[i];
        elements[i] = elements[j];
        elements[j] = t;
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append("[");
        for (int i = 0; i <= size; i++) {
            if (i > 0) {
                sb.append(", ");
            }
            sb.append(elements[i]);
        }
        sb.append("]");
        return sb.toString();
    }

}
```
