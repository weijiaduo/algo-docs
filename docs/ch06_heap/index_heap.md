# 索引堆

## 一、什么是索引堆？

索引堆（Index Heap）是二叉堆的一个变种，是二叉堆的一种增强。

相比于二叉堆只能访问堆顶元素，索引堆可以通过索引访问堆中的任意元素。

比如下面的索引堆：

```
       1(30)      
    /        \   
 4(28)       3(16)  
    \         /    
   5(22)    2(12)    
```

除了可以直接访问堆顶元素 30，还可以通过索引访问其他元素：

<!--more-->

- 1: 30
- 2: 12
- 3: 16
- 4: 28
- 5: 22

这相当于给堆元素增加了索引，因此称为索引堆。

## 二、为什么用索引堆？

二叉堆，每次只能操作堆顶元素，没办法操作其他的元素。

在某些场景下，有时候需要操作一下非堆顶的元素，这种二叉堆就做不到了。

索引堆就是专门为了解决操作非堆顶元素问题而设计的。


## 三、索引堆如何实现？

一种简单的方式是给元素加上索引属性，但是这需要重新实现一套新代码。

但另一种常见方式是，基于现有的二叉堆来改造实现：

- 使用额外一个数组记录元素索引，将索引数组建成堆

其结构类似这样子：

```
                     6(30)     
                  /        \  
               4(28)       3(16)                   -- 索引堆结构
              /    \       /   
            2(18) 5(22)  1(12)   
  
   ___ ____ ____ ____ ____ ____ ____
  |   | 6  | 4  | 3  | 2  | 5  | 1  |              -- 索引堆数组

   ___ ____ ____ ____ ____ ____ ____
  |   | 12 | 18 | 16 | 28 | 22 | 30 |              -- 元素数组

```

其中，`6(30)` 表示索引为 6 的元素，对应的值为 30。

- 元素数组：按照元素索引保存，元素位置会一直保持不变
- 索引堆数组：保存的是元素索引，会按照堆的结构来调整位置

有了这两个数组，就能随意获取堆里面的元素了：

- 获取堆顶元素时，可通过索引堆数组获取
- 根据索引获取元素时，可通过元素数组获取

这其实相当于在二叉堆的基础上，加了一个中间层，用索引数组来替代元素数组建堆而已。

不过利用这种方式，就可以通过直接改造二叉堆的代码来实现索引堆了。

## 四、实际案例

问题描述：

> 多路归并问题，将多个有序的输入流，合并成一个有序的输出流。

解决方案：

这个问题，就可以使用索引堆来解决。

每个堆元素表示一路输入流，堆顶元素就是当前最小的那一路输入流。

```java
// 创建一个最小索引堆（会同时记录输入流的索引和值）
IndexHeap<Integer> indexHeap = new MinIndexHeap<>(n);

// 初始化多路数据
for (int i = 0; i < n; i++) {
    int num = ins[i].readInt();
    indexHeap.insert(i, num);
}

// 对多路数据进行排序
while (indexHeap.size() > 0) {
    // 移除堆顶记录，追加到输出流
    int index = indexHeap.firstIndex();
    int num = indexHeap.removeFirst();
    out.write(num);

    // 某一路数据已经读完了
    if (ins[index] == null) {
       continue;
    }

    // 从被移除的那一路补充数据
    num = ins[index].readInt();
    indexHeap.insert(index, num);
}
```

合并多路输入流时，需要操作非堆顶的元素。

如果用普通的二叉堆就不好实现了，而索引堆则正好解决这个问题。

## 参考

> 《算法（第4版）》

## 附录

### 索引堆接口

```java
/**
 * 索引堆接口
 *
 * @author weijiaduo
 * @since 2023/2/25
 */
public interface IndexHeap<T> {

    /**
     * 插入新值
     *
     * @param index 指定索引
     * @param val   新值
     */
    void insert(int index, T val);

    /**
     * 移除第一个节点
     *
     * @return 第一个节点值
     */
    T removeFirst();

    /**
     * 删除指定索引的元素
     *
     * @param index 指定索引
     * @return 被删除元素
     */
    T remove(int index);

    /**
     * @return 第一个元素
     */
    T first();

    /**
     * 根节点元素的索引
     *
     * @return 索引
     */
    int firstIndex();

    /**
     * 指定索引的元素是否存在
     *
     * @param index 指定索引
     * @return true/false
     */
    boolean containsIndex(int index);

    /**
     * @return 元素数量
     */
    int size();

}
```

### 索引堆实现

```java
/**
 * 索引堆
 *
 * @author weijiaduo
 * @since 2023/2/25
 */
public class IndexHeapImpl<T extends Comparable<T>> implements IndexHeap<T> {

    /**
     * 元素数组
     */
    private final T[] elements;
    /**
     * 堆数组，堆元素 -> 元素数组索引
     * <p>
     * 另外，heap[0] 不用
     */
    private final int[] heap;
    /**
     * 元素数组索引 -> 堆数组索引
     */
    private final int[] idxMap;
    /**
     * 元素比较器
     */
    private final Comparator<T> cmp;
    /**
     * 元素数量
     */
    private int size;

    public IndexHeapImpl(int capacity) {
        this(capacity, Comparator.reverseOrder());
    }

    public IndexHeapImpl(int capacity, Comparator<T> cmp) {
        this.cmp = cmp;
        //noinspection unchecked
        elements = (T[]) new Comparable[capacity];
        heap = new int[capacity + 1];
        idxMap = new int[capacity];
        Arrays.fill(heap, -1);
        Arrays.fill(idxMap, -1);
    }

    @Override
    public void insert(int idx, T val) {
        if (idx < 0 || idx >= elements.length) {
            throw new IllegalStateException(String.format("size: %d, index: %d", elements.length, idx));
        }

        if (containsIndex(idx)) {
            // 更新元素
            elements[idx] = val;
            int hp = idxMap[idx];
            siftUp(hp);
            siftDown(hp);
        } else {
            // 插入元素
            elements[idx] = val;
            int hp = ++size;
            heap[hp] = idx;
            idxMap[idx] = hp;
            siftUp(hp);
        }
    }

    @Override
    public T removeFirst() {
        if (size <= 0) {
            throw new IllegalStateException("Heap isEmpty!");
        }

        int hp = 1;
        int idx = heap[hp];
        T val = elements[idx];
        swap(hp, size);
        heap[size] = -1;
        idxMap[idx] = -1;
        elements[idx] = null;
        size--;

        siftDown(hp);
        return val;
    }

    @Override
    public T remove(int idx) {
        if (!containsIndex(idx)) {
            throw new IllegalStateException(String.format("size: %d, index: %d", elements.length, idx));
        }

        int hp = idxMap[idx];
        T val = elements[idx];
        swap(hp, size);
        heap[size] = -1;
        idxMap[idx] = -1;
        elements[idx] = null;
        size--;

        // 删除最后一个元素无需处理
        if (hp <= size) {
            siftUp(hp);
            siftDown(hp);
        }
        return val;
    }

    @Override
    public T first() {
        if (size <= 0) {
            throw new IllegalStateException("Heap isEmpty!");
        }
        return elements[heap[1]];
    }

    @Override
    public int firstIndex() {
        if (size <= 0) {
            throw new IllegalStateException("Heap isEmpty!");
        }
        return heap[1];
    }

    @Override
    public boolean containsIndex(int idx) {
        if (idx < 0 || idx >= elements.length) {
            return false;
        }
        return idxMap[idx] != -1;
    }

    @Override
    public int size() {
        return size;
    }

    /**
     * 从上往下调整
     *
     * @param hp 当前节点，堆数组索引
     */
    private void siftDown(int hp) {
        int i = hp;
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
     * @param hp 当前节点，堆数组索引
     */
    private void siftUp(int hp) {
        int i = hp;
        while (i > 1) {
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
     * @param hp 当前节点，堆数组索引
     * @return 父节点索引
     */
    private int parent(int hp) {
        return hp / 2;
    }

    /**
     * 左子节点索引
     *
     * @param hp 当前节点，堆数组索引
     * @return 左子节点索引
     */
    private int left(int hp) {
        return 2 * hp;
    }

    /**
     * 右子节点索引
     *
     * @param hp 当前节点，堆数组索引
     * @return 右子节点索引
     */
    private int right(int hp) {
        return 2 * hp + 1;
    }

    /**
     * elements[heap[hp]] 是否优先于 elements[heap[hq]]
     *
     * @param hp 堆数组索引 hp
     * @param hq 堆数组索引 hq
     * @return true/false
     */
    private boolean prior(int hp, int hq) {
        return cmp.compare(elements[heap[hp]], elements[heap[hq]]) < 0;
    }

    /**
     * 交换 2 个元素的位置
     *
     * @param hp 堆数组索引 hp
     * @param hq 堆数组索引 hq
     */
    private void swap(int hp, int hq) {
        if (hp == hq) {
            return;
        }
        int t = heap[hp];
        heap[hp] = heap[hq];
        heap[hq] = t;
        idxMap[heap[hp]] = hp;
        idxMap[heap[hq]] = hq;
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append("[");
        for (int i = 1; i <= size; i++) {
            if (i > 1) {
                sb.append(", ");
            }
            sb.append("(")
                    .append(heap[i])
                    .append(", ")
                    .append(elements[heap[i]])
                    .append(")");
        }
        sb.append("]");
        return sb.toString();
    }

}
```
