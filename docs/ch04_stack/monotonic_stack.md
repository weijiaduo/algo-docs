# 单调栈

## 一、什么是单调栈？

单调栈（Monotonic Stack）是一种特殊的栈，在栈的「先进后出」基础上，要求「从栈顶到栈底的元素是单调的（递增或递减）」。

与普通栈不同，单调栈的元素是按照特定的单调性（递增或递减）排列的：

```
 普通栈        单调栈

|     |       |     |                    
|     |       |     |                     
|  3  |       |  1  |    <-- 栈顶
|  1  |       |  2  |                 
|__2__|       |__3__|    <-- 栈底
```

按照单调栈的元素是按照特定的单调性，可分为两种类型：

<!--more-->

- 单调递增栈：从栈顶到栈底的元素是单调递增的
- 单调递减栈：从栈顶到栈底的元素是单调递减的

注意，单调栈定义的顺序是从「栈顶」到「栈底」。

### 1.1 单调递增栈

栈中的元素按照递增（非严格递增）的顺序排列，即从栈底到栈顶的元素值逐渐增加。

```
|     |
|     |
|  1  |    <-- 栈顶
|  2  |
|__3__|    <-- 栈底
```

### 1.2 单调递减栈

栈中的元素按照递减（非严格递减）的顺序排列，即从栈底到栈顶的元素值逐渐减小

```
|     |
|     |
|  3  |    <-- 栈顶
|  2  |
|__1__|    <-- 栈底
```

## 二、为什么用单调栈？

单调栈的主要优势在于可以高效地解决一类特定问题，这些问题涉及到维护元素的单调性（递增或递减）。使用单调栈的原因包括：

- 提高算法的性能：单调栈通常能够以线性时间复杂度解决问题，而不需要嵌套循环。
- 减少空间复杂度：与一些其他数据结构相比，单调栈通常需要较少的额外空间，因为它只存储有限数量的元素。
- 解决特定问题：单调栈特别适用于下一个更大或更小元素的问题，这些问题很难用传统的方法解决，但单调栈可以在常数时间内解决它们。

单调栈可用于高效地解决特定问题，尤其是涉及到维护元素的单调性和查找下一个更大或更小元素的问题。

## 三、如何实现单调栈？

单调栈的实现细节可能更多地关注在两个方面：单调性和遍历方向。

### 3.1 如何选择单调性？

单调栈最关键的就是选择单调性，而单调性的选择（递增或递减）通常取决于问题的性质：

- 递增栈：上/下一个更大元素问题
- 递减栈：上/下一个更小元素问题

问题定义了需要找到上/下一个更大或更小元素，根据这个性质，就可以选择使用递增栈或递减栈。

#### 1) 递增栈

求上/下一个更大的元素，则采用单调递增栈的方式，栈中元素按递增顺序排列。

单调递增栈的出入栈过程如下：

- 持续比较栈顶和新元素，如果新元素比栈顶元素更大，就将栈顶元素弹出
- 当栈为空，或者新元素比栈顶元素更小时，就将新元素压入栈中

其伪代码类似这样：

```java
while (!stack.empty() && x > stack.top()) {
    stack.pop();
}
stack.push(x);
```

比如，求 `[73, 74, 75, 71, 69, 72, 76, 73]` 中每个元素的右侧第一个更大元素。

一开始是空栈，直接将 `73` 压入栈中：

```
 插入73
   |
|  v   |
|      |
|      |
|      |
|__73__|    <-- 栈顶/栈底

右侧第一个更大元素：[--, --, --, --, --, --, --, --]
```

`74 > 73`，所以需先将栈顶弹出，再将 `74` 压入栈中：

```
 弹出73         插入74
   ^              |
|  |   |      |   v  |
|      |      |      |
|      |      |      |
|      |      |      |
|______|      |__74__|    <-- 栈顶/栈底

右侧第一个更大元素：[74, --, --, --, --, --, --, --]
```

`75 > 74`，所以需先将栈顶弹出，再将 `75` 压入栈中：

```
 弹出74         插入75
   ^              |
|  |   |      |   v  |
|      |      |      |
|      |      |      |
|      |      |      |
|______|      |__75__|    <-- 栈顶/栈底

右侧第一个更大元素：[74, 75, --, --, --, --, --, --]
```

`71 < 75`，所以将 `71` 压入栈中：

```
 插入71
   |
|  v   |
|      |
|      |
|  71  |    <-- 栈顶
|__75__|    <-- 栈底

右侧第一个更大元素：[74, 75, --, --, --, --, --, --]
```

`69 < 71`，所以将 `69` 压入栈中：

```
 插入69
   |
|  v   |
|      |
|  69  |    <-- 栈顶
|  71  |
|__75__|    <-- 栈底

右侧第一个更大元素：[74, 75, --, --, --, --, --, --]
```

`72 > 69` 且 `71 >71`，需先将 `69`、`71` 弹出，才将 `72` 压入栈中：

