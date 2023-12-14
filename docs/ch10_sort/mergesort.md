# 归并排序

## 一、算法描述

### 1.1 核心思想

- 二分，将数据二等均分，然后分别排序，再合并2个排好序的数据
- 递归，一直二等均分数据，直到无法分割后，才开始递归合并返回
- 整个二分和合并的过程类似于一棵二叉树，从下往上合并数据，先对子树排序，再合并成根节点

### 1.2  细节解释

<!--more-->

比如说，数组 `[5, 7, 2, 4, 3, 6, 1]`。

首先是一直二等均分数据，直到无法分割为止：

```
 ___     ___     ___     ___     ___     ___     ___
| 5 |   | 7 |   | 2 |   | 4 |   | 3 |   | 6 |   | 1 |
```

然后从下往上开始合并排序返回：

```
 ___     ___     ___     ___     ___     ___     ___
| 5 |   | 7 |   | 2 |   | 4 |   | 3 |   | 6 |   | 1 |       -- 原始数组
  |       |       |       |       |       |       |
  |_______|       |_______|       |_______|       |
      |               |               |           |
      V               V               V           V
   ___ ___         ___ ___         ___ ___       ___
  | 5 | 7 |       | 2 | 4 |       | 3 | 6 |     | 1 |       -- 第一层合并
      |               |               |           |
      |_______________|               |___________|
              |                             |
              V                             V
       ___ ___ ___ ___                 ___ ___ ___
      | 2 | 4 | 5 | 7 |               | 1 | 3 | 6 |         -- 第二层合并
              |                             |
              |_____________________________|
                             |
                             V
                ___ ___ ___ ___ ___ ___ ___
               | 1 | 2 | 3 | 4 | 5 | 6 | 7 |                -- 第三层合并
```

归并排序采用的是分治思想，将小的有序数据逐渐合并成大的有序数据。

将 2 个有序数组合并成 1 个数组时，一般采用双指针，类似这样：

```
        lp                              rp
        |                               |
        V                               V
       ___ ___ ___ ___                 ___ ___ ___
      | 2 | 4 | 5 | 7 |               | 1 | 3 | 6 |         -- 当前数组


                ___ ___ ___ ___ ___ ___ ___
               |   |   |   |   |   |   |   |                -- 额外空间
```

判断双指针指向元素的大小，小的放到额外空间中，然后移动指针：

```
        lp                                  rp
        |                                   |
        V                                   V
       ___ ___ ___ ___                 ___ ___ ___
      | 2 | 4 | 5 | 7 |               | 1 | 3 | 6 |         -- 当前数组


                ___ ___ ___ ___ ___ ___ ___
               | 1 |   |   |   |   |   |   |                -- 额外空间
```

通过不断判断和移动双指针，最终可以完成对 2 个有序数组的合并：

```
            lp                              rp
            |                               |
            V                               V
       ___ ___ ___ ___                 ___ ___ ___
      | 2 | 4 | 5 | 7 |               | 1 | 3 | 6 |         -- 当前数组


                ___ ___ ___ ___ ___ ___ ___
               | 1 | 2 |   |   |   |   |   |                -- 额外空间
```
```
            lp                                 rp
            |                                   |
            V                                   V
       ___ ___ ___ ___                 ___ ___ ___
      | 2 | 4 | 5 | 7 |               | 1 | 3 | 6 |         -- 当前数组


                ___ ___ ___ ___ ___ ___ ___
               | 1 | 2 | 3 |   |   |   |   |                -- 额外空间
```
```
                lp                              rp
                |                               |
                V                               V
       ___ ___ ___ ___                 ___ ___ ___
      | 2 | 4 | 5 | 7 |               | 1 | 3 | 6 |         -- 当前数组


                ___ ___ ___ ___ ___ ___ ___
               | 1 | 2 | 3 | 4 |   |   |   |                -- 额外空间
```
```
                    lp                          rp
                    |                           |
                    V                           V
       ___ ___ ___ ___                 ___ ___ ___
      | 2 | 4 | 5 | 7 |               | 1 | 3 | 6 |         -- 当前数组


                ___ ___ ___ ___ ___ ___ ___
               | 1 | 2 | 3 | 4 | 5 |   |   |                -- 额外空间
```
```
                    lp                              rp
                    |                               |
                    V                               V
       ___ ___ ___ ___                 ___ ___ ___
      | 2 | 4 | 5 | 7 |               | 1 | 3 | 6 |         -- 当前数组


                ___ ___ ___ ___ ___ ___ ___
               | 1 | 2 | 3 | 4 | 5 | 6 |   |                -- 额外空间
```
```
                        lp                          rp
                        |                           |
                        V                           V
       ___ ___ ___ ___                 ___ ___ ___
      | 2 | 4 | 5 | 7 |               | 1 | 3 | 6 |         -- 当前数组


                ___ ___ ___ ___ ___ ___ ___
               | 1 | 2 | 3 | 4 | 5 | 6 | 7 |                -- 额外空间
```

