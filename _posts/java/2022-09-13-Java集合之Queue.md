---
layout:       post
title:        "Java集合之Queue"
author:       "Libing"
header-style: text
catalog:      true
tags:
    - Java
    - 算法

    
---

> 平时很少用到集合派生类Queue相关，恐怕那是因为掌握得不够好吧

### Queue

先来一张类图

<img src="/img/java/queue/arraydeque.png">

Queue 是单端队列，只能从一端插入元素，另一端删除元素，实现上一般遵循 先进先出（FIFO） 规则。

Queue 扩展了 Collection 的接口，根据因为容量问题而导致操作失败后处理方式的不同
可以分为两类方法: 一种在操作失败后会抛出异常，另一种则会返回特殊值，我也记不住，具体请看源码。

Deque 是双端队列，在队列的两端均可以插入或删除元素。既可以实现队列的先进先出，也可以实现栈的先进后出。

Deque 扩展了 Queue 的接口, 增加了在队首和队尾进行插入和删除的方法，同样根据失败后处理方式的不同分为两类。

下面着重从Deque的实现类ArrayDeque来描述两个核心的设计要点。一个是实例化时的初始大小计算，一个是add元素时的指针计算。

### ArrayDeque

在初始化ArrayDeque时，可以指定默认的容量大小。但是ArrayDeque会转换为2的幂次方的容量大小。假设传递的是9，则会转换为16的容量。

```java
    private static int calculateSize(int numElements) {
        int initialCapacity = MIN_INITIAL_CAPACITY;
        // Find the best power of two to hold elements.
        // Tests "<=" because arrays aren't kept full.
        if (numElements >= initialCapacity) {
            initialCapacity = numElements;
            initialCapacity |= (initialCapacity >>>  1);
            initialCapacity |= (initialCapacity >>>  2);
            initialCapacity |= (initialCapacity >>>  4);
            initialCapacity |= (initialCapacity >>>  8);
            initialCapacity |= (initialCapacity >>> 16);
            initialCapacity++;

            if (initialCapacity < 0)   // Too many elements, must back off
                initialCapacity >>>= 1;// Good luck allocating 2 ^ 30 elements
        }
        return initialCapacity;
    }
```
上面的位移运算，以及后面的+1保证了扩展后的长度一定是2的幂次方值。为什么呢？大家可以算一算。我按自己的理解和计算如下

`>>>` 是无符号右移操作，`|`是位或操作，经过五次右移和位或操作可以保证得到大小为2^k-1的数。 

看一下这个例子：
```
 0 0 1 ? ? ? ? ?     //n
 0 0 1 1 ? ? ? ?     //n |= n >>> 1;
 0 0 1 1 1 1 ? ?     //n |= n >>> 2;
 0 0 1 1 1 1 1 1     //n |= n >>> 4;
 0 0 1 1 1 1 1 1     //n |= n >>> 8;
```
这里省略了右移16位的操作，实际上右移8位也没有意义。因为每一位都已经是1，和自身进行或运算都会是1，所以再移动已经没有意义。

但是一定要执行完这5次移位，因为按整数最大32位计算，除去最高位符号位，则余下31位，需要最多执行这5次移位才能达到从低位到高位的
每一位都是1的效果。

可见，在进行5次位移操作和位或操作后就可以得到2^k-1。
最后加1的目的是：当我们+1之后会不断的进位，最终达到只有一个比特位置是1的效果，因此它一定是2的整数倍。
。奇妙啊！


### 双指针位移

在进行添加元素时，如addFirst和addLast时，会利用位运算计算head或tail的指针，以便找到位置存放数据。我们先看一下addFirst。
```java
    /**
     * Inserts the specified element at the front of this deque.
     *
     * @param e the element to add
     * @throws NullPointerException if the specified element is null
     */
    public void addFirst(E e) {
        if (e == null)
            throw new NullPointerException();
        elements[head = (head - 1) & (elements.length - 1)] = e;
        if (head == tail)
            doubleCapacity();
    }
```
首先看，我们先假设都执行addFirst方法，看队头新增元素时，头指针是如何移位的。
通过` (head - 1) & (elements.length - 1)`进行了位移操作。head初始为0，那么head-1=-1，二进制每一位都是1。
而elements默认初始大小为16，16-1=15，这里15代表的是数组下标最大为15。我们知道&是位与符，对应的两个二进制相同位
都是1，结果为1，否则为0。所以任意2个二进制相与，结果都不会超过最大的那个二进制，所以这里elements.length - 1就相当于起到了边界限制的作用。

所以第一次，当-1与上elements.length - 1，则结果一定是elements.length - 1，那么head的指针就移动到了数组最大的边界处，按默认初始容量，head指向15的下标。
而后，每一次循环，都执行head-1。则(head - 1) & (elements.length - 1)的结果会依次减1，头指针一直往前移，直到head与tail相遇，进行了扩容。


举例：

假设现在head的指针指向10。tail的指针指向9。现在执行addfirst，head-1=9。9&15=9。
这时head和tail指针相同，进行扩容。因为要保证elements.length - 1作为边界，也就是需要1个最大值。 
而扩大为2倍表示为2的x次方，最高位一定是1，那么再-1，则低位的每一位都是1，这样保证了边界是最大值，也就保证了指针不越界。
---

下面我们看看addLast方法,和addFirst很像，但是确有差别。

```java
    public void addLast(E e) {
        if (e == null)
            throw new NullPointerException();
        elements[tail] = e;
        if ( (tail = (tail + 1) & (elements.length - 1)) == head)
            doubleCapacity();
    }
```
这里我们可以直观看出亮点差别
1. 对于head，是先移动指针位置再赋值。对于tail，是先赋值再移动指针位置
2. 对于head，移动指针是通过head-1&边界。对于tail，是通过tail+1&边界。所以我们可以知道他们俩始终是一个从远到近，再到重合的一个过程。

### 扩容

紧接着上面的例子，执行了doubleCapacity，分析一下doubleCapacity的源码
```java
    private void doubleCapacity() {
        assert head == tail;
        int p = head;
        int n = elements.length;
        int r = n - p; // number of elements to the right of p
        int newCapacity = n << 1;
        if (newCapacity < 0)
            throw new IllegalStateException("Sorry, deque too big");
        Object[] a = new Object[newCapacity];
        System.arraycopy(elements, p, a, 0, r);
        System.arraycopy(elements, 0, a, r, p);
        elements = a;
        head = 0;
        tail = n;
    }
```
这里可以看到，每次扩容都是执行n<<1，也就是当前容量的2倍。扩容完成后，通过拷贝原始数组到新数组。
同时head指针指向头部，tail指针指向尾部。重复的里程又重新开始！

---
后记：还是要画一些图会描述得更清晰些。有空补一下。

