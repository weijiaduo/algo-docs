# RK 算法

RK 算法全称是 Rabin-Karp 算法，由它的两位发明者 Rabin 和 Karp 的名字来命名。

## 一、原理

暴力匹配算法最大的问题就是，每次都需要对子串的每个字符进行匹配判断，重复工作非常多。

RK 算法的思想就是，不去匹配每个字符，而是将子串映射成一个哈希数值，然后利用这个哈希值去直接匹配。

利用子串哈希值匹配的过程类似这样：

```
pat: abc          = 963        -- 模式串哈希值
txt: ababababca
sub: aba          = 652
      bab         = 728
       aba        = 31
        bab       = 314
         aba      = 201
          bab     = 442
           abc    = 963        -- 匹配成功
            bca   = 864
```

将子串映射成一个哈希值后，如果模式串哈希值和子串哈希值相等，那基本上就能说明子串和模式串匹配成功了。

而且因为哈希值是数字，数字之间的比较是非常快速的，所以 RK 算法的效率就比暴力算法高了很多。

但是有一个问题，就是如何计算子串的哈希值呢？如果需要遍历每个字符来计算，那么效率就和暴力算法一样了，没什么意义。

因此 RK 算法采用了一种更高效的计算方式，那就是类似滑动窗口的哈希算法（这里假设字母哈希是字母本身）：

```
txt: ababababca
sub: aba          = a*26*26 + b*26 + a
      bab         =           b*26*26 + a*26 + b
       aba        =                     a*26*26 + b*26 + a
        bab       =                               b*26*26 + a*26 + b
        ...
```

原理很简单，计算下一个子串的哈希值时，可以利用上一个子串的哈希值算出来，避免了每次都遍历所有字符。

1. 首先减去第一个字符的哈希值
2. 然后乘以进制数（比如例子里的 26）
3. 最后加上新增字符的哈希值

这样就能得到下一个子串的哈希值了，这种哈希计算的思路就像是滑动窗口一样，每次滑动一点点距离。

不过有一点需要注意的是，如果子串的长度很长，那么哈希值可能会超出数字的表示范围（溢出），因此需要对哈希值取模。

```
txt: ababababca
sub: aba          = (a*26*26 + b*26 + a) % MOD
      bab         =           (b*26*26 + a*26 + b) % MOD
       aba        =                     (a*26*26 + b*26 + a) % MOD
        bab       =                               (b*26*26 + a*26 + b) % MOD
        ...
```

其中 MOD 是一个比较大的质数，这既可以保证哈希值不会超出数字的表示范围（溢出），又能尽量避免哈希冲突。

## 二、验证

使用哈希值来进行匹配，虽然效率很高，但是也有可能出现哈希值相等，但是子串和模式串并不相等的情况。

因此在哈希值相等的情况下，还需要验证是否真的匹配成功了，验证的方式分为 2 种：

- 蒙特卡洛算法：只验证哈希值是否相等，不验证子串和模式串是否相等
- 拉斯维加斯算法：先验证哈希值是否相等，如果相等再验证子串和模式串是否相等

蒙特卡洛算法的优点是简单高效，但是缺点是可能会出现误判，即哈希值相等，但是子串和模式串并不相等。

拉斯维加斯算法的优点是能够避免误判，但是缺点是需要多一次验证，效率会降低。

## 三、分析

所有子串的哈希值，只需要遍历一轮主串就能计算出来，因此时间复杂度是 O(n)。

每个子串与模式串的匹配，只需要比较一次哈希值，因此时间复杂度是 O(1)。

因此总的时间复杂度是 O(n)。

空间复杂度只需要存储一个哈希值，因此是 O(1)。

# 参考

> https://time.geekbang.org/column/article/71187

# 附录

```java
/**
 * Rabin Karp 算法，又称指纹字符串查找算法
 * <p>
 * 复杂度：复杂度：最好时间 O(n) 最坏时间 O(mn) 空间 O(1)
 *
 * @author weijiaduo
 * @since 2023/3/30
 */
public class RabinKarpSearch implements Search {

    /**
     * 进制数
     */
    private static final int R = 10;
    /**
     * 对哈希值取余，避免溢出
     */
    private long mod = 99999999999997L;

    @Override
    public int search(String pat, String txt) {
        int m = pat.length(), n = txt.length();
        if (m == 0 || n < m) {
            return -1;
        }

        mod = longRandomPrime();
        long phash = 0, thash = 0, rm = 1;
        for (int i = 0; i < m - 1; i++) {
            phash = (R * phash + pat.charAt(i)) % mod;
            thash = (R * thash + txt.charAt(i)) % mod;
            // 首字符的幂值 R^(M-1)
            rm = (R * rm) % mod;
        }
        phash = (R * phash + pat.charAt(m - 1)) % mod;

        for (int i = m - 1, j = 0; i < n; i++, j++) {
            // 追加新字符的哈希值
            thash = (R * thash + txt.charAt(i)) % mod;
            // 验证子串[j, i]与模式串是否匹配
            if (phash == thash && check(pat, txt, j, m)) {
                return j;
            }
            // 减去首字符的哈希值
            thash = (thash - txt.charAt(j) * rm + mod) % mod;
        }
        return -1;
    }

    /**
     * 校验两个字符串是否相等
     *
     * @param pat    模式串
     * @param txt    主串
     * @param offset 主串起始索引
     * @param length 长度
     * @return true/false
     */
    private boolean check(String pat, String txt, int offset, int length) {
        for (int i = 0; i < length; i++) {
            if (pat.charAt(i) != txt.charAt(offset + i)) {
                return false;
            }
        }
        return true;
    }

    /**
     * @return 随机大素数
     */
    private static long longRandomPrime() {
        // 素数的大小要随着测试用例变化而修改
        BigInteger prime = BigInteger.probablePrime(42, new Random());
        return prime.longValue();
    }

}
```
