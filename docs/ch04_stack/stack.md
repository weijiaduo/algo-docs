# 栈

## 一、结构特性

基本定义：

- 栈是一种遵循先入后出的逻辑的线性数据结构

主要特点：

- 数据是先入后出，后入先出
- 只允许在栈的一端操作数据

栈的两端分别称为“栈顶”和“栈底”：

<!--more-->

- 栈顶：最后入栈的元素
- 栈底：最先入栈的元素

```
|     |
|     |
|  3  |    <-- 栈顶
|  2  |
|__1__|    <-- 栈底
```

“栈顶”位置随元素数量变化，“栈底”位置一直保持不变。

栈只允许操作“栈顶”，入栈和出栈都是在栈顶操作。


## 二、常用操作

栈的常用操作有 3 种：

- 访问：访问栈顶的数据
- 入栈：添加数据到栈顶
- 出栈：从栈顶移除数据

注意栈只能操作栈底，这是唯一的开口。

### 2.1 访问操作

栈只能访问栈底的元素：

```
访问栈顶3

|     |
|     |
|  3  |    <-- 栈顶
|  2  |
|__1__|    <-- 栈底
```

访问栈顶时间复杂度：O(1)。

### 2.2 入栈操作

入栈类似于堆叠盘子，从栈底开始放，从下往上：

```
 先放1         再放2          再放3

   1             2             3
   |             |             |
   v             v             v

|     |       |     |       |     |
|     |       |     |       |     |
|     |  ==>  |     |  ==>  |  3  |
|     |       |  2  |       |  2  |
|__1__|       |__1__|       |__1__|
```

入栈时间复杂度：O(1)。

### 2.3 出栈操作

出栈类似于取盘子，从栈顶开始取，从上往下：

```
 先出3         再出2          再出1

   3             2             1
   ^             ^             ^
   |             |             |

|     |       |     |       |     |
|     |       |     |       |     |
|  3  |  ==>  |     |  ==>  |     |
|  2  |       |  2  |       |     |
|__1__|       |__1__|       |__1__|
```

出栈时间复杂度：O(1)。


## 三、实现方式

根据栈的底层存储形式，实现方式可以分为 2 种：

- 顺序栈：基于数组实现
- 链式栈：基于链表实现

### 3.1 顺序栈

基于数组实现栈时，一般将数组的尾部作为栈顶：

```
 栈底        栈顶
  |           |
  v           v
 ___ ___ ___ ___ ___ ___ ___
| 1 | 2 | 3 | 4 |   |   |   |   <-- 剩余空间
```

入栈和出栈都是在数组尾部进行操作，比如入栈：

```
 栈底            栈顶
  |               |
  v               v
 ___ ___ ___ ___ ___ ___ ___
| 1 | 2 | 3 | 4 | 5 |   |   |   <-- 剩余空间

                  ^
                  |
                 入栈
```

存在的缺点：

- 数组尾部会有空间浪费
- 数组大小固定，栈空间会受到限制

好处就在于：

- 效率高，只需要改变数组索引
- 有很好的局部缓存性

另外，顺序栈还有一种实现，那就是动态扩容顺序栈，它就不会限制大小了。

### 3.2 链式栈

基于链表实现栈时，一般是将表头作为栈顶：

```  
   栈顶                            栈底
    |                               |
    v                               v
 ___ ______      ___ ______      ___ ______
| 3 | next | -> | 2 | next | -> | 1 | next | -> NULL
```

入栈和出栈都是在链表头部进行操作，比如入栈：

```  
   栈顶                                            栈底
    |                                               |
    v                                               v
 ___ ______      ___ ______      ___ ______      ___ ______
| 4 | next | -> | 3 | next | -> | 2 | next | -> | 1 | next | -> NULL

    ^
    |
   入栈
```

这种在表头操作节点的方式称为“头插法”。

存在的缺点：

- 需要额外的空间存储指针
- 操作效率低，需修改指针
- 缓存局部性不好

好处就在于：

- 大小不受限制，可以无限扩容

其实顺序栈和链式栈各有优缺点，根据实际情况选择即可。

## 四、复杂度

- 入栈：时间复杂度 O(1)，空间复杂度 O(1)
- 出栈：时间复杂度 O(1)，空间复杂度 O(1)


## 五、典型应用

- 函数调用栈
- 浏览器的前进后退
- 软件中的撤销和反撤销
- 表达式求值（双栈：操作数栈和操作符栈）
- 括号匹配


## 参考

> https://www.hello-algo.com/chapter_stack_and_queue/stack/
>
> https://time.geekbang.org/column/article/41222
