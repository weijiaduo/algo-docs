# BF 算法

BF 算法，即 Brute Force，暴力匹配算法，也叫朴素匹配算法。

## 一、原理

所谓的暴力匹配，就是：

<!--more-->

- 检查主串的所有子串，看是否与模式串匹配

比如，主串是 `ababababca`，模式串是 `abababca`。

- 从前往后，依次检查主串的每一个子串是否跟模式串匹配

第 1 轮（比较第 1 个子串）：

```
   匹配失败
      |
      v
ababababca
abababca
```

第 2 轮（比较第 2 个子串）：

```
匹配失败
 |
 v
ababababca
 abababca
```

第 3 轮（比较第 3 个子串）：

```
      匹配成功
         |
         v
ababababca
  abababca
```

BF 算法就是通过匹配全部子串的方式，来实现字符串匹配。

## 二、分析

假设主串长度是 n，模式串长度是 m。

那么匹配次数（即主串中长度为 m 的子串个数）是：

```
n - m + 1
```

每次匹配的时间复杂度是 `O(m)`，所以总的时间复杂度是：

```
O((n - m + 1) * m), 即 O(n * m)
```

## 三、应用

虽然 BF 算法的时间复杂度比较高，但是它在实际应用中还是会经常用到：

- 很多场景下，主串和模式串的长度都不会太长，所以时间复杂度也不会太高
- BF 算法思想简单，实现起来也比较简单，代码不容易出错，调试起来也简单

所以，大部分的简单场景下，BF 算法是可以满足需求的。

# 参考

> https://time.geekbang.org/column/article/71187
>
> https://www.zhihu.com/question/21923021

# 附录

```java
/**
 * 暴力搜索法
 *
 * @author weijiaduo
 * @since 2023/3/28
 */
public class BruteForceSearch implements Search {

    @Override
    public int search(String pat, String txt) {
        int n = txt.length();
        int m = pat.length();
        for (int i = 0; i <= n - m; i++) {
            int j = 0;
            for (; j < m; j++) {
                if (txt.charAt(i + j) != pat.charAt(j)) {
                    break;
                }
            }
            // 匹配成功
            if (j == m) {
                return i;
            }
        }
        return -1;
    }

}
```