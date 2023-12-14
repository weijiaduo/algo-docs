---
title: 红黑树
date: 2023-02-09 23:39:30
categories: [数据结构与算法, 数据结构]
tags: [数据结构, 算法, 红黑树]
thumbnail: /img/structure.jpg
top: true
---

# 红黑树

## 一、什么是红黑树？

红黑树是一棵只有红色节点和黑色节点的平衡二叉搜索树。

其结构类似这样：

```
      b4
   /      \  
  r2      r6
 /  \    /  \
b1  b3  b5  b7
              \
              r9
```

其中，`b` 表示黑色，`r` 表示红色。

### 1.1 红黑树的特性

在《算法导论》里，红黑树是这样定义的：

<!--more-->

1. 每个结点或是红色的，或是黑色的
2. 根节点是黑色的
3. 每个叶结点（NIL）是黑色的
4. 如果一个节点是红色的，则它的两个儿子都是黑色的
5. 对于每个结点，从该结点到其子孙结点的所有路径上包含相同数目的黑色结点

前 4 点好理解，第 5 点需要说明一下，它的意思是：

- 从任一节点到其每个叶子的所有路径都包含相同数目的黑色节点

其中，黑节点的数目称为黑高（`black-height`）。

比如上面的红黑树，从 `b4` 节点开始的路径有 4 条：

- `b4 -> r2 -> b1`：2 个黑色节点 `[b4, b1]`
- `b4 -> r2 -> b3`：2 个黑色节点 `[b4, b3]`
- `b4 -> r6 -> b5`：2 个黑色节点 `[b4, b5]`
- `b4 -> r6 -> b7`：2 个黑色节点 `[b4, b7]`

每条路径包含的黑色节点数量是一样的。所以，

- 红黑树并不是“高度”平衡的二叉树，而是“黑高”平衡的二叉树

至于为什么是“黑高”平衡，这和红黑树的起源有关。

## 二、红黑树的起源

### 2.1 与 4-阶 B-树的关系

红黑树本质上是由 4 阶 B-树演化而来的，节点之间的对应关系是：

1）节点有 1 个元素

```
4 阶 B-树节点                红黑树节点

 ___ ___ ___
| 1 |   |   |      =>           b1
```

2）节点有 2 个元素

```
4 阶 B-树节点                红黑树节点

 ___ ___ ___                    b1                b2
| 1 | 2 |   |      =>            \       或       / 
                                 r2              r1
```

3）节点有 3 个元素

```
4 阶 B-树节点                红黑树节点

 ___ ___ ___                    b2
| 1 | 2 | 3 |      =>          /  \
                              r1  r3
```

需要说明的是：

- 1 个 4 阶 B-树节点对应的红黑树结构，只会有 1 个黑色节点
- 4 阶 B-树节点有 3 个元素时，对应的红黑树节点只有 1 种情况
- 合法的 4 阶 B-树 3 元素节点对应的红色节点只能在左右子节点中

所以下面这些情况都是不合法的：

```
    b3       b3       b1        b1            
    /        /         \         \
  r2        r1          r3        r2 
  /          \          /          \
r1            r2       r2           r3
```

简单点说，就是：

- 先有黑色节点，才有红色节点
- 红色节点不能单独存在，总是与黑色节点绑定的
- 红色节点的父节点只能是黑色
- 2 个红色节点不能够相邻

这些全都是红黑树的特性，总之就是：

- 将红色节点和它的黑色父节点，看成是 1 个节点就行了
- 1 个节点最多带有 3 个元素，最多时只能是“1黑色 + 2红色”

上面说红黑树是“黑高”平衡的，是因为 1 个 4 阶 B-树节点对应到红黑树上只会有 1 个黑色节点。

而 B-树是高度平衡的，所以对应到红黑树时，就变成了“黑高”平衡。

### 2.2 为什么不直接用 4 阶 B-树？

实际上，相对于 4 阶 B-树，红黑树会有一些优势：

- 空间利用率
    - B-树有空间浪费，空间利用率不高
    - 红黑树的空间利用率总是 100%
- 二分查找
    - B-树是多叉树，采用的是多路搜索，没有二分查找那么方便
    - 红黑树本身就是二叉搜索树

比如，上面的 1 元素节点：

```
4 阶 B-树节点                红黑树节点

 ___ ___ ___
| 1 |   |   |      =>           b1
```

对于 B-树而言，空间利用率只有 33%，而红黑树的空间利用率就是 100%。

除了这些优点，还有一点：

- B-树是多叉树，代码实现比红黑树要复杂得多
- 红黑树本身是二叉树，代码再复杂也不会比多叉树复杂

综合各种情况，还是红黑树会比 4阶 B-树更优一些。

### 2.3 为什么是红色和黑色？

为什么是红黑树，而不是黑白树？黄绿树？

目前发现有 2 种说法：

- 红色和黑色是激光打印机最好找到的颜色
- 红色和黑色是最容易获得的笔色

不管是哪种说法都差不多，大概就是：

- 因为平时常用红色和黑色来作图，所以才称之为红黑树

红黑色的来源确实贴近生活，符合实际。

## 三、为什么用红黑树？

红黑树的优略势：

- 相比于 BST 树，红黑树结构更加平衡，稳定性更好
- 相比于 AVL 树，红黑树在插入和删除过程中的旋转更少
- 相比于 B-树，红黑树结构更简单，空间利用率更高

红黑树的适用场景：

- 频繁的插入、删除，采用红黑树
- 频繁的搜索，采用 AVL 树


## 四、如何实现红黑树？

声明，下面默认是将红黑树节点看成是 4 阶 B-树节点：

- 将红色节点和它的黑色父节点，看成是 1 个节点
- 1 个节点最多带有 3 个元素，最多时只能是“1黑色 + 2红色”

这样是为了简化理解，从 B-树的角度去理解红黑树为什么要这么调整。

### 4.1 检索

红黑树本身就是 BST 树，所以检索也是采用的二分查找。

伪代码如下：

```java
private RBTNode get(RBTNode h, int val) {
    // 没有找到
    if (h == null) {
        return null;
    }
    // 找到对应的节点
    if (h.val == val) {
        return h;
    }
    if (h.val > val) {
        return get(h.left, val);
    } else {
        return get(h.right, val);
    }
}
```

红黑树的检索时间复杂度是 O(logn)。

### 4.2 插入

红黑树是一棵平衡树，所以插入节点后可能会需要调整，维持平衡。

红黑树插入新节点的原则：

- 新节点总是在叶子节点处插入
- 新节点的颜色总是红色

根据这 2 个原则，插入新节点后可能出现的情况有 6 种：

```
黑色父节点        红色父节点
                               
    b                b            
   / \             /   \  
                  r     r 
                 / \   / \
```

其中，在插入红色节点后，6 种情况里：

- 父节点是黑色的 2 种情况，不影响红黑树的“黑高”平衡，不需要调整
- 只有父节点是红色的 4 种情况，需要调整红黑树的结构，以维持平衡

所以，需要特殊处理的只是父节点是红色的 4 种情况。

#### 4.2.1 `Left-Left-Red`

`Left-Left-Red` 表示的红黑树类似这样（新插入节点是 r1）：

```
    b3          
    /
  r2
  /
r1
```

由于 2 个红色节点相邻了，不符合红黑树的定义，需要调整。

调整方式很简单，只需要做 2 步处理：

```
                  红色节点右旋           父子节点反色
                               
    b3
    /                 b2                    r2
  r2        =>       /  \         =>       /  \
  /                 r1  r3                b1  b3
r1
```

做完处理后，父节点变成了红色，此时还可能会出现连续 2 个红色节点的情况。

- 父节点变红后，可以将父节点 r2 当作新插入的节点，继续往上递归调整

往上递归调整一直到结构平衡，或者根节点为止。

#### 4.2.2 `Left-Right-Red`

`Left-Right-Red` 表示的红黑树类似这样（新插入节点是 r2）：

```
    b3      
    /
  r1
   \
    r2
```

平衡的调整过程如下：

```
                  红色节点左旋 
                
    b3                b3 
    /                 /
  r1        =>      r2
   \                /
    r2            r1
```

