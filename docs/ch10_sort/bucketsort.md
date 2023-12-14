---
title: 桶排序
date: 2022-09-05 17:26:54
categories: [数据结构与算法, 排序算法]
tags: [算法, 排序, 桶排序]
thumbnail: /img/structure.jpg
top: true
---

# 桶排序

## 一、算法描述

### 1.1 核心思想

- 把不同范围的数据划分到不同的桶中
- 桶与桶之间是有序的，桶间无需排序
- 只需要对桶内的数据排序

### 1.2 细节解释

<!--more-->

比如说，数组 `[2, 7, 4, 9, 4, 8, 0, 7, 7, 2, 4, 4, 1, 6]`。

数据范围是 `[0, 9]`，假设桶大小为 3，那么可以划分成 4 个桶。

```
|   |     |   |     |   |     |   |
|   |     |   |     |   |     |   |
|   |     |   |     |   |     |   |
|   |     |   |     |   |     |   |
|   |     |   |     |   |     |   |
|___|     |___|     |___|     |___|
  0         1         2         3             -- 桶编号
[0,2]     [3,5]     [6,8]     [9,11]          -- 桶数据范围
```

然后遍历排序数组，将数据按照范围分发到不同的桶中：

```
|   |     |   |     |   |     |   |
|   |     |   |     | 6 |     |   |
| 1 |     | 4 |     | 7 |     |   |
| 2 |     | 4 |     | 7 |     |   |
| 0 |     | 4 |     | 8 |     |   |
|_2_|     |_4_|     |_7_|     |_9_|
  0         1         2         3             -- 桶编号
[0,2]     [3,5]     [6,8]     [9,11]          -- 桶数据范围
```

这个时候，桶之间的数据是有序的，所以只需对桶内数据进行排序即可：

```
|   |     |   |     |   |     |   |
|   |     |   |     | 8 |     |   |
| 2 |     | 4 |     | 7 |     |   |
| 2 |     | 4 |     | 7 |     |   |
| 1 |     | 4 |     | 7 |     |   |
|_0_|     |_4_|     |_6_|     |_9_|
  0         1         2         3             -- 桶编号
[0,2]     [3,5]     [6,8]     [9,11]          -- 桶数据范围
```

桶内排好序之后，按照桶的顺序遍历获取数据即可：

```
[0, 1, 2, 2, 4, 4, 4, 4, 6, 7, 7, 7, 8, 9]
```

桶排序就是利用了桶的有序性，避免了大范围的排序，换成了小范围排序。


## 二、算法实现

排序步骤可分为几步：

1. 根据数据范围划分桶
2. 将数据分发到不同桶
3. 对桶内数据排序
4. 按顺序收集桶数据

下面结束这几个步骤的具体内容。

### 2.1 根据数据范围划分桶

桶大小需要根据实际情况设置，不同类型的数据，桶大小可能不一样。

```java
// 找出数据的范围（最小最大值）
int min = Integer.MAX_VALUE;
int max = Integer.MIN_VALUE;
for (int num : arr) {
    if (num > max) {
        max = num;
    }
    if (num < min) {
        min = num;
    }
}

// 根据数据范围平均划分桶
int n = (max - min) / width + 1;
List<List<Integer>> buckets = new ArrayList<>(n);
for (int i = 0; i < n; i++) {
    buckets.add(new ArrayList<>());
}
```

### 2.2 将数据分发到不同桶

```java
// 把不同范围的数据划分到不同的桶里面
for (int num : arr) {
    int index = (num - min) / width;
    List<Integer> list = buckets.get(index);
    list.add(num);
}
```

### 2.3 对桶内数据排序

```java
// 桶之间已经是有序的了，只需对桶内排序
for (List<Integer> bucket : buckets) {
    Collections.sort(bucket);
}
```

### 2.4 按顺序收集桶数据

```java
// 按桶的顺序遍历所有数据，就是已经排好序的了
int k = 0;
for (List<Integer> bucket : buckets) {
    for (Integer num : bucket) {
        arr[k++] = num;
    }
}
```


## 三、算法分析

### 3.1 时间复杂度

如果数据都划分到同一个桶内，那么就回退成单数组排序的性能了。

- 最好时间复杂度：O(n)
- 最坏时间复杂度：O(nlogn)
- 平均时间复杂度：O(nlogn)

### 3.2 空间复杂度

- 空间复杂度：O(n)
- 非原地算法

### 3.3 稳定性

- 稳定排序算法


## 四、适用场景

- 外部排序
- 数据量比较大
- 数据容易划分为多个桶
- 数据分布范围比较均匀


## 五、限制条件

- 要求排序数据能很容易的划分成 n 个桶，且桶与桶之间是有序的
- 分布在各个桶之间的数据是比较均匀的


## 附录

```java
/**
 * 桶排序
 * <p>
 * 时间复杂度：O(n)
 * <p>
 * 空间复杂度：O(w)
 * <p>
 * 稳定性：稳定
 *
 * @author weijiaduo
 * @since 2022/9/5
 */
public class BucketSort implements Sort {

    int width;

    public BucketSort() {
        this(10);
    }

    public BucketSort(int width) {
        this.width = width;
    }

    @Override
    public void sort(int[] arr) {
        // 找出数据的范围（最小最大值）
        int min = Integer.MAX_VALUE;
        int max = Integer.MIN_VALUE;
        for (int num : arr) {
            if (num > max) {
                max = num;
            }
            if (num < min) {
                min = num;
            }
        }

        // 根据数据范围平均划分桶
        int n = (max - min + width) / width;
        List<List<Integer>> buckets = new ArrayList<>(n);
        for (int i = 0; i < n; i++) {
            buckets.add(new ArrayList<>());
        }

        // 把不同范围的数据划分到不同的桶里面
        for (int num : arr) {
            int index = (num - min) / width;
            List<Integer> list = buckets.get(index);
            list.add(num);
        }

        // 桶之间已经是有序的了，只需对桶内排序
        for (List<Integer> bucket : buckets) {
            Collections.sort(bucket);
        }

        // 按桶的顺序遍历所有数据，就是已经排好序的了
        int k = 0;
        for (List<Integer> bucket : buckets) {
            for (Integer num : bucket) {
                arr[k++] = num;
            }
        }
    }

}
```