```
 弹出69         弹出71         插入72
   ^              ^             |
|  |   |      |   |  |      |   v  |
|      |      |      |      |      |
|      |      |      |      |      |
|  71  |      |      |      |  72  |    <-- 栈顶
|__75__|      |__75__|      |__75__|    <-- 栈底

右侧第一个更大元素：[74, 75, --, 72, 72, --, --, --]
```

> 注意：此处右侧第一个更大元素数组的更新，75 的位置还未更新更大值。

`76 > 72` 且 `76 > 75`，需先将 `71`、`75` 弹出，再将 `76` 压入栈中：

```
 弹出72         弹出75         插入76
   ^              ^             |
|  |   |      |   |  |      |   v  |
|      |      |      |      |      |
|      |      |      |      |      |
|      |      |      |      |      |
|__75__|      |______|      |__76__|    <-- 栈顶栈底

右侧第一个更大元素：[74, 75, 76, 72, 72, 76, --, --]
```

`73 < 76`，所以将 `73` 压入栈中：

```
 插入73
   |
|  v   |
|      |
|      |
|  73  |    <-- 栈顶
|__76__|    <-- 栈底

右侧第一个更大元素：[74, 75, 76, 72, 72, 76, -, -]
```

至此，就得到了每个元素的右侧第一个更大元素。

```
[73, 74, 75, 71, 69, 72, 76, 73]
[74, 75, 76, 72, 72, 76, --, --]
```

这里使用元素值来举例的，正常来说，单调栈里面一般都存储的是元素索引，因为方便定位元素。

#### 2) 递减栈

求上/下一个更小的元素，则采用单调递减栈的方式，栈中元素按递减顺序排列。

单调递减栈的出入栈过程如下：

- 持续比较栈顶和新元素，如果新元素比栈顶元素更小，就将栈顶元素弹出
- 当栈为空，或者新元素比栈顶元素更大时，就将新元素压入栈中

其伪代码类似这样：

```java
while (!stack.empty() && x < stack.top()) {
    stack.pop();
}
stack.push(x);
```

实际的操作和递增栈类似，只是比较的方向相反。

### 3.2 如何选择遍历方向？

从左往右的正向遍历和从右往左的反向遍历并不会影响单调栈的单调性选择。

遍历顺序只影响到了元素的顺序，但并不改变问题本身的要求。因此，无论遍历方向如何，问题要求的性质仍然相同。

比如，求元素的右侧第一个更大元素，分别采用正向遍历和反向遍历的方式实现：

```java
// 正向遍历
for (int i = 0; i < n; i++) {
    // 注意这里的符号是 >
    while (!stack.empty() && x > stack.top()) {
        y = stack.pop();
        ans[y] = x;
    }
    stack.push(x);
}
```
```java
// 反向遍历
for (int i = n - 1; i >= 0; i--) {
    // 注意这里的符号是 >=
    while (!stack.empty() && x >= stack.top()) {
        stack.pop();
    }
    ans[x] = stack.empty() ? -1 : stack.top();
    stack.push(x);
}
```

从代码可以看出，正向遍历和反向遍历在单调性上是一致的，都是递增栈，而区别就在于：

- 正向遍历和反向遍历的递增栈，一个是严格递增，一个是非严格递增

同理，对于其他的问题，比如求下一个更小元素，结论也是一样的，遍历方向不会影响单调性的选择，只会影响单调的严格性。

所以，首先要理解问题的性质，然后选择适当的栈类型，而不必过于担心遍历的方向。

### 小结

- 单调栈的单调性（递增或递减）取决于问题的性质：
    - 递增栈：上/下一个更大元素问题。
    - 递减栈：上/下一个更小元素问题。
- 遍历方向（正向或反向）不会影响单调栈的单调性选择，只会影响单调的严格性：
    - 正向遍历和反向遍历的单调栈，其中一个是严格的，一个是非严格的

## 四、单调栈的典型应用

- 计算柱状图中的最大矩形面积。
- 寻找每个元素的右侧第一个更大元素，如在温度预测问题中找到下一个更热的日期。
- 寻找每个元素的左侧第一个更小元素，如在计算每个元素的右侧最大矩形问题中。
- 滑动窗口最大值问题。
- 语法分析器，单调栈可以用于编写语法分析器，解析和评估数学表达式等。

## 附录

### 下一个更大值

```java
/**
 * 单调递增栈：下一个更大元素的索引
 *
 * @author weijiaduo
 * @since 2023/10/12
 */
public class NextGreater {

    /**
     * 正向遍历，不存在更大值时返回 -1
     *
     * @param arr 数组
     * @return 下一个更大元素索引数组
     */
    public int[] forward(int[] arr) {
        int n = arr.length;
        int[] ans = new int[n];
        Arrays.fill(ans, -1);
        Deque<Integer> incStack = new LinkedList<>();
        for (int i = 0; i < n; i++) {
            // 注意这里是 >
            while (!incStack.isEmpty() && arr[i] > arr[incStack.peek()]) {
                int idx = incStack.pop();
                ans[idx] = i;
            }
            incStack.push(i);
        }
        return ans;
    }

    /**
     * 逆向遍历，不存在更大值时返回 -1
     *
     * @param arr 数组
     * @return 下一个更大元素索引数组
     */
    public int[] backward(int[] arr) {
        int n = arr.length;
        int[] ans = new int[n];
        Arrays.fill(ans, -1);
        Deque<Integer> incStack = new LinkedList<>();
        for (int i = n - 1; i >= 0; i--) {
            // 注意这里是 >=
            while (!incStack.isEmpty() && arr[i] >= arr[incStack.peek()]) {
                incStack.pop();
            }
            ans[i] = incStack.isEmpty() ? -1 : incStack.peek();
            incStack.push(i);
        }
        return ans;
    }

}
```

