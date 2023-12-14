# 二分查找

## 一、代码模板

常见的二分查找模板代码如下：

```java
left = low, right = high;
while (left <= right) {
    int mid = left + (right - left) / 2;
    if (满足条件) {
        right = mid - 1;
    } else {
        left = mid + 1;
    }
}
return left; // 或 right
```

在二分查找中，有几个需要注意的地方：

<!--more-->

- 循环条件（`while (left <= right)`）
- 中值计算（`mid`）
- 区间收缩（`left = mid + 1`、`right = mid - 1`）
- 返回值（`left`、`right`）

这些地方写错了就很容易导致最后的结果不对。

## 二、循环条件

在二分查找中，常用的循环条件有 2 种：

- `while (left <= right)`
- `while (left < right)`

什么时候用 `<=`？什么时候用 `<` 呢？实际上这和区间定义有关。

常见的区间定义可以分为 2 种：

- 左闭右闭 `[low, high]`
- 左闭右开 `[low, high)`

不同的区间定义，循环条件的写法也不一样。为什么呢？这是因为，

- 区间定义属于是循环不变量，在整个循环过程中要一直保持有意义

所以循环条件的写法，其实是根据区间是否有意义来决定的：

- 左闭右闭：当 `left==right` 时，`[left, right]` 有意义，因此有 `=` 号
- 左闭右开：当 `left==right` 时，`[left, right)` 无意义，因此无 `=` 号

因此，循环条件上要不要 `=` 号，取决于区间定义是否有意义。

- 左闭右闭 `[low, high]`：`while (left <= right)`
- 左闭右开 `[low, high)`：`while (left < right)`

正常来说，这两种写法都是可行的，不过还是，

- 推荐使用左闭右闭 `[low, high]`：`while (left <= right)`

因为两边都是闭区间，是对称的，所以两边的区间操作也是对称的，不容易出错。


## 三、中值计算

一般来说，二分中值的计算方式有以下几种：

1. `mid = (left + right) / 2`
2. `mid = left + (right - left) / 2`
3. `mid = left + ((right - left) >> 1)`

使用哪种方式比较好呢？

- 推荐使用第 2 种：`mid = left + (right - left) / 2`

第 1 种存在整数溢出问题，所以用的比较少。

第 3 种和第 2 种本质上是一样的，而且从理论上说移位 `>>` 比除法 `/` 还要快。

但是 `>>` 的优先级比 `+` 号低，需要加多一层括号，而括号很容易遗漏，不够直观。

所以从整体上来说，还是第 2 种写法更好一点。


## 四、区间收缩

二分查找实际上就是收缩左右边界，直到找到满足条件的值为止。

不过区间收缩的方式有好几种，包括：

- `left = mid + 1`
- `right = mid - 1`
- `left = mid`
- `right = mid`

这几种方式可以组合成各种情况，用不好的话，就很容易造成死循环。

比如说，使用 `left = mid` 和 `right = mid - 1` 来这样组合：

```java
while (left <= right) {
    int mid = left + (right - left) / 2;
    if (arr[mid] >= x) {
        left = mid;
    } else {
        right = mid - 1;
    }
}
```

假设此时 `left = 0, right = 1, arr = {1, 2}, x = 1`，那么代码就会陷入死循环。

死循环的原因就是左右边界没有收缩，导致一直无法达到循环结束条件。

- 死循环基本都是出现在 `left = mid` 或 `right = mid` 上

因为 `left = mid` 和 `right = mid` 在某些情况下可能会导致区间一直没有收缩。

区间收缩的正确使用方式应该是怎么样呢？这就要看区间定义了：

- 左闭右闭 `[low, high]`：`left = mid + 1` 和 `right = mid - 1`
- 左闭右开 `[low, high)`：`left = mid + 1` 和 `right = mid`
- 左开右闭 `(low, high]`：`left = mid` 和 `right = mid - 1`
- 左开右开 `(low, high)`：`left = mid` 和 `right = mid`

比如说，要寻找第一个 `>= x` 的位置，不同区间的写法如下：

1）左闭右闭 `[low, high]`

```java
while (left <= right) {
    // left <= mid <= right
    int mid = left + (right - left) / 2;
    if (arr[mid] >= x) {
        // [left, mid - 1]，因为 mid <= right，所以区间在收缩
        right = mid - 1;
    } else {
        // [mid + 1, right]，因为 mid >= left，所以区间在收缩
        left = mid + 1;
    }
}
```

2）左闭右开 `[low, high)`

```java
while (left < right) {
    // left <= mid < right
    int mid = left + (right - left) / 2;
    if (arr[mid] >= x) {
        // [left, mid)，因为 mid < right，所以区间在收缩
        right = mid;
    } else {
        // [mid + 1, right)，因为 mid >= left，所以区间在收缩
        left = mid + 1;
    }
}
```

区间的收缩方式要和区间定义相匹配，才能保证区间收缩的正确性。

