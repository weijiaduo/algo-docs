# 散列表

## 一、什么是散列表？

### 1.1 定义

散列表（Hash Table），也称为哈希表，其定义如下：

- 散列表是一种能够根据关键字，直接访问到值的数据结构
- 散列表建立了关键字和存储地址之间的一种直接映射关系

其中，关键字称为 `Key`，对应的值称为 `Value`。

因此散列表也可以说是：

<!--more-->

- 将 `Key` 直接「映射」到一张「记录表」上，以此找到 `Value` 的地址

其中，「映射」方式称为散列函数，存储记录的表就称为哈希表。

哈希表的映射逻辑类似这样：

```
Key                    散列函数                  散列表

                                                ________ 
                                      -------> | value4 |
                                      |         ________ 
key1   ---------                      |        |        |
               |                      |         ________ 
key2   ---------                      -------> | value1 |
               |                      |         ________  
key3   -------------->  hash(key)  ----------> | value5 |
               |                      |         ________
key4   ---------                      |        |        |
               |                      |         ________
key5   ---------                      -------> | value2 |
                                      |         _________ 
                                      -------> | value3 |
```

一般来说：

- 散列表底层存储结构是数组，散列函数负责将 `Key` 映射到数组索引，即 `Value` 地址

映射的地址，取决于散列函数，不同散列函数可能会不一样。

### 1.2 结构

常见的散列表结构（数组 + 链表）如下：

```
索引   数组
       ___ 
 0    |   |
       ___        ___        ____
 1    | 8 | ---> | 1 | ---> | 22 |
       ___ 
 2    | 9 |
       ___        ____ 
 3    | 3 | ---> | 17 |
       ___        ____        ____        ____ 
 4    | 4 | ---> | 32 | ---> | 18 | ---> | 46 |
       ___
 5    |   |
       ___ 
 6    | 6 |
```

这里的散列函数是取余 `key % 7`，例如 `22 % 7 = 1`，那么 `22` 就放在了数组索引为 `1` 的位置上。

### 1.3 例子

举个更具体的例子来说，手机通讯录中，联系人是按照拼音的首字母排序的，暂记为通讯录 `[A - Z]`。

假如要找张三，其首字母是 `Z`，那么就能直接定位到通讯录中以 `Z` 开头的地方，再开始查找。

通讯录 `[A - Z]` 实际上就是散列表，将关键字（张三），映射到地址（`Z`）。


## 二、为什么要用散列表？

查找时，一般都是基于「比较」实现的，比如数组、链表、树等，都是遍历数据逐个「比较」，直到找到目标为止。

- 对于这些基于「比较」的数据结构，查找效率取决于「比较」次数，数据量越大，性能就会越差

因为数据量变大会导致「比较」次数变多，总耗时就会变长，所以散列表采用了另外一种方式来实现查找：

- 散列表不是基于「比较」实现的，而是将键直接「映射」到数据地址上

理想情况下，散列表的查找性能与数据量无关，不会因为数据量变大而导致性能降低，因此，

- 散列表的一个主要作用就是，解决大数据量情况下，快速定位值的问题

散列表带来的性能提升，不是凭空得来的，而是一种用空间换时间的实现方式。


## 三、如何实现散列表？

### 3.1 快速地址访问

散列表是通过 `Key` 得到 `Value` 地址后，访问地址才能拿到数据。

比如，`Key = 22` 通过计算得到了索引地址 `22 % 7 = 1`，但是 `1` 只是索引地址，不是数据，真正的数据是 `arr[1]`。

那么如果不能通过地址快速访问到数据，那散列表就没有意义了，所以有一个基本要求必须要满足：

- 散列表的实现，必须能通过映射地址快速定位到对应的数据

而数组刚好支持通过索引进行随机访问，提供了快速访问地址数据的手段，所以才会有：

- 散列表基本都是通过数组实现的，这实际上是利用了数组的随机访问特性

当然，只要是能够快速访问地址的数据结构，都可以用到散列表里，不一定是数组，只是数组刚好满足要求了而已。


### 3.2 散列设计

散列就是将关键字映射到地址，所以散列的设计至关重要，它有一些基本要求：

- 散列必须能够映射所有可能的关键字
- 散列对于同一个关键字，映射地址应该保持一致
- 散列应该尽可能的把映射地址均匀分布开来
- 散列应该尽量简单，能够短时间计算出来

这是散列的几个基本要求，除此之外还有：

- 散列设计还要包括散列冲突的解决策略


### 3.3 散列冲突