旋转后，`Left-Right-Red` 就变成了 `Left-Left-Red`，按照上面的处理即可。

#### 4.2.3 `Right-Right-Red`

`Right-Right-Red` 表示的红黑树类似这样（新插入节点是 r3）：

```
b1
 \
  r2
   \
    r3
```

平衡的调整过程如下：

```
                  红色节点左旋           父子节点反色
                
    b1                                      
     \                 b2                    r2
      r2     =>       /  \         =>       /  \
       \             r1  r3                b1  b3
        r3
```

做完处理后，父节点变成了红色，此时还可能会出现连续 2 个红色节点的情况。

这个和 `Left-Left-Red` 差不多，因为红黑树的对称的二叉树，按照和它一样的方式处理即可。

#### 4.2.4 `Right-Left-Red`

`Left-Right-Red` 表示的红黑树类似这样（新插入节点是 r2）：

```
    b1      
     \
      r3
      /
    r2
```

平衡的调整过程如下：

```
                  红色节点右旋 
                
    b1                b1 
     \                 \
      r3        =>      r2
      /                  \
    r2                    r3
```

旋转后，`Right-Left-Red` 就变成了 `Right-Right-Red`，按照上面的处理即可。

#### 小结

综合上述的 6 种情况，插入的伪代码如下：

```java
 public void insert(int val) {
    // 插入新节点
    RBTNode h = new RBTNode(val);
    root = insertNode(root, h);
    
    // 调整树结构
    balanceInsertion(h);

    // 根节点始终是黑色
    setColor(root, BLACK);
}
```

插入新节点后，调整红黑树结构以维持平衡：

```java
private void balanceInsertion(RBTNode h) {
    // 父节点是黑色时，不用调整
    RBTNode p = parent(h);
    if (!isRed(p)) {
        return;
    }

    // 父节点是红色的 4 种情况
    RBTNode gp = parent(p);
    if (p == gp.left) {
        // Left-Right-Red
        if (h == p.right) {
            rotateLeft(p);
        }
        // Left-Left-Red
        p = rotateRight(gp);
    } else {
        // Right-Left-Red
        if (h == p.left) {
            rotateRight(p);
        }
        // Right-Right-Red
        p = rotateLeft(gp);
    }

    // 父子节点反色
    setColor(p, RED);
    setColor(p.left, BLACK);
    setColor(p.right, BLACK);

    // 父节点递归往上处理
    balanceInsertion(p);
}
```

另外，红黑树根节点始终黑色的，但是插入调整往上递归时，有可能把根节点变成红色了。

所以在最终的插入完成后，一般都要将根节点的颜色设置成黑色。

### 4.3 删除

红黑树的删除大致可以分为几个步骤：

1. 替代删除节点：寻找替代当前节点的节点，如果没有，就直接删除当前节点
2. 删除平衡调整：删除前先调整树结构，保证删除节点后，红黑树的结构依旧是稳定的
3. 真正删除节点

下面详细说明这几个步骤。

#### 4.3.1 替代删除节点

替换删除节点，是 BST 树删除都要做的事情，因为要保证 BST 树的有序性。

红黑树也是 BST 树，所以一开始也是要寻找替代的节点：

```
删除 r1 叶子节点             没有替代

      b2                       b2
     /  \         =>          /  \
    r1  r3                   r1  r3
```

```
删除 b2 内部单子节点         b2 和前继 r1 交换

      b2                       b1
     /            =>          /
    r1                       r2
```

```
删除 b2 内部单子节点         b2 和后继 r3 交换

      b2                       b3
        \         =>             \
        r3                       r2
```

```
删除 b2 内部双子节点       b2 和后继 r3 交换

      b2                       b3
     /  \                     /  \
    b1  b4        =>         b1  b4
        /                        /
       r3                       r2
```

除删除叶子节点外，其余情况都使用前继/后继节点替换。

#### 4.3.2 删除平衡调整

红黑树是由 B-树演化而来的，所以可以参考 B-树的删除平衡调整。

B-树的删除平衡调整可以分为 3 种情况：

