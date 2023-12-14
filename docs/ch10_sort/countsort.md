# 计数排序

## 一、算法描述

### 1.1 核心思想

- 计数排序是桶大小为 1 的桶排序的一种特殊情况
- 由于桶大小为 1，所以桶内都是相同的值
- 桶内都是相同的值，无需桶内排序，只需要记录数据频率
- 最后排序时，按照数据频率将数据填充回原数组

### 1.2 细节解释

<!--more-->

比如说，数组 `[6, 0, 5, 0, 9, 4, 1, 7, 4]`。

数据范围是 `[0, 9]`，所以可以划分为 10 个桶，每个桶大小是 1。

这里的桶直接用数组替代，因为只需要计数就够了，不需要保留原数据。

```
  0   1   2   3   4   5   6   7   8   9
 ___ ___ ___ ___ ___ ___ ___ ___ ___ ___
|   |   |   |   |   |   |   |   |   |   |
```

然后遍历排序数组，将数字频率统计到计数数组中：

```
  0   1   2   3   4   5   6   7   8   9
 ___ ___ ___ ___ ___ ___ ___ ___ ___ ___
| 2 | 1 | 0 | 0 | 2 | 1 | 1 | 1 | 0 | 1 |
```

桶内的数值表示的是对应数字的频率。

这里可以通过扫描计数数组，填充原数组就能排好序了：

```java
int k = 0;
for (int i = 0; i < counts.length; i++) {
    for (int j = 0; j < counts[i].length; j++) {
        arr[k++] = i;
    }
}
```

简单点说，计数排序的处理过程就是：

1. 统计所有不同数字的出现频率
2. 按照频率将数字填充到数组

处理过程不复杂，关键就在于利用了数字的天然有序性。


### 1.3 频率累计和

按照数字频率填充数字回原数组时，可以利用频率累计和来实现。

比如上面的计数数组，将其从左至右累计起来，可得到：

```
  0   1   2   3   4   5   6   7   8   9
 ___ ___ ___ ___ ___ ___ ___ ___ ___ ___
| 2 | 3 | 3 | 3 | 5 | 6 | 7 | 8 | 8 | 9 |
```

累计和 `counts[i]` 的意思就是，小于等于 `i` 的值有 `counts[i]` 个。

利用这个特性，可以将数字 x 直接定位到它最终排序好的位置。

比如 `x = 4`，`counts[4] = 5`，说明小于等于 x 的有 5 个。

那么 x 排好序后的位置就是第 5 位，放到数组中就是 `arr[4] = x`。

```
  0   1   2   3   4   5   6   7   8
 ___ ___ ___ ___ ___ ___ ___ ___ ___
|   |   |   |   | x |   |   |   |   |
```

但是如果接着遍历时又碰到一个 `y = 4`，这时应该放哪里呢？

难道也放 `arr[4]` 吗？这会覆盖掉上一个 x 的，不可以放。

实际上，在放入 x 时，需同时修改累计和，将其减一 `count[4]--`，表示已经有 1 个数字放进去了。

所以当遍历到 y 时，实际上此时 `counts[4] = 4`，所以 y 的位置应该是 `arr[3]`：

```
  0   1   2   3   4   5   6   7   8
 ___ ___ ___ ___ ___ ___ ___ ___ ___
|   |   |   | y | x |   |   |   |   |
```

通过频率累计和的方式，可以逐个将数据摆到正确的位置。


## 二、算法实现

### 2.1 划分成宽为 1 的桶

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

// 初始化计数范围
int n = max - min + 1;
int[] counts = new int[n];
```

### 2.2 将数据映射到桶内

通大小是1，都是相同值，不用记录数据，只需要计数就行：

```java
// 统计数字的频率
for (int num : arr) {
    int index = num - min;
    counts[index]++;
}
```

### 2.3 统计频率累计和

```java
// 频率累计和
for (int i = 1; i < counts.length; i++) {
    counts[i] += counts[i - 1];
}
```

### 2.4 倒序摆放数据

倒序的作用是为了保证数组数据的稳定性。

如果不需要稳定性，正序遍历也是可以的。

```java
// 倒序遍历获取排序结果
int[] copy = Arrays.copyOf(arr, arr.length);
for (int i = copy.length - 1; i >= 0; i--) {
    int index = copy[i] - min;
    counts[index]--;
    arr[counts[index]] = copy[i];
}
```


## 三、算法分析

### 3.1 时间复杂度

- 最好时间复杂度：O(n)
- 最坏时间复杂度：O(n)
- 平均时间复杂度：O(n)

### 3.2 空间复杂度

- 空间复杂度：O(n)
- 非原地算法

### 3.3 稳定性

- 稳定排序算法（倒序摆放）


## 四、适用场景

- 数据范围（max - min）比较小
- 数据范围（max - min）比数据量（n）要小得多


## 附录

```java
/**
 * 计数排序
 * <p>
 * 时间复杂度：O(n)
 * <p>
 * 空间复杂度：O(n)
 * <p>
 * 稳定性：稳定
 *
 * @author weijiaduo
 * @since 2022/9/5
 */
public class CountSort implements Sort {

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

        // 初始化计数范围
        int n = max - min + 1;
        int[] counts = new int[n];

        // 统计数字的数量
        for (int num : arr) {
            int index = num - min;
            counts[index]++;
        }

        // 累计数量和
        for (int i = 1; i < counts.length; i++) {
            counts[i] += counts[i - 1];
        }

        // 倒序遍历获取排序结果
        int[] copy = Arrays.copyOf(arr, arr.length);
        for (int i = copy.length - 1; i >= 0; i--) {
            int index = copy[i] - min;
            counts[index]--;
            arr[counts[index]] = copy[i];
        }
    }

}
```
