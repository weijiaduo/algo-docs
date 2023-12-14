# KMP 算法

KMP 算法是一种字符串匹配算法，可以在 `O(n+m)` 的时间复杂度内实现两个字符串的匹配。

KMP 算法是根据三位作者（D.E.Knuth，J.H.Morris 和 V.R.Pratt）的名字来命名的，全称是 Knuth-Morris-Pratt 算法。

## 一、暴力：一次滑一位

最简单的方式就是暴力逐位匹配，比如：

<!--more-->

```
// 第1轮
a b c a b c a b d
a b c a b d
```
```
// 第2轮
a b c a b c a b d
  a b c a b d
```
```
// 第3轮
a b c a b c a b d
    a b c a b d
```
```
// 第4轮
a b c a b c a b d
      a b c a b d
```

暴力匹配的思路很简单，就是：

- 每次匹配失败，模式串就往后移动一位，再重新开始匹配

这种方式非常慢，复杂度超高，不适合大规模文本的匹配。

## 二、想法：一次滑几位

既然暴力匹配失败后，一位一位地滑动很慢，那如果：

- 利用之前的匹配信息，匹配失败后，尽量一次滑多几位

这样的匹配方式是否能够提升性能？

比如：

```
// 第1轮
a b c a b c a b d
a b c a b d
```

很明显后面2位（`bc`）都不是 `a`，所以不用匹配了，直接滑过：

```
// 第2轮
a b c a b c a b d
      a b c a b d
```

如果每次都能滑多几位，那么就能减少匹配的轮数，速度会大大提升。

不过，假如能滑多几位，那么能滑多远呢？

- 滑到下一个可能匹配成功的位置

比如上面的例子，下一个可能匹配的位置（开头是 `a`）是：

```
              -----
T:     a b c | a b | c a b d
P:     a b c | a b | d 
P:        -> | a b | c a b d
              -----
```

观察发现：

- 上一轮已经匹配好的子串是：`abcab`
- 下一轮开始：`abcab` 的前缀子串（`ab`）和后缀子串（`ab`）刚好有交集的位置

其滑动效果就类似这样：

```
前缀子串    后缀子串
 ----       ----
| ab |  c  | ab |
 ----       ----
            ----       ----
   -->     | ab |  c  | ab |
            ----       ----
```

这个滑动，实际就是将「前缀子串」滑到「后缀子串」的位置，即 `ab` 的位置。

也就是说，只要找到这种相等前后缀子串，每次匹配失败后就能一次往后滑多几位。

## 三、实现：Pmt 数组的匹配

Pmt 数组，全称是：Partial Match Table，即部分匹配表。

- Pmt 数组记录了模式串每个子串的最长相等前后缀的长度
- Pmt 数组本质上是利用了相等前后缀子串的性质，来加快字符串的匹配

比如，模式串 `abcabd` 的 Pmt 数组为：

```
P:   a b c a b d
Pmt: 0 0 0 1 2 0
```

其中，

- `Pmt[3] = 1` 表示，`abca` 的最长相等前后缀的长度为 `1`，即 `a`
- `Pmt[4] = 2` 表示，`abcab` 的最长相等前后缀的长度为 `2`，即 `ab`

利用 Pmt 数组，字符串匹配过程可以这样：

```java
if (T[i] == P[j]) {
   // 匹配成功，i 和 x 同时向后移动一位
   i++;
   j++;
} else {
   // 匹配失败
   if (j > 0) {
      // 模式串滑到最长相等前后缀的位置重新开始
      j = Pmt[j - 1];
   } else {
      // 第一个字符就匹配失败，i 向后移动一位
      i++;
   }
}
```

其中，`j = pmt[j - 1]` 实际上就是将模式串滑动到最长相等前后缀的位置。

Pmt 数组的匹配过程类似这样：

```
            匹配失败
               |
               v
T:   a b c a b c a b d
P:   a b c a b d
Pmt: 0 0 0 1 2 0
```

此时，`j = 5`，`pmt[j - 1] = 2`。

然后保持主串位置不变，模式串往后滑动，从索引 2 开始重新匹配：

```
            重新开始
               |
               v
T:   a b c a b c a b d
P:         a b c a b d
Pmt:       0 0 0 1 2 0
```

按照这种方式不断匹配下去，直到匹配结束为止。

- 匹配过程中，主串索引只会不断递增，而不会回头

所以，相比于暴力的逐位匹配，Pmt 数组可以极大地提升匹配速度：

- 时间复杂度：暴力匹配是 `O(m * n)`；而 Pmt 的是 `O(m + n)`
- 空间复杂度：暴力匹配是 `O(1)`；而 Pmt 的是 `O(m)`

虽然空间复杂度从 `O(1)` 升到了 `O(m)`，但是模式串 `m` 一般都比较小，所以也不是很大的问题。

而对于大文本匹配来说，时间复杂度从 `O(m * n)` 降到了 `O(m + n)`，则是一个非常大的提升。

## 四、关键：Pmt 数组的构造

Pmt 数组确实可以提升匹配速度，那么如何求得 Pmt 数组呢？

- 利用动态规划，通过 `Pmt[0]` ... `Pmt[i]` 来求 `Pmt[i + 1]`

假设已经知道了 `Pmt[i-1] = k`，即意味着模式串：`P[0..k-1]` == `P[i-k..i-1]`：

```
          ----------相等----------
         |                       |
         v                       v
    ------------         --------------
P: | 0  ..  k-1 | k ... | i-k  ..  i-1 | i
    ------------         --------------
```

那么对于 `Pmt[i]` 而言，可分为 2 种情况：

(1) `P[k]` == `P[i]`

很明显，这种情况下 `Pmt[i]` 只需要在上一个的基础上扩展一位就行了，即：

- `Pmt[i]` = `k + 1`

扩展后的效果是这样的：

```
          ----------相等----------
         |                       |
         v                       v
    ---------------       -----------------
P: | 0  ..  k-1  k | ... | i-k  ..  i-1  i |
    ---------------       -----------------
```

(2) `P[k]` != `P[i]`

这个时候，为了使得前后缀相等，就需要缩短前后缀长度，直到找到 `P[x]` == `P[i]` 为止：

```
          ------------------相等-----------------
         |                                      |
         v                                      v
    ------------                         --------------
P: | 0  ..  x-1 | x ... k-1 ... i-k ... | i-x  ..  i-1 | i
    ------------                         --------------
```

其中，每次缩短，新的前后缀都要求满足这样的关系：

- `P[0..x-1]` == `P[i-x..i-1]`
- `P[0..x-1]` 是 `P[0..k-1]` 的前缀
- `P[i-x..i-1]` 是 `P[i-k..i-1]` 的后缀

而且由于前提条件中 `P[0..k-1]` == `P[i-k..i-1]`，因此：

- `P[0..x-1]` 是 `P[0..k-1]` 的前缀
- `P[i-x..i-1]` 是 `P[0..k-1]` 的后缀

简单来说就是：

- `P[0..x-1]` 是 `P[0..k-1]` 的一个相等前后缀

为了求最长的相等前后缀，x 应该尽量地大，否则找到的就不是最长的相等前后缀了。

- 要保证 x 尽量地大，因此 `P[0..x-1]` 实际上就是 `P[0..k-1]` 最长相等前后缀

所以，找 `P[x]` == `P[i]` 的过程类似这样：

```java
while (x > 0 && P[x] != P[i]) {
    x = pmt[x - 1];
}
```

就是一个不断递归寻找上一级最长相等前后缀，再匹配的过程。

综合上面的 2 种情况，Pmt 数组构造对应的代码就是这样的：

```java
if (P[j] == P[i]) {
   // 最长相等前后缀长度
   pmt[i] = j + 1;
   i++;
   j++;
} else {
   if (j != 0) {
      // 找次长相等前后缀递归匹配
      j = pmt[j - 1];
   } else {
      // 一开始就匹配失败了
      pmt[i] = 0;
      i++;
   }
}
```

至此，Pmt 数组的构造就完成了。

## 五、改进：Next 数组

在 Pmt 算法中，一旦第 j 位匹配失败，就会用 `Pmt[j-1]` 进行回溯，所以：

- 为了编程方便，直接将 Pmt 数组向右移动 1 位，得到 Next 数组

Next 数组的结果类似这样：

```
P:    a b c a b d
Pmt:  0 0 0 1 2 0
Next:   0 0 0 1 2
```

至于第 0 位的值，可以赋值为 -1，这样做只是为了编程方面，并无其他意义。

```
P:     a b c a b d
Pmt:   0 0 0 1 2 0
Next: -1 0 0 0 1 2
```

基于 Next 数组的算法和基于 Pmt 的算法是一样的，只是在编程上更加方便了。

Next 数组的匹配算法如下：

```java
if (j == -1 || P[j] == P[i]) {
   // 匹配主串和模式串的下一个字符
   i++;
   j++;
} else {
   // 一次性滑到最长相等前后缀的位置开始匹配
   j = next[j];
}
```

构造 Next 数组的算法如下：

```java
if (j == -1 || P[j] == P[i]) {
   i++;
   j++;
   // 最长相等前后缀长度，索引右移了1位
   next[i] = j;
} else {
   // 找次长相等前后缀递归匹配
   j = next[j];
}
```

Next 数组和 Pmt 数组并没有本质区别，算法复杂度是一样的。

Next 数组改变的仅仅是编程的便利性，在实际中，Next 数组会更常用一些。

# 参考

> https://time.geekbang.org/column/article/71845
> 
> https://www.zhihu.com/question/21923021
> 
> https://zhuanlan.zhihu.com/p/83334559
>
> https://oi-wiki.org/string/kmp/

# 附录

## 基于 Pmt 数组的 KMP 算法实现

```java
/**
 * KMP(Partial Match Table) 算法
 *
 * @author weijiaduo
 * @since 2023/3/28
 */
public class PmtKMP {

    public int search(String pat, String txt) {
        int[] pmt = getPmt(pat);
        int i = 0, j = 0;
        int m = pat.length(), n = txt.length();
        while (i < n && j < m) {
            if (txt.charAt(i) == pat.charAt(j)) {
                // 匹配成功，匹配下一个字符
                i++;
                j++;
            } else {
                // 匹配失败
                if (j != 0) {
                    // 模式串滑到最长相等前后缀的位置重新开始
                    j = pmt[j - 1];
                } else {
                    // 第一个字符就匹配失败了
                    i++;
                }
            }
        }
        // 匹配成功
        if (j == m) {
            return i - j;
        }
        return -1;
    }

    /**
     * 获取 Partial Match Table
     * <p>
     * pmt[i] 表示 pat[0...i] 的最长相等前后缀的长度
     *
     * @param pat 模式串
     * @return pmt 数组
     */
    private int[] getPmt(String pat) {
        int m = pat.length();
        int[] pmt = new int[m];
        pmt[0] = 0;
        int i = 1, j = 0;
        while (i < m) {
            if (pat.charAt(i) == pat.charAt(j)) {
                // 最长相等前后缀长度
                pmt[i] = j + 1;
                i++;
                j++;
            } else {
                if (j != 0) {
                    // 找次长相等前后缀递归匹配
                    j = pmt[j - 1];
                } else {
                    // 一开始就匹配失败了
                    pmt[i] = 0;
                    i++;
                }
            }
        }
        return pmt;
    }

}
```

## 基于 Next 数组的 KMP 算法实现

```java
/**
 * KMP(Next Match Table) 算法
 *
 * @author weijiaduo
 * @since 2023/3/28
 */
public class NextKMP {

    public int search(String pat, String txt) {
        int[] next = getNext(pat);
        int i = 0, j = 0;
        int m = pat.length(), n = txt.length();
        while (i < n && j < m) {
            if (j == -1 || txt.charAt(i) == pat.charAt(j)) {
                // 匹配主串和模式串的下一个字符
                i++;
                j++;
            } else {
                // 一次性滑到最长相等前后缀的位置开始匹配
                j = next[j];
            }
        }
        // 匹配成功
        if (j == m) {
            return i - j;
        }
        return -1;
    }

    /**
     * 获取 Next Match Table
     * <p>
     * next[i] 表示模式串下一次进行比较的索引位置
     * <p>
     * next[i] 也表示 pat[0...j-1] 的最长相等前后缀的长度
     * <p>
     * next 数组实际就是 pmt 数组往右移动一位得到的
     *
     * @param pat 模式串
     * @return next 数组
     */
    private int[] getNext(String pat) {
        int m = pat.length();
        int[] next = new int[m];
        next[0] = -1;
        int i = 0, j = -1;
        while (i < m - 1) {
            if (j == -1 || pat.charAt(i) == pat.charAt(j)) {
                i++;
                j++;
                // 最长相等前后缀长度，索引右移了1位
                next[i] = j;
            } else {
                // 找次长相等前后缀递归匹配
                j = next[j];
            }
        }
        return next;
    }

}
```