散列是将关键字映射到地址，而地址就是数组的索引，这存在一个问题：不同关键字映射的索引地址可能一样。

- 因为数组空间是有限的，而关键字可能是无限的，所以不管怎么设计散列映射，都不可避免的会出现冲突

所以，要求散列设计的时候，要同时考虑：散列冲突了怎么解决？

散列冲突的处理有多种方式，可以视实际情况选择。


### 3.4 负载扩容

当数组内的空间基本都用完时，散列冲突的概率会非常高。这个时候要怎么办？

- 空间不足时，就需要对散列表的数组进行扩容

那什么时候进行扩容呢？等到数组空间用完吗？那个时候已经太晚了，一般是在数组空间大部分都被占用时就开始扩容。

如何判断空间被占用了大部分？一般是使用负载因子来判断空间的占用比例：

- `负载因子 = 散列表中的元素 / 散列表的数组大小`，负载因子越大，表示空间占用比例越高

例如，散列表数组大小是 10，元素数量是 7，那么负载因子就是 `7 / 10 = 0.7`，表示大约有 70% 的空间已经被占用了。

所以，一般是根据负载因子来判断是否要数组扩容：

- 当负载因子达到某个阈值时，就可以认为空间不足，需要对数组扩容了


## 四、散列函数构造

散列函数，就是将关键字映射到地址的函数。可以记为：

```
Hash(key) = Addr
```

散列函数的构造要求，遵循的就是散列设计的要求。

常见的散列函数有：

### 4.1 直接定址法

直接取关键字的某个线性函数值，作为散列地址。

```
H(key) = a * key + b
```

线性映射不会造成散列冲突，适合关键字连续的情况。

若是关键字不连续，则会导致空位较多，浪费空间。

### 4.2 除留余数法

取关键字被某个不大于散列表表长 m 的数 p 除后所得的余数为散列地址。

```
H(key) = key % p
```

其中 `q < m`。

这种方式简单，最常用，但是因为对数数值取余的原因，有可能会出现散列冲突。

关键在于选好 p，不同的 p 对空间的利用是不同的。

### 4.3 数字分析法

取数码分布较为均匀的若干位作为散列地址。

```
H(key) = key & mask
```

比如说关键字是 10 位数，类似这样的：

```
1001234567
1002345678
1003456789
```

关键字中有些数位的码字是分布不均匀的，比如前 3 位 `100` 都是一样的。

这个时候就不要用 `100` 作为散列地址了，因为很容易就散列冲突了。

应该取比较均匀的其他数位作为散列地址，比如这里的后 7 位。

这种方法适合关键字集合已知的情况，如果关键字的分布情况变了，散列函数也需要重新构造。

### 4.4 平方取中法

取关键字的平方值的中间几位作为散列地址。

```
H(key) = (key * key) & mask
```

具体取多少位，视情况而定。

适用于关键字每一位取值都不够均匀，或者小于散列地址所需位数的情况。

这种方法得到的散列地址与关键字的每一位都有关系，使得散列地址分布比较均匀。

### 4.5 折叠法

将关键字分割成位数相同的几部分，然后取这几部分的叠加和作为散列地址。

```
H(key) = (key & mask1) | (key & mask2) | (key & mask3)

mask2 = mask1 << k, mask3 = mask2 << k
```

适用于关键字位数较多，且每一位的数字分布大致均匀的情况。


## 五、散列冲突处理

散列冲突是不可避免的，那一般怎么处理散列冲突呢？

常见的解决方法分为 2 种：

- 开放寻址法
- 拉链法

下面分别说明这 2 种方法。

### 5.1 开放寻址法

开放寻址法，指的是在其他散列地址空闲的位置，插入冲突的数据。

这就意味着，关键字散列到的地址，对应的数据不一定是它的，有可能是别人占用了。

其数学递推公式是：

```
hi = (H(key) + di) % m
```

其中，i = 1,2,3...，m 表示散列表长，di 为增量序列。

当选定某一增量序列后，对应的处理方法就确定了，常见的增量序列有以下几种。

#### 5.1.1 线性探测

线性探测，就是增量为 1 的序列 d = 1, 2, 3, ...

探测过程就是，如果出现冲突，就按顺序查看表中下一个单元，找到第一个空闲位置（直到遍历完整张表）。

比如散列函数是取余 `H(key) = key % 5`，当前数组是这样的：

```
     ___ 
0   |   |
     ___
1   | 1 |
     ___ 
2   |   |
     ___
3   |   |
     ___ 
4   | 4 |
```