1. 当前节点元素充足，可以直接删除，且不需要调整树结构
2. 当前节点元素不足，兄弟节点有多余元素，从兄弟节点借一个元素
3. 当前节点元素不足，兄弟节点也不能借，就将父节点下溢，合并父子节点为新节点

红黑树和 B-树是对应的，红黑树的平衡调整也可以分为这 3 种情况。

##### 4.3.2.1 红色节点直接删除

当被删除的 B-树节点元素充足时，可以直接删除，不用调整：

```
                                                               无需调整树结构

           ___ ___ ___             remove 1                     ___ ___ ___ 
          | 1 | 2 | 3 |               =>                       | 2 | 3 |   | 
```

这种情况对应到红黑树就是：

- 删除节点是 1 个黑色节点 + 1/2 个红色节点
- 删除红黑树的红色节点时，可以直接删除，不用调整树结构
- 但不能是删除黑色节点，因为删除黑色节点相当于删除了所有相关节点，情况不同

删除红色节点的过程就类似这样：

```
删除红色节点                  删除后

    b2       remove r1        b2
   /  \         =>             \
  r1  r3                       r3
```

```
删除红色节点                  删除后

    b2       remove r1        
   /            =>            b2
  r1
```

删除红色节点的情况就是直接删除即可，也不需要调整结构，比较简单。

##### 4.3.2.2 从兄弟借红色节点

当被删除的 B-树节点元素不足时，会先从兄弟节点借一个元素：

```
                                                          左旋，从右兄弟借一个元素

           ___ ___ ___                                          ___ ___ ___ 
          | 2 |   |   |                                        | 3 |   |   | 
                |                                                    |
      -------------------           remove 1                -------------------
      |                 |              =>                   |                 |
 ___ ___ ___       ___ ___ ___                         ___ ___ ___       ___ ___ ___ 
| 1 |   |   |     | 3 | 5 |   |                       | 2 |   |   |     | 5 |   |   | 
```

这种情况对应到红黑树就是：

- 删除节点是黑色节点，而兄弟则是 1 个黑色节点 + 1/2 个红色节点（子节点）
- 删除黑色节点时，如果兄弟附带有红色节点（即红色子节点），就可以从兄弟那边借一个红色节点

这种情况的处理过程就类似这样：

```
  b2      borrow right       b3       remove b1         b3
 /  \         =>            /  \         =>            /  \
b1  b3                     b2  b5                     b2  b5
      \                   /
      r5                 b1
```

```
  b2      rotate right       b2
 /  \         =>            /  \        =>  按照上面的处理
b1  b5                     b1  b3
    /                            \
   r3                            r5
```

先从兄弟借一个红色节点，再删除元素，这样就不会损坏树平衡。不过，

- 当兄弟附带有红色子节点时才可以借，所以兄弟必须是黑色节点才行

如果兄弟是红色节点，需要先旋转一次，将兄弟变成黑色才行，类似这样：

```
     删除1                             6 左旋

      b2                                b6
   /      \        rotate left       /      \
  b1      r6          =>           r2       b7      =>   按照上面的处理
         /  \                     /  \
        b4  b7                   b1  b4
       /  \                         /  \
      r3  r5                       r3  r5 
```

旋转以后，删除节点的兄弟由红色 r6 变成黑色 b4，此时就能从兄弟借元素了。

##### 4.3.2.3 父节点下溢合并

当被删除的 B-树节点元素不足，并且兄弟也没办法借时，父节点就会产生下溢，并和左右子节点合并：

```
                                                        父节点下溢，合并父子节点

           ___ ___ ___
          | 2 |   |   |
                |                                                     
      -------------------           remove 1                  ___ ___ ___ 
      |                 |              =>                    | 2 | 3 |   |
 ___ ___ ___       ___ ___ ___ 
| 1 |   |   |     | 3 |   |   |
```

这种情况对应到红黑树就是：

- 删除节点是黑色节点，并且兄弟也只是 1 个黑色节点
- 删除黑色节点时，如果兄弟没附带有红色节点（即红色子节点），就需要父节点下溢，和子节点合并

这种情况的处理过程就类似这样：