而循环内区间收缩的核心思想就是：

- 不管哪种区间定义，都要保证每轮循环结束后，区间在缩小

只要保证区间一直在缩小，那么就不会出现死循环的问题。


## 五、返回值

在一般的二分查找中，返回值有时候是 `left`，有时候是 `right`。

比如说，要寻找第一个 `>= x` 的位置，返回值就是 `left`：

```java
int firstGreatEqualSearch(int[] arr, int x) {
    int n = arr.length;
    // 循环不变量（循环中始终成立的条件）：
    // arr[left - 1] < x
    // arr[right + 1] >= x
    int left = 0, right = n - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] >= x) {
            right = mid - 1;
        } else {
            left = mid + 1;
        }
    }
    // 循环终止条件：right + 1 == left
    return left; // 或 right + 1
}
```

要寻找最后一个 `<= x` 的位置，返回值就是 `right`：

```java
int lastLessEqualSearch(int[] arr, int x) {
    int n = arr.length;
    // 循环不变量（循环中始终成立的条件）：
    // arr[left - 1] <= x
    // arr[right + 1] > x
    int left = 0, right = n - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] <= x) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    // 循环终止条件：right + 1 == left
    return right; // 或 left - 1
}
```

按照这种模板写代码的话，就需要，

- 末尾返回值根据循环不变量来决定是 `left` 还是 `right`

但是，这种末尾返回值的写法，需要根据实际情况来调整，不好理解，写起来也容易出错。

因此，可以换另外一种写法：

- 将末尾返回值，改成在循环体内判断成立条件，然后返回值

比如，要寻找第一个 `>= x` 的位置：

```java
int firstGreatEqualSearch(int[] arr, int x) {
    int n = arr.length;
    int left = 0, right = n - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] >= x) {
            // 改成在循环体内判断成立条件，然后返回值
            if (mid == 0 || arr[mid - 1] < x) {
              return mid;
            }
            right = mid - 1;
        } else {
            left = mid + 1;
        }
    }
    // 注意，当所有元素都不满足时，这里返回了 -1
    // 而上面的代码则返回的不一样，上面返回的是 n
    return -1;
}
```

这种循环体内返回的写法虽然没那么简洁，但是至少它逻辑清晰、简单易懂。

因此，除非对末尾返回值的写法很熟练了，否则推荐用在循环中返回值的写法。


## 总结

| 区间定义 | 左闭右闭 `[low, high]` | 左闭右开 `[low, high)` |
|:--|:--|:--|
| 初始边界 | low = 0, high = n - 1 | low = 0, high = n |
| 循环条件 | left <= right | left < right |
| 中值计算 | mid = left + (right - left) / 2 | mid = left + (right - left) / 2 |
| 收缩方式 | left = mid + 1, right = mid - 1 | left = mid + 1, right = mid |
| 终止情况 | left == right + 1 | left == right |
| left 范围 | [low, high + 1] | [low, high] |
| right 范围 | [low - 1, high] | [low, high] |


## 参考

> https://www.hello-algo.com/chapter_searching/binary_search/
>
> https://www.programmercarl.com/0704.%E4%BA%8C%E5%88%86%E6%9F%A5%E6%89%BE.html
>
> https://time.geekbang.org/column/article/42733


## 附录

### 等于给定值的位置

```java
public int equalSearch(int[] arr, int x) {
    int n = arr.length;
    int low = 0, high = n - 1;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (arr[mid] == x) {
            return mid;
        } else if (arr[mid] < x) {
            low = mid + 1;
        } else {
            high = mid - 1;
        }
    }
    return -1;
}
```

### 第一个等于给定值的位置

```java
int firstEqualSearch(int[] arr, int x) {
    int n = arr.length;
    int left = 0, right = n - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] == x) {
            if (mid == 0 || arr[mid - 1] != x) {
              return mid;
            }
            right = mid - 1;
        } else if (arr[mid] < x) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    return -1;
}
```

### 最后一个等于给定值的位置

```java
int lastEqualSearch(int[] arr, int x) {
    int n = arr.length;
    int left = 0, right = n - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] == x) {
            if (mid == n - 1 || arr[mid + 1] != x) {
              return mid;
            }
            left = mid + 1;
        } else if (arr[mid] < x) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    return -1;
}
```

### 第一个大于等于给定值的位置

```java
int firstGreatEqualSearch(int[] arr, int x) {
    int n = arr.length;
    int left = 0, right = n - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] >= x) {
            if (mid == 0 || arr[mid - 1] < x) {
              return mid;
            }
            right = mid - 1;
        } else {
            left = mid + 1;
        }
    }
    return -1;
}
```

### 最后一个小于等于给定值的位置

```java
int lastLessEqualSearch(int[] arr, int x) {
    int n = arr.length;
    int left = 0, right = n - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] <= x) {
            if (mid == n - 1 || arr[mid + 1] > x) {
              return mid;
            }
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    return -1;
}
```