此时插入一个 6，`6 % 5 = 1`，对应地址是 `1`，但是此时已经有人占用了。

按顺序往下找一个空位，找到了 `2` 的位置，然后插入：

```
     ___ 
0   |   |
     ___
1   | 1 |
     ___ 
2   | 6 |
     ___
3   |   |
     ___ 
4   | 4 |
```

接着再插入一个 15，`11 % 5 = 1`，还是地址 `1`。

按顺序往下找，直至找到了 `3`，最后放进去：


```
     ____ 
0   |    |
     ____
1   | 1  |
     ____ 
2   | 8  |
     ____
3   | 11 |
     ____ 
4   | 4  |
```

这种情况下，散列地址 `1` 实际上是占用了 `2` 和 `3` 的空间。

线性探测会导致同一个地址的数据聚集在一起，而且占用相邻地址的空间。

#### 5.1.2 平方探测

平方探测，就是增量为平方的序列 d = 1^2, 2^2, 3^2, 4^2, ...

线性探测会导致数据堆积，所以增量步长换成了它的平方，避免堆积问题。

缺点是不能探测到散列表上的所有位置，但至少能探测到一半。

#### 5.1.3 再散列

再散列，就是增量序列是用另一个散列函数生成的 `d = H2(key)`。

再散列的增量序列就是：

```
d1 = H2(H(key))
d2 = H2(d1)
d3 = H2(d2)
d4 = H2(d3)
...
```

比如说，再散列函数是 `H2(d) = (4 * d + 3) % 5`。

假如散列值地址是 `H(key) = 1`，那么它对应的增量序列就是：

```
H2(1) = (4 * 1 + 3) % 5 = 2
H2(2) = (4 * 2 + 3) % 5 = 1
```

也即是，冲突探测地址分别是 `[1, 1 + 2, 1 + 2 + 1] = [1, 3, 2]`。

再散列就是用额外的散列函数生成增量序列，达到寻找空位的目的。

#### 5.1.4 伪随机序列

伪随机序列，就是使用伪随机数作为增量序列。

为什么是伪随机数？因为散列函数的一个要求是：

- 同一个关键字，对应的地址必须相等

如果使用了真的随机数，那么同一个关键字映射的地址就不确定了。

所以只能用伪随机数。

### 5.2 拉链法

开放寻址法，是用别的地址来存储自己地址的数据，这就很容易导致冲突。

拉链法，不占用别人的空间，而是将相同地址的数据放到一条链表里。

其大概结构如下：

```
     ___ 
0   |   |
     ___        ___        ____
1   | 8 | ---> | 1 | ---> | 22 |
     ___ 
2   | 9 |
     ___        ____ 
3   | 3 | ---> | 17 |
     ___        ____        ____        ____ 
4   | 4 | ---> | 32 | ---> | 18 | ---> | 46 |
     ___
5   |   |
```

相同地址冲突的数据，会形成一条链表，这样就可以避免冲突的问题。

拉链法适用于经常进行插入和删除的情况。

### 小结

开放寻址法：

- 使用原本的数组空间，会占用到别的地址来存放同地址的数据
- 不能直接删除，否则会导致冲突寻址序列中断
- 适合当数据量比较小、装载因子小的情况

拉链法：

- 新开额外的链表空间，存放同地址的数据
- 适合存储大对象、大数据量的情况
- 适合经常进行插入和删除的情况


## 六、散列表的性能分析

散列表的查找效率取决于 3 个因素：

- 散列函数：散列函数直接关系到冲突的概率，冲突概率越高，查询效率就越低
- 处理冲突的方法：不同的处理冲突的方法，也会导致查询长度变化，从而影响查询效率
- 负载因子：直观上看，负载因子越大，散列表越满，冲突也就也多，查找的效率也就越低

散列表的查询性能，是实时变化的，随着数据分布情况和数据量而变化。

一般来说，散列表的平均查找长度依赖于负载因子，负载因子越大，平均查找长度就越大。

所以应该尽量避免负载因子过大。


## 七、散列表的特点

- 查找速度快：正常情况下是 `O(1)` 的查找效率
- 额外空间浪费：很难做到用完数组的空间，总是会有一些剩余空位
- 数据无序：散列表是通过关键字直接映射的地址，不保证数据在数组中的顺序
- 产生冲突：很难设计一个没有冲突的散列函数

## 八、散列表的应用场景

- 适合大数据量下快速查找
- 适合缓存，快速定位


## 总结