```
                              父节点下溢
             
      b2        remove 1         b2
     /  \          =>              \
    b1  b3                         r3
```

不过由于父节点下溢了，就可能会产生新的不平衡，比如这样：

```
    要删除1                      2变黑，3变红                       删除1

      b4                            b4                             b4
   /      \       merge b2       /      \      remove r1        /      \
  b2      b6         =>         b2      r6         =>          b2      b6
 /  \    /  \                  /  \    /  \                      \    /  \
b1  b3  b5  b7                r1  r3  b5  b7                     r3  b5  b7
```

假如这样删除 1 节点，那么最后左子树的黑高是 2，而右子树的黑高是 3，这其实是不平衡的结构。

和 B-树的一样，对于这种情况，就需要递归调整不平衡结构，将父节点当作删除节点再往上递归调整：

```
    要删除1                     2变黑，3变红                   4变黑，6变红                        删除1

      b4                            b4                             b4                             b4
   /      \       merge b2       /      \       merge b4        /      \      remove r1        /      \
  b2      b6         =>         b2      r6         =>          b2      r6         =>          b2      r6
 /  \    /  \                  /  \    /  \                   /  \    /  \                      \    /  \
b1  b3  b5  b7                r1  r3  b5  b7                 r1  r3  b5  b7                     r3  b5  b7
```

按照这种方式一直往上递归，直到结构平衡或者抵达根节点为止。

- 为了避免产生级联下溢，应该优先选择红色父节点下溢

所以，如果父节点有红色节点（即兄弟是红色），先将红色节点转到删除节点这边，然后再操作：

```
    要删除2                         6左旋

      b4                            b6
   /      \      rotate left     /      \
  b2      r6         =>         r4      b7         =>     按照上面的处理
         /  \                  /  \
        b5  b7                b2  b5
```

旋转过后，删除节点的父节点由黑色 b4 变成了红色 r4，这样就不会产生级联下溢了。

#### 4.3.3 真正删除节点

删除平衡调整完成后，就开始真正地删除节点了。

由于第 1 步使用后继节点替代了删除节点，所以最后被删除节点的结构是很简单的：

- 被删除节点至多只有 1 个子节点
- 直接删除节点，然后返回它的非空子节点即可

最后的删除节点过程类似这样：

```
   删除 b2 节点           返回非空子节点

      b2       remove b2         
     /            =>          r1
    r1                       
```

```
   删除 b2 节点           返回非空子节点

      b2       remove b2          
        \         =>          r3
        r3
```

```
   删除 b2 节点           返回非空子节点

                remove b2
      b2          =>          nil
```

最后的删除节点最简单，难的是前面的删除平衡调整。

#### 小结

提取“从兄弟借红节点”和“父节点下溢合并”的公共行为：

- 删除节点的兄弟是红色，就先将红色节点转到删除节点这一边

然后再根据上面说明的几种情况，删除的伪代码类似这样：

```java
public void remove(int val) {
    RBTNode h = get(root, val);
    if (h == null) {
        return;
    }

    // 替换节点（前继/后继）
    h = swapReplacer(h);

    // 先调整树结构
    balanceDeletion(h);

    // 真正删除节点
    removeNode(h);

    // 根节点始终是黑色
    setColor(root, BLACK);
}
```

删除前调整树平衡：

```java
private void balanceDeletion(RBTNode h) {
    // 1. 删除红色节点，不需要调整
    RBTNode p = parent(h);
    if (p == null || isRed(h)) {
        return;
    }

    // 2. 兄弟是红色，先转成黑色
    RBTNode s = sibling(h);
    if (isRed(s)) {
        rotateOpposite(h);
        s = sibling(h);
    }

    // 3. 看兄弟是否有红色节点可以借一个
    if (hasRedChild(s)) {
        borrowSiblingRed(h);
        return;
    }

    // 4. 兄弟没有红色，父节点下溢合并
    boolean balanced = underflow(h);

    // 视情况往上递归调整
    if (!balanced) {
        balanceDeletion(p);
    }
}
```

和插入一样，删除平衡调整后，最后也要将根节点重新设为黑色。

## 总结

红黑树的定义：