### 上一个更大值

```java
/**
 * 单调递增栈：上一个更大元素的索引
 *
 * @author weijiaduo
 * @since 2023/10/12
 */
public class PrevGreater {

    /**
     * 正向遍历，不存在更大值时返回 -1
     *
     * @param arr 数组
     * @return 上一个更大元素索引数组
     */
    public int[] forward(int[] arr) {
        int n = arr.length;
        int[] ans = new int[n];
        Arrays.fill(ans, -1);
        Deque<Integer> incStack = new LinkedList<>();
        for (int i = 0; i < n; i++) {
            // 注意这里是 >=
            while (!incStack.isEmpty() && arr[i] >= arr[incStack.peek()]) {
                incStack.pop();
            }
            ans[i] = incStack.isEmpty() ? -1 : incStack.peek();
            incStack.push(i);
        }
        return ans;
    }

    /**
     * 逆向遍历，不存在更大值时返回 -1
     *
     * @param arr 数组
     * @return 上一个更大元素索引数组
     */
    public int[] backward(int[] arr) {
        int n = arr.length;
        int[] ans = new int[n];
        Arrays.fill(ans, -1);
        Deque<Integer> incStack = new LinkedList<>();
        for (int i = n - 1; i >= 0; i--) {
            // 注意这里是 >
            while (!incStack.isEmpty() && arr[i] > arr[incStack.peek()]) {
                int idx = incStack.pop();
                ans[idx] = i;
            }
            incStack.push(i);
        }
        return ans;
    }

}
```

### 下一个更小值

```java
/**
 * 单调递减栈：下一个更小元素的索引
 *
 * @author weijiaduo
 * @since 2023/10/12
 */
public class NextSmaller {

    /**
     * 正向遍历，不存在更小值时返回 -1
     *
     * @param arr 数组
     * @return 下一个更小元素索引数组
     */
    public int[] forward(int[] arr) {
        int n = arr.length;
        int[] ans = new int[n];
        Arrays.fill(ans, -1);
        Deque<Integer> decStack = new LinkedList<>();
        for (int i = 0; i < n; i++) {
            // 注意这里是 <
            while (!decStack.isEmpty() && arr[i] < arr[decStack.peek()]) {
                int idx = decStack.pop();
                ans[idx] = i;
            }
            decStack.push(i);
        }
        return ans;
    }

    /**
     * 逆向遍历，不存在更小值时返回 -1
     *
     * @param arr 数组
     * @return 下一个更小元素索引数组
     */
    public int[] backward(int[] arr) {
        int n = arr.length;
        int[] ans = new int[n];
        Arrays.fill(ans, -1);
        Deque<Integer> decStack = new LinkedList<>();
        for (int i = n - 1; i >= 0; i--) {
            // 注意这里是 <=
            while (!decStack.isEmpty() && arr[i] <= arr[decStack.peek()]) {
                decStack.pop();
            }
            ans[i] = decStack.isEmpty() ? -1 : decStack.peek();
            decStack.push(i);
        }
        return ans;
    }

}
```

### 上一个更小值

```java
/**
 * 单调递减栈：上一个更小元素的索引
 * 
 * @author weijiaduo
 * @since 2023/10/12
 */
public class PrevSmaller {

    /**
     * 正向遍历，不存在更小值时返回 -1
     *
     * @param arr 数组
     * @return 上一个更小元素索引数组
     */
    public int[] forward(int[] arr) {
        int n = arr.length;
        int[] ans = new int[n];
        Arrays.fill(ans, -1);
        Deque<Integer> decStack = new LinkedList<>();
        for (int i = 0; i < n; i++) {
            // 注意这里是 <=
            while (!decStack.isEmpty() && arr[i] <= arr[decStack.peek()]) {
                decStack.pop();
            }
            ans[i] = decStack.isEmpty() ? -1 : decStack.peek();
            decStack.push(i);
        }
        return ans;
    }

    /**
     * 逆向遍历，不存在更小值时返回 -1
     *
     * @param arr 数组
     * @return 上一个更小元素索引数组
     */
    public int[] backward(int[] arr) {
        int n = arr.length;
        int[] ans = new int[n];
        Arrays.fill(ans, -1);
        Deque<Integer> decStack = new LinkedList<>();
        for (int i = n - 1; i >= 0; i--) {
            // 注意这里是 <
            while (!decStack.isEmpty() && arr[i] < arr[decStack.peek()]) {
                int idx = decStack.pop();
                ans[idx] = i;
            }
            decStack.push(i);
        }
        return ans;
    }
    
}
```