设计散列表时，有几个关键的地方：

- 散列函数的设计
- 散列冲突的处理
- 动态的负载扩容

散列函数的设计要求：

- 散列必须能够映射所有可能的关键字
- 散列对于同一个关键字，映射地址应该保持一致
- 散列应该尽可能的把映射地址均匀分布开来
- 散列应该尽量简单，能够短时间计算出来

散列冲突的处理：

开放寻址法：

- 使用原本的数组空间，会占用到别的地址来存放同地址的数据
- 不能直接删除，否则会导致冲突寻址序列中断
- 适合当数据量比较小、装载因子小的情况

拉链法：

- 新开额外的链表空间，存放同地址的数据
- 适合存储大对象、大数据量的情况
- 适合经常进行插入和删除的情况


## 参考

> http://data.biancheng.net/view/107.html
> 
> https://zhuanlan.zhihu.com/p/537528259
>
> http://t.zoukankan.com/Yunrui-blogs-p-11967732.html


## 附录

### 散列表-开放寻址法

```java
/**
 * 散列表-开放寻址法
 * <p>
 * 散列函数：折叠法 + 取余
 * <p>
 * 散列冲突：线性探测法
 */
public class OpenHashTable implements HashTable {

    /**
     * 散列表数组
     */
    Node[] table;
    /**
     * 元素数量
     */
    int size;
    /**
     * 负载因子
     */
    double factory;

    static class Node {
        final int key;
        int value;
        boolean deleted;

        public Node(int key, int value) {
            this.key = key;
            this.value = value;
        }
    }

    public OpenHashTable() {
        this(16);
    }

    public OpenHashTable(int capacity) {
        this(capacity, 0.75);
    }

    public OpenHashTable(int capacity, double factory) {
        if (capacity <= 0) {
            capacity = 16;
        }
        table = new Node[capacity];
        size = 0;
        this.factory = factory;
    }

    @Override
    public void put(int key, int value) {
        boolean succeed = false;
        int length = table.length;
        int index = hash(key);
        for (int i = 0; i < length; i++) {
            Node node = table[index];
            // 插入节点
            if (node == null || node.deleted) {
                node = new Node(key, value);
                table[index] = node;
                size++;
                succeed = true;
                break;
            }
            // 更新节点
            if (node.key == key) {
                node.value = value;
                succeed = true;
                break;
            }
            index = conflict(index);
        }

        // 验证添加结果
        if (!succeed) {
            // 没有添加成功，扩容后再尝试
            resize();
            put(key, value);
        } else {
            // 添加成功后，检查是否需要扩容
            checkResize();
        }
    }

    @Override
    public int remove(int key) {
        int index = findIndex(key);
        if (index < 0) {
            return -1;
        }

        // 只标记删除，不清除节点
        Node node = table[index];
        node.deleted = true;
        size--;
        return node.value;
    }

    @Override
    public int get(int key) {
        int index = findIndex(key);
        if (index < 0) {
            return -1;
        }
        return table[index].value;
    }

    @Override
    public boolean contains(int key) {
        return findIndex(key) != -1;
    }

    @Override
    public int size() {
        return size;
    }

    /**
     * 查找指定key的索引位置
     *
     * @param key 关键字
     * @return 对应的索引/-1表示不存在
     */
    protected int findIndex(int key) {
        int length = table.length;
        int index = hash(key);
        for (int i = 0; i < length; i++) {
            Node node = table[index];
            if (node == null) {
                return -1;
            }
            // key 值存在，且没有删除标记
            if (node.key == key && !node.deleted) {
                return index;
            }
            index = conflict(index);
        }
        return -1;
    }

    /**
     * 散列函数
     * <p>
     * 关键字低 16 位和高 16 位进行异或运算，然后取余
     *
     * @param key 关键字
     * @return 散列地址
     */
    protected int hash(int key) {
        return ((key ^ (key >>> 16)) & 0xFFFF) % table.length;
    }

    /**
     * 散列冲突的下一个序列值
     * <p>
     * 线性探测法
     *
     * @param index 索引
     * @return 冲突的下一个索引
     */
    protected int conflict(int index) {
        return (index + 1) % table.length;
    }

    /**
     * 检查并进行数组扩容
     */
    protected void checkResize() {
        // 大于负载因子时，进行扩容
        if (1.0 * size / table.length > factory) {
            resize();
        }
    }

    /**
     * 动态扩容
     */
    protected synchronized void resize() {
        // 数组进行 2 倍扩容
        Node[] oldTable =  table;
        int newLength = oldTable.length << 1;
        table = new Node[newLength];

        // 重新映射数组元素
        for (Node node : oldTable) {
            if (node.deleted) {
                continue;
            }
            int index = hash(node.key);
            while (table[index] != null) {
                index = conflict(index);
            }
            table[index] = node;
        }
    }

}
```