- 红黑树是一棵只有红色节点和黑色节点的平衡二叉搜索树
    1. 每个结点或是红色的，或是黑色的
    2. 根节点是黑色的
    3. 每个叶结点（NIL）是黑色的
    4. 如果一个节点是红色的，则它的两个儿子都是黑色的
    5. 对于每个结点，从该结点到其子孙结点的所有路径上包含相同数目的黑色结点
- 红黑树并不是“高度”平衡的二叉树，而是“黑高”平衡的二叉树

红黑树的来源：

- 红黑树实际上是由 4 阶 B-树演化而来的，是互相对应的
- 因为平时常用红色和黑色来作图，所以才称之为红黑树

红黑树的优略势：

- 相比于 BST 树，红黑树结构更加平衡，稳定性更好
- 相比于 AVL 树，红黑树在插入和删除过程中的旋转更少
- 相比于 B-树，红黑树结构更简单，空间利用率更高

红黑树的适用场景：

- 频繁的插入、删除，采用红黑树
- 频繁的搜索，采用 AVL 树

红黑树的检索：

- 采用的二分查找

红黑树的插入：

- 新节点总是在叶子节点处插入
- 新节点的颜色总是红色
- 插入新节点后可能出现的情况有 6 种
    - 父节点是黑色
        - 新插入节点是左子节点
        - 新插入节点是右子节点
    - 父节点是红色
        - 父节点是左子节点
            - 新插入节点是左子节点
            - 新插入节点是右子节点
        - 父节点是右子节点
            - 新插入节点是左子节点
            - 新插入节点是右子节点
- 父节点是黑色的 2 种情况，不影响红黑树的“黑高”平衡，所以不需要调整
- 只有父节点是红色的 4 种情况需要调整红黑树结构，以维持平衡

红黑树的删除：

- 通用的删除步骤
    1. 替代删除节点：寻找替代当前节点的后继节点，如果没有，就直接删除当前节点
    2. 删除平衡调整：删除前先调整树结构，保证删除节点后，红黑树的结构依旧是稳定的
    3. 真正删除节点
- 删除平衡调整
    - 红色节点直接删除
    - 从兄弟借红色节点
    - 父节点下溢合并


## 参考

> 《算法》（第4版）
> 
> 《数据结构与算法分析（第三版）》
>
> https://blog.csdn.net/zzy893037505/article/details/115177354
>
> http://events.jianshu.io/p/48331a5a11f4
> 
> https://wenku.baidu.com/view/225ca967cb50ad02de80d4d8d15abe23492f037b.html?_wkts_=1675956297191
>
> https://en.wikipedia.org/wiki/Red%E2%80%93black_tree
>
> https://www.cnblogs.com/nullzx/p/6192984.html
>
> https://blog.csdn.net/Sun_TTTT/article/details/65445754
>
> https://blog.csdn.net/cy973071263/article/details/122543826


## 附录

### 红黑树接口

```java
/**
 * 红黑树接口
 *
 * @author weijiaduo
 * @since 2022/12/21
 */
public interface RBTree {

    /**
     * 获取指定值的节点
     *
     * @param val 指定值
     * @return val对应的节点/null
     */
    Integer get(int val);

    /**
     * 插入新节点
     *
     * @param val 新值
     */
    void insert(int val);

    /**
     * 删除指定值的节点
     *
     * @param val 指定值
     */
    void remove(int val);

}
```

### 红黑树实现

