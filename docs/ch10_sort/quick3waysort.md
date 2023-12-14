# 三向快速排序

## 一、算法描述

### 1.1 核心思想

- 三分，选取一个分区值，将数据分割成 3 部分：小于、等于、大于
- 递归，对小于大于两部分继续排序，不断选点分割数据，直到无法分割为止
- 整个过程和快速排序类似，只是原来的一个分区点变成了一个区间而已

### 1.2 细节解释

<!--more-->

比如说， 数组 `[3, 5, 2, 1, 3, 5, 1]`。

首先选取一个分区值（随意选一个），比如说 3。

以 3 基准，将其他数据分割成 3 部分：小于 3、等于 3、大于 3。

```
    小于3             等于3          大于3
 ___ ___ ___        ___ ___        ___ ___
| 2 | 1 | 1 |      | 3 | 3 |      | 5 | 5 |
```

等于 3 的部分就不用管了，因为已经排好了。

接着对小于 3 和大于 3 的部分进行递归排序。

(1) 先对小于 3 的部分排序

首先选取一个分区值 2，然后以 2 为基准分割成 3 部分：

```
   小于2          等于2          大于2
 ___ ___          ___ 
| 1 | 1 |        | 2 |
```

等于 2 的部分就不用管了，因为已经排好了。

接着对小于 2 和大于 2 的部分进行递归排序，最终可以得到：

```
 ___ ___ ___ 
| 1 | 1 | 2 |
```

(2) 再对大于 3 的部分排序

一样地，选取一个分区值 5，然后以 5 为基准分割成 3 部分：

```
   小于5          等于5          大于5
                ___ ___
               | 5 | 5 |
```

等于 5 的部分就不用管了，因为已经排好了。

至此，整个排序实际上已经拍好了，结果就是：

```
 ___ ___ ___ ___ ___ ___ ___
| 1 | 1 | 2 | 3 | 3 | 5 | 5 |
```

三向快速排序和快速排序差不多，只是中间的单个元素变成了一个区间而已。

但是，如果数组中有大量重复元素，三向快速排序的效率会比快速排序高很多。


## 二、算法实现

```java
private void partSort(int[] arr, int start, int end) {
    if (start >= end) {
        return;
    }
    // 三分数据（小于、等于、大于）
    int x = arr[start];
    int lp = start - 1, rp = end;
    for (int i = lp + 1; i < rp; ) {
        if (arr[i] < x) {
            swap(arr, i++, ++lp);
        } else if (arr[i] > x) {
            swap(arr, i, --rp);
        } else {
            i++;
        }
    }
    // 对小于的部分排序
    partSort(arr, start, lp + 1);
    // 对大于的部分排序
    partSort(arr, rp, end);
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

- 含有大量重复元素


## 附录

```java
/**
 * 三向快速排序
 * <p>
 * 时间复杂度：最好 O(nlogn) 最差 O(n^2) 平均 O(nlogn)
 * <p>
 * 空间复杂度：O(logn)
 * <p>
 * 稳定性：不稳定
 *
 * @author weijiaduo
 * @since 2023/4/16
 */
public class Quick3WaySort implements Sort {

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
        if (start >= end) {
            return;
        }
        // 三分数据（小于、等于、大于）
        int x = arr[start];
        int lp = start - 1, rp = end;
        for (int i = lp + 1; i < rp; ) {
            if (arr[i] < x) {
                swap(arr, i++, ++lp);
            } else if (arr[i] > x) {
                swap(arr, i, --rp);
            } else {
                i++;
            }
        }
        // 对小于的部分排序
        partSort(arr, start, lp + 1);
        // 对大于的部分排序
        partSort(arr, rp, end);
    }

}
```