### 散列表-拉链法

```java
/**
 * 散列表-拉链法
 * <p>
 * 散列函数：折叠法 + 取余
 * <p>
 * 散列冲突：双向拉链法
 */
public class LinkedHashTable implements HashTable {

    /**
     * 散列表数组
     */
    Node[] table;
    /**
     * 元素数量
     */
    int size;
    /**
     * 负载因子
     */
    double factory;

    static class Node {
        final int key;
        int value;

        Node prev;
        Node next;

        public Node(int key, int value) {
            this.key = key;
            this.value = value;
        }
    }

    public LinkedHashTable() {
        this(16);
    }

    public LinkedHashTable(int capacity) {
        this(capacity, 0.75);
    }

    public LinkedHashTable(int capacity, double factory) {
        if (capacity <= 0) {
            capacity = 16;
        }
        table = new Node[capacity];
        size = 0;
        this.factory = factory;
    }

    @Override
    public void put(int key, int value) {
        int index = hash(key);
        Node head = table[index];
        if (head == null) {
            // 数组链表头节点
            table[index] = new Node(key, value);
        } else {
            // 查找 key 值是否存在
            Node node = head, prev = null;
            while (node != null && node.key != key) {
                prev = node;
                node = node.next;
            }

            // 更新节点
            if (node != null) {
                node.value = value;
                return;
            }

            // 追加节点
            node = new Node(key, value);
            prev.next = node;
            node.prev = prev;
        }
        size++;

        // 检查是否需要扩容
        checkResize();
    }

    @Override
    public int remove(int key) {
        int index = hash(key);
        Node head = table[index];
        if (head == null) {
            return -1;
        }

        // 查找 key 的节点
        Node node = head;
        while (node != null && node.key != key) {
            node = node.next;
        }
        if (node == null) {
            return -1;
        }

        // 移除链表节点
        if (node.prev == null) {
            // 数组链表头节点
            table[index] = node.next;
            node.next = null;
        } else if (node.next == null) {
            // 链表尾节点
            node.prev.next = null;
            node.prev = null;
        } else {
            // 链表中间节点
            node.next.prev = node.prev;
            node.prev.next = node.next;
            node.prev = node.next = null;
        }

        size--;
        return node.value;
    }

    @Override
    public int get(int key) {
        Node node = find(key);
        if (node == null) {
            return -1;
        }
        return node.value;
    }

    @Override
    public boolean contains(int key) {
        return find(key) != null;
    }

    @Override
    public int size() {
        return size;
    }

    /**
     * 查找指定key的索引位置
     *
     * @param key 关键字
     * @return 对应的索引/-1表示不存在
     */
    protected Node find(int key) {
        int index = hash(key);
        Node node = table[index];
        if (node == null) {
            return null;
        }
        while (node != null && node.key != key) {
            node = node.next;
        }
        return node;
    }

    /**
     * 散列函数
     * <p>
     * 关键字低 16 位和高 16 位进行异或运算，然后取余
     *
     * @param key 关键字
     * @return 散列地址
     */
    protected int hash(int key) {
        return ((key ^ (key >>> 16)) & 0xFFFF) % table.length;
    }

    /**
     * 检查并进行数组扩容
     */
    protected void checkResize() {
        // 大于负载因子时，进行扩容
        if (1.0 * size / table.length > factory) {
            resize();
        }
    }

    /**
     * 动态扩容
     */
    protected synchronized void resize() {
        // 数组进行 2 倍扩容
        Node[] oldTable = table;
        int newLength = oldTable.length << 1;
        table = new Node[newLength];

        // 重新映射数组元素
        for (Node head : oldTable) {
            for (Node node = head; node != null; ) {
                Node next = node.next;
                // 从旧链表断开
                node.prev = node.next = null;
                // 连接到新链表
                int index = hash(node.key);
                if (table[index] == null) {
                    // 链表头节点
                    table[index] = node;
                } else {
                    // 添加到链表尾部
                    Node tail = table[index];
                    while (tail.next != null) {
                        tail = tail.next;
                    }
                    tail.next = node;
                    node.prev = tail;
                }
                node = next;
            }
        }
    }

}
```