```java
/**
 * 双偏向（Both-Leaning）红黑树（2-3-4树）
 * <p>
 * 1. 红色节点可以出现在左右两边
 *
 * @author weijiaduo
 * @since 2023/2/2
 */
public class BLRBTree implements RBTree {

    /**
     * 红色
     */
    private static final boolean RED = true;
    /**
     * 黑色
     */
    private static final boolean BLACK = false;

    /**
     * 根节点
     */
    private RBTNode root;

    @Override
    public Integer get(int val) {
        RBTNode node = get(root, val);
        return node != null ? node.val : null;
    }

    /**
     * 查找指定值对应的节点
     *
     * @param h   当前节点
     * @param val 指定值
     * @return 指定值的节点
     */
    private RBTNode get(RBTNode h, int val) {
        // 没有找到
        if (h == null) {
            return null;
        }
        // 找到对应的节点
        if (h.val == val) {
            return h;
        }
        if (h.val > val) {
            return get(h.left, val);
        } else {
            return get(h.right, val);
        }
    }

    @Override
    public void insert(int val) {
        if (get(val) != null) {
            return;
        }

        // 插入新节点
        RBTNode h = new RBTNode(val);
        root = insertNode(root, h);
        // 调整树结构
        balanceInsertion(h);

        // 根节点始终是黑色
        setColor(root, BLACK);
    }

    /**
     * 插入新节点
     *
     * @param h       当前节点
     * @param newNode 新节点
     * @return 新当前节点
     */
    private RBTNode insertNode(RBTNode h, RBTNode newNode) {
        if (h == null) {
            return newNode;
        }
        if (newNode.val < h.val) {
            h.left = insertNode(h.left, newNode);
            h.left.parent = h;
        } else {
            h.right = insertNode(h.right, newNode);
            h.right.parent = h;
        }
        return h;
    }

    /**
     * 修正插入后的红黑树结构
     *
     * @param h 新插入的节点
     */
    private void balanceInsertion(RBTNode h) {
        // 父节点是黑色时，不用调整
        RBTNode p = h.parent;
        if (!isRed(p)) {
            return;
        }

        // 父节点是红色
        RBTNode gp = p.parent;
        if (p == gp.left) {
            // LR 双红节点
            if (h == p.right) {
                rotateLeft(p);
            }
            // LL 双红节点
            p = rotateRight(gp);
        } else {
            // RL 双红节点
            if (h == p.left) {
                rotateRight(p);
            }
            // RR 双红节点
            p = rotateLeft(gp);
        }

        // 父子反色
        setColor(p, RED);
        setColor(p.left, BLACK);
        setColor(p.right, BLACK);

        // 递归往上处理
        balanceInsertion(p);
    }

    @Override
    public void remove(int val) {
        RBTNode h = get(root, val);
        if (h == null) {
            return;
        }

        // 替换节点（后继/前继）
        h = swapReplacer(h);
        // 先调整树结构
        balanceDeletion(h);
        // 真正删除节点
        removeNode(h);

        // 根节点始终是黑色
        setColor(root, BLACK);
    }

    /**
     * 交换当前节点到后继/前继节点
     *
     * @param h 当前节点
     * @return 替换节点
     */
    private RBTNode swapReplacer(RBTNode h) {
        RBTNode replacer = h;
        if (h.left != null && h.right != null) {
            replacer = h.right;
            while (replacer.left != null) {
                replacer = replacer.left;
            }
        } else if (h.right != null) {
            replacer = h.right;
        } else if (h.left != null) {
            replacer = h.left;
        }

        // TODO: 简单点，仅替换值，节点不变
        int val = h.val;
        h.val = replacer.val;
        replacer.val = val;
        return replacer;
    }

    /**
     * 移除节点（后继最小值/前继最大值）
     *
     * @param h 被移除节点，确保没有孩子或只有1个孩子
     */
    private void removeNode(RBTNode h) {
        RBTNode p = h.parent;
        if (p == null) {
            if (h.left == null) {
                root = h.right;
            } else {
                root = h.left;
            }
        } else {
            if (h == p.left) {
                // 后继最小值
                p.left = h.right;
            } else {
                // 前继最大值
                p.right = h.left;
            }
        }
        h.parent = h.left = h.right = null;
    }

    /**
     * 修正删除节点前的红黑树结构
     *
     * @param h 被删除节点
     */
    private void balanceDeletion(RBTNode h) {
        // 删除红色节点，不影响平衡性
        RBTNode p = h.parent;
        if (p == null || isRed(h)) {
            return;
        }

        // 删除黑色节点
        if (h == p.left) {
            // 1. 兄弟是红色，先转成黑色
            RBTNode s = p.right;
            if (isRed(s)) {
                rotateLeft(p);
            }
            // 2. 从兄弟借红色孩子
            if (borrowRight(h)) {
                return;
            }
            // 3. 父节点下溢合并
            if (underflow(h)) {
                return;
            }
        } else {
            // 镜像处理
            RBTNode s = p.left;
            if (isRed(s)) {
                rotateRight(p);
            }
            if (borrowLeft(h)) {
                return;
            }
            if (underflow(h)) {
                return;
            }
        }

        // 递归向上调整
        balanceDeletion(p);
    }

    /**
     * 父节点下溢，父子节点合并
     *
     * @param h 当前节点
     * @return true下溢后已平衡/false下溢后未平衡
     */
    private boolean underflow(RBTNode h) {
        RBTNode p = h.parent;
        RBTNode s;
        if (p.left == h) {
            s = p.right;
        } else {
            s = p.left;
        }
        boolean balanced = isRed(p);
        setColor(p, BLACK);
        setColor(s, RED);
        return balanced;
    }

    /**
     * 从右兄弟借一个红色节点过来
     *
     * @param h 当前节点
     * @return true借用成功/false借用失败
     */
    private boolean borrowRight(RBTNode h) {
        RBTNode p = h.parent;
        RBTNode s = p.right;
        RBTNode sl = s.left, sr = s.right;
        if (!isRed(sl) && !isRed(sr)) {
            // 没有红色节点可借
            return false;
        }

        // 远侄子是黑色，近侄子是红色
        if (!isRed(sr)) {
            rotateRight(s);
        }
        // 远侄子是红色
        p = rotateLeft(p);
        setColor(p.left, BLACK);
        setColor(p.right, BLACK);
        return true;
    }

    /**
     * 从左兄弟借一个红色节点过来
     *
     * @param h 当前节点
     * @return true借用成功/false借用失败
     */
    private boolean borrowLeft(RBTNode h) {
        RBTNode p = h.parent;
        RBTNode s = p.left;
        RBTNode sl = s.left, sr = s.right;
        if (!isRed(sl) && !isRed(sr)) {
            // 没有红色节点可借
            return false;
        }

        // 远侄子是黑色，近侄子是红色
        if (!isRed(sl)) {
            rotateLeft(s);
        }
        // 远侄子是红色
        p = rotateRight(p);
        setColor(p.left, BLACK);
        setColor(p.right, BLACK);
        return true;
    }

    /**
     * 左旋，将红色节点转到左边
     *
     * @param h 当前节点
     * @return 新当前节点
     */
    private RBTNode rotateLeft(RBTNode h) {
        if (h == null || h.right == null) {
            return h;
        }

        RBTNode p = h.parent;
        RBTNode r = h.right;
        r.parent = p;
        if (p == null) {
            root = r;
            r.color = BLACK;
        } else {
            r.color = p.color;
            if (h == p.left) {
                p.left = r;
            } else {
                p.right = r;
            }
        }

        h.right = r.left;
        if (h.right != null) {
            h.right.parent = h;
        }
        h.parent = r;
        r.left = h;
        h.color = RED;
        return r;
    }

    /**
     * 右旋，将红色节点转到右边
     *
     * @param h 当前节点
     * @return 新当前节点
     */
    private RBTNode rotateRight(RBTNode h) {
        if (h == null || h.left == null) {
            return h;
        }

        RBTNode p = h.parent;
        RBTNode l = h.left;
        l.parent = p;
        if (p == null) {
            root = l;
            l.color = BLACK;
        } else {
            l.color = p.color;
            if (h == p.left) {
                p.left = l;
            } else {
                p.right = l;
            }
        }

        h.left = l.right;
        if (h.left != null) {
            h.left.parent = h;
        }
        h.parent = l;
        l.right = h;
        h.color = RED;
        return l;
    }

    /**
     * 设置节点颜色
     *
     * @param h     节点
     * @param color 颜色
     */
    private void setColor(RBTNode h, boolean color) {
        if (h != null) {
            h.color = color;
        }
    }

    /**
     * 是否是红色
     *
     * @param h 节点
     * @return true/false
     */
    private boolean isRed(RBTNode h) {
        return h != null && h.color == RED;
    }

}
```
