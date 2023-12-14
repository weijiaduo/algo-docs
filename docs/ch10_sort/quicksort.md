# 快速排序

## 一、算法描述

### 1.1 核心思想

- 二分，选取一个分区值，将数据分割成 2 部分：小于和大于
- 递归，对小于大于两部分再排序，不断选点分割数据，直到无法分割为止
- 整个二分过程类似于一棵二叉树，从上往下排序，先确定根节点位置，再处理子树排序

### 1.2 细节解释

<!--more-->

比如说， 数组 `[5, 7, 2, 4, 3, 6, 1]`。

首先选取一个分区值（随意选一个），比如说 5。

以 5 基准，将其他数据分割成 2 部分：一部分小于 5，一部分大于 5。

```
      小于5                     大于5
 ___ ___ ___ ___               ___ ___
| 2 | 4 | 3 | 1 |             | 7 | 6 |
```

然后将 5 插入 2 部分的中间：

```
      小于5                     大于5
 ___ ___ ___ ___      ___      ___ ___
| 2 | 4 | 3 | 1 |    | 5 |    | 7 | 6 |
```

此时 5 就已经排好了。

下面接着对左右 2 部分进行递归排序。

(1) 先对左边小于 5 的部分排序

首先选取一个分区值 2，然后以 2 为基准分割成 2 部分：

```
小于2           大于2
 ___           ___ ___
| 1 |         | 4 | 3 |
```

将 2 插入中间位置：

```
小于2           大于2
 ___    ___    ___ ___
| 1 |  | 2 |  | 4 | 3 |
```

此时 2 就排好了。

接着不断往下递归，直到无法分割为止，最后可以得到：

```
 ___    ___    ___    ___
| 1 |  | 2 |  | 3 |  | 4 |
```

(2) 再对右边大于 5 的部分排序

同理，也是递归分割数据，最后可以得到：

```
 ___    ___
| 6 |  | 7 |
```

(3) 最后完整的序列结果

```
 ___    ___    ___    ___    ___    ___    ___
| 1 |  | 2 |  | 3 |  | 4 |  | 5 |  | 6 |  | 7 |
```

至此排序就完成了，快速排序本质上是一种分治算法，将大排序变成小排序来做。

## 二、算法实现

### 2.1 递归二分排序

```java
void partSort(int[] arr, int start, int end) {
    if (start < end) {
        // 分区
        int m = partition(arr, start, end);
        // 左边排序
        partSort(arr, start, m);
        // 右边排序
        partSort(arr, m + 1, end);
    }
}
```

### 2.2 对数据进行分区

```java
int partition(int[] arr, int start, int end) {
    // 选取分区点
    int p = pivot(arr, start, end - 1);
    // 将分区点放到最前面
    swap(arr, start, p);
    int ref = arr[start];
    int lp = start;
    // 将数据与分区点对比，分成小于和大于2部分
    for (int i = lp + 1; i < end; i++) {
        if (arr[i] <= ref) {
            swap(arr, ++lp, i);
        }
    }
    // 将分区点放到它最终的位置
    swap(arr, start, lp);
    return lp;
}
```

### 2.3 分区点的选择

```java
// 采用左中右三点取中值的方式
int pivot(int[] arr, int i, int j) {
    int mid = i + (j - i) / 2;
    if (arr[i] < arr[j]) {
        return arr[mid] > arr[i] ? mid : j;
    } else {
        return arr[mid] < arr[i] ? mid : i;
    }
}
```


## 三、算法分析

### 3.1 时间复杂度

- 最好时间复杂度：O(nlogn)
- 最坏时间复杂度：O(n^2)
- 平均时间复杂度：O(nlogn)

### 3.2 空间复杂度

- 空间复杂度：O(logn)
- 原地算法

### 3.3 稳定性

- 不稳定排序算法


## 四、适用场景

- 数据量比较大
- 数据有序性比较差


## 附录

```java
/**
 * 快速排序
 * <p>
 * 时间复杂度：最好 O(nlogn) 最差 O(n^2) 平均 O(nlogn)
 * <p>
 * 空间复杂度：O(logn)
 * <p>
 * 稳定性：不稳定
 *
 * @author weijiaduo
 * @since 2022/7/16
 */
public class QuickSort implements Sort {

    /**
     * 排序
     *
     * @param arr 数组
     */
    @Override
    public void sort(int[] arr) {
        partSort(arr, 0, arr.length);
    }

    /**
     * 递归排序
     *
     * @param arr   数组
     * @param start [start, end)
     * @param end   [start, end)
     */
    private void partSort(int[] arr, int start, int end) {
        if (start < end) {
            // 分区
            int m = partition(arr, start, end);
            // 左边排序
            partSort(arr, start, m);
            // 右边排序
            partSort(arr, m + 1, end);
        }
    }

    /**
     * 二分数组
     *
     * @param arr   数组
     * @param start [start, end)
     * @param end   [start, end)
     * @return 分隔点索引
     */
    private int partition(int[] arr, int start, int end) {
        // 选取分区点
        int p = pivot(arr, start, end - 1);
        // 将分区点放到最前面
        swap(arr, start, p);
        int ref = arr[start];
        int lp = start;
        // 将数据与分区点对比，分成小于和大于2部分
        for (int i = lp + 1; i < end; i++) {
            if (arr[i] <= ref) {
                swap(arr, ++lp, i);
            }
        }
        // 将分区点放到它最终的位置
        swap(arr, start, lp);
        return lp;
    }

    /**
     * 选择分区点（选择三个点的中值位置）
     *
     * @param arr 数组
     * @param i   [i, j]
     * @param j   [i, j]
     * @return 分区点索引
     */
    private int pivot(int[] arr, int i, int j) {
        int mid = i + (j - i) / 2;
        if (arr[i] < arr[j]) {
            return arr[mid] > arr[i] ? mid : j;
        } else {
            return arr[mid] < arr[i] ? mid : i;
        }
    }

}
```