利用额外空间合并完成之后，再把排好序的数据回写到原数组即可。


## 二、算法实现

二分排序代码，将数据分割成2部分，分别排序：

```java
sort(int[] arr, int start, int end) {
    if (start + 1 >= end) {
        return;
    }

    // 计算中点
    int mid = start + (end - start) / 2;
    // 对左边进行排序
    sort(arr, start, mid);
    // 对右边进行排序
    sort(arr, mid, end);
    // 合并左右两边的数据
    merge(arr, start, mid, end);
}
```

合并排好序的数据：

```java
merge(int[] arr, int start, int mid, int end) {
    int n = end - start;
    int[] tempArr = new int[n];
    int i = start, j = mid, k = 0;
    while (i < mid && j < end) {
        // 优先排左边的，保证稳定性
        if (arr[i] <= arr[j]) {
            tempArr[k++] = arr[i++];
        } else {
            tempArr[k++] = arr[j++];
        }
    }
    // 左边剩余元素
    while (i < mid) {
        tempArr[k++] = arr[i++];
    }
    // 右边剩余元素
    while (j < end) {
        tempArr[k++] = arr[j++];
    }
    // 注意复制到原数组时，起点是 start
    System.arraycopy(tempArr, 0, arr, start, n);
}
```


## 三、算法分析

### 3.1 时间复杂度

- 最好时间复杂度：O(nlogn)
- 最坏时间复杂度：O(nlogn)
- 平均时间复杂度：O(nlogn)

### 3.2 空间复杂度

- 空间复杂度：O(n)
- 非原地算法，需要额外空间

### 3.3 稳定性

- 稳定排序算法


## 四、适用场景

- 空间比较充足
- 利用空间换时间提升性能


## 附录

```java
/**
 * 归并排序
 * <p>
 * 时间复杂度：O(nlogn)
 * <p>
 * 空间复杂度：O(1)
 * <p>
 * 稳定性：稳定
 *
 * @author weijiaduo
 * @since 2022/9/1
 */
public class MergeSort implements Sort {

    @Override
    public void sort(int[] arr) {
        sort(arr, 0, arr.length);
    }

    /**
     * 排序
     *
     * @param arr   数组
     * @param start [start, end)
     * @param end   [start, end)
     */
    private void sort(int[] arr, int start, int end) {
        if (start + 1 >= end) {
            return;
        }

        // 计算中点
        int mid = start + (end - start) / 2;
        // 对左边进行排序
        sort(arr, start, mid);
        // 对右边进行排序
        sort(arr, mid, end);
        // 合并左右两边的数据
        merge(arr, start, mid, end);
    }

    /**
     * 合并排序数组
     *
     * @param arr   数组
     * @param start [start, end)
     * @param mid   [start, mid) 和 [mid, end)
     * @param end   [start, end)
     */
    private void merge(int[] arr, int start, int mid, int end) {
        int n = end - start;
        int[] mergeArr = new int[n];
        int i = start, j = mid, k = 0;
        while (i < mid && j < end) {
            // 优先排左边的，保证稳定性
            if (arr[i] <= arr[j]) {
                mergeArr[k++] = arr[i++];
            } else {
                mergeArr[k++] = arr[j++];
            }
        }
        // 左边剩余元素
        while (i < mid) {
            mergeArr[k++] = arr[i++];
        }
        // 右边剩余元素
        while (j < end) {
            mergeArr[k++] = arr[j++];
        }
        // 注意复制到原数组时，起点是 start
        System.arraycopy(mergeArr, 0, arr, start, n);
    }

}
```
