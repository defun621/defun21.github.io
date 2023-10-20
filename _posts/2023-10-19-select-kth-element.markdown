---
layout: post
title:  分治思想：select第k大元素
date:   2023-10-19 21:59:08 -0700
categories: [Algorithm-design-with-Haskell]
---

教科书链接：<https://www.cambridge.org/core/books/algorithm-design-with-haskell/824BE0319E3762CE8BA5B1D91EEA3F52>

- 求min或max，就是一个fold

    ```scala
    def max[A: Ordering](as: List[A]): A = 
        val ord = summon[Ordering[A]]
        as.tail.foldRight(as.head)(ord.max)

    def min[A: Ordering](as: List[A]): A = 
        val ord = summon[Ordering[A]]
        as.tail.foldRight(as.head)(ord.min)
    ```

- 同时求min和max，可以利用tupling的技巧，一个fold解决

    ```scala
    def minmax[A: Ordering](as: List[A]): (A, A) = 
        val ord = summon[Ordering[A]]
        type R = (A, A)
        val step: (A, R) => R = (a: A, acc: R) => (ord.min(a, acc._1), ord.max(a, acc._2))
        as.tail.foldRight((as.head, as.head))(step)
    ```
    这样的话比较次数是2n - 2。

    可以通过分治法来进一步减少比较次数：

    ```scala
    def minmax[A: Ordering](as: List[A]): (A, A) = 
        val ord = summon[Ordering[A]]
        as match
            case List(x) => (x, x)
            case x :: y :: Nil => if ord.compare(x, y) < 0 then (x, y) else (y, x)
            case _ =>
                val (xs, ys) = halve(as)
                val (a1, b1) = minmax(xs)
                val (a2, b2) = minmax(ys)
                (ord.min(a1, a2), ord.max(b1, b2))
    ```

    halve 在之前定义过：
    ```scala
    val halve: [A] => List[A] => (List[A], List[A]) = [A] => (l: List[A]) => l match
        type R = (List[A], List[A])
        val initR: R = (Nil, Nil)
        val step: (A, R) => R = (a: A, r: R) => 
            val (xs, ys) = r
            (ys, a :: xs)
        l.foldRight(initR)(step)
    ```

    按照第二种定义的话，不妨令长为n的List求minmax需要的比较次数为C(n)。则
    
    C(1) = 0

    C(2) = 1

    C(n) = C(&#x2308;n/2&#x2309;) + C(&#x230A;n/2&#x230B;) + 2

    如果n是2的幂次，则C(n) = 3n / 2 - 2。

- select 第k小元素

    如果利用sort的话，可以如下这么定义，select的时间复杂度为O(nlogn)：

    ```scala
    def select[A: Ordering](k: Int)(as: List[A]): A = 
        val sorted = as.sorted
        sorted(k - 1)
        
    def median[A: Ordering](as: List[A]): A = 
        val k = (as.size() + 1) / 2
        select(k)(as)
    ```

    但是可以利用分治法使得select的时间复杂度为&Theta;(n)。关键在于选择何时的pivot，把list分成三段。

    我们先给出根据某个pivot，把list分成3段的方法partition3的定义：
    ```scala
    def partition3[A: Ordering](p: A)(as: List[A]): (List[A], List[A], List[A]) = 
        type R = (List[A], List[A], List[A])
        val ord = summon[Ordering[A]]
        val initR: R = (Nil, Nil, Nil)
        val step: (A, R) => R = (a: A, r: R) => ord.compare(a, p) match
            case x if x < 0 => (a :: r._1, r._2, r._3)
            case 0 => (r._1, a :: r._2, r._3)
            case _ => (r._1, r._2, a :: r._3)
        as.foldRight(initR)(step)
    ```
    显然partition3的时间复杂度为&Theta;(n)

    分成三段之后，可以递归的定义select:
    ```scala
    def select[A: Ordering](k: Int)(as: List[A]): A = 
        val (xs, ys, zs) = partition3(pivot(as))(as)
        val m = xs.size
        val n = ys.size
        if k <= m then select(k)(xs)
        else if k <= m + n then ys(k - m - 1)
        else select(k - m - n)(zs)
    ```

    关键就在于怎么定义pivot函数。

    可以先把数组5个5个分组：

    ```scala
    val group: [A] => Int => List[A] => List[List[A]] = [A] => (n: Int) => xs => xs match
        case Nil => Nil
        case _ => 
            (ys, zs) = xs.splitAt(n)
            ys :: (group(n)(zs))
    ```

    然后计算这每5个数的中位数构成的list:
    ```scala
    def medians[A: Ordering](as: List[A]): List[A] = 
        def middle(xs: List[A]): A = xs((xs.size + 1) / 2 - 1)
        group(5)(as).map(l => middle(l.sorted))
    ```

    有了以上这些，我们就可以定义pivot了：

    ```scala
    def pivot[A: Ordering](as: List[A]): A = as match
        case List(x) => x
        case _ => 
            val ms = medians(as)
            select((ms.size + 1) / 2)(ms)
    ```

    这么定义的select时间复杂度是多少？

    算pivot的时间复杂度为T(&#x2308;n/5&#x2309;) + &Theta;(n)

    在最差情况下，每次用pivot分段后，可以删掉的element个数为： &#x230A;3 &times;(&#x2308;n/5&#x2309; + 1)/2&#x230B; - 1 &ge; 3n/10。

    所以剩下的元素个数最多为7n/10。

    所以递归式为： T(n) &le; T(n/5) + T(7n/10) + cn

    可以凑出来，当b = 10c时候，bn/5 + 7bn/10 + cn &le; bn。

    所以T(n) = &Theta;(n).

    

---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}