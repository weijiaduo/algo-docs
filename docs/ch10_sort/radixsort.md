---
title: 基数排序
date: 2022-09-06 13:26:54
categories: [数据结构与算法, 排序算法]
tags: [算法, 排序, 基数排序]
thumbnail: /img/structure.jpg
top: true
---

# 基数排序

## 一、算法描述

### 1.1 核心思想

- 数据有高低位之分，位之间有递进关系
- 高位相等的情况下，才去对比低位大小
- 按照低位到高位的顺序，使用稳定排序算法对每一位排序

### 1.2 细节解释

<!--more-->

数据有高低位之分，比如十进制数字 `1232`，字符串 `dwsc`，二进制 `1001` 等等。

这些类型的数据都有高低位，而且是高位大的数据肯定比高位小的数据大。

比如 `123` 和 `234`，`123` 最高位 `1` 小于 `234` 最高位 `2`，所以  `123 < 234`。

- 基数排序就是利用数据的每一位进行排序，最终得到有序序列的

比如说，数组 `[12, 3, 154, 78, 9, 245, 35, 92]`。

因为数字长度不同，所以首先将所有数字补 0 进行长度对齐：

```
012
003
154
078
009
245
035
092
```

然后按照从低位到高位进行排序：

先按“个位”排序：

```
当前数组     取出“个”位     按“个”位排序

  012           xx2            012
  003           xx3            092
  154           xx4            003
  078    =>     xx8     =>     154
  009           xx9            245
  245           xx5            035
  035           xx5            078
  092           xx2            009
```

再按“十位”排序：

```
当前数组     取出“十”位     按“十”位排序

  012           x1x            003
  092           x9x            009
  003           x0x            012
  154    =>     x5x     =>     035
  245           x4x            245
  035           x3x            154
  078           x7x            078
  009           x0x            092
```

再按“百位”排序：

```
当前数组     取出“百”位     按“百”位排序

  003           0xx            003
  009           0xx            009
  012           0xx            012
  035    =>     0xx     =>     035
  245           2xx            078
  154           1xx            092
  078           0xx            154
  092           0xx            245
```

每个“位”都排完之后，就是最终的排序结果了：

```
[3, 9, 12, 35, 78, 92, 154, 245]
```

基数排序的关键在于从低位往高位排序，这样能保证高位排序是最后的。


## 二、算法实现

### 2.1 计算数据的基数

```java
// 找到最大值
int max = Integer.MIN_VALUE;
for (int a : arr) {
    if (a > max) {
        max = a;
    }
}

// 算出基数（即位数）
int radix = 0;
while(max > 0) {
    max = max / 10;
    radix++;
}
```

### 2.2 从低位到高位排序

```java
// 从低位到高位对数组进行排序
int exp = 1;
for (int i = 0; i < radix; i++) {
    countSort(arr, exp);
    exp *= 10;
}
```

对每一位进行排序，则使用计数排序（或者其他稳定排序算法也行）。


## 三、算法分析

### 3.1 时间复杂度

- 最好时间复杂度：O(n)
- 最坏时间复杂度：O(n)
- 平均时间复杂度：O(n)

### 3.2 空间复杂度

- 空间复杂度：O(n)
- 非原地算法

### 3.3 稳定性

- 稳定排序算法


## 四、适用场景

- 数据可以分割出“位”来比较
- 数据有高低位之分，位之间有递进关系
- 每一位的数据范围不能太大，要可以用线性排序算法来排序


## 附录

```java
/**
 * 基数排序
 * <p>
 * 时间复杂度：O(n)
 * <p>
 * 空间复杂度：O(n)
 * <p>
 * 稳定性：稳定
 *
 * @author weijiaduo
 * @since 2022/9/6
 */
public class RadixSort implements Sort {

    @Override
    public void sort(int[] arr) {
        // 找到数组最大值
        int max = Integer.MIN_VALUE;
        for (int a : arr) {
            if (a > max) {
                max = a;
            }
        }

        // 计算基数大小
        int radix = 0;
        while(max > 0) {
            max = max / 10;
            radix++;
        }

        // 从低位到高位对数组进行排序
        int exp = 1;
        for (int i = 0; i < radix; i++) {
            countSort(arr, exp);
            exp *= 10;
        }
    }

    /**
     * 计数排序
     *
     * @param arr 数组
     * @param exp 指数
     */
    private void countSort(int[] arr, int exp) {
        // 统计每个数字（0-9）的次数
        int[] counts = new int[10];
        for (int a : arr) {
            counts[(a / exp) % 10]++;
        }

        // 累计数字的次数和
        for (int i = 1; i < counts.length; i++) {
            counts[i] += counts[i - 1];
        }

        // 更新排序结果到原数组
        int[] copy = Arrays.copyOf(arr, arr.length);
        for (int i = copy.length - 1; i >= 0; i--) {
            int index = (copy[i] / exp) % 10;
            counts[index]--;
            arr[counts[index]] = copy[i];
        }
    }

}
```
