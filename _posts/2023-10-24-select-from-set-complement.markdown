---
layout: post
title:  从set的补集里select
date:   2023-10-24 22:54:08 -0700
categories: [Algorithm-design-with-Haskell]
---

教科书链接：<https://www.cambridge.org/core/books/algorithm-design-with-haskell/824BE0319E3762CE8BA5B1D91EEA3F52>

- 给定一个未排序的不含重复元素的自然数list，找到没在这个list里的最小自然数

    该问题的specification应该是：

    ```scala
    def minus[A](xs: LazyList[A])(ys: LazyList[A]): LazyList[A] = xs.filter(i => !ys.contains(i))

    def select(xs: List[Int]): Int = minus(LazyList.form(0))(LazyList.from(xs)).head
        
    ```

    这样的实现时间复杂度为O(n<sup>2</sup>)

- 如果先排序的话，时间复杂度可以到O(nlogn)。

    ```scala
    def select(xs: List[Int]): Int = selectFrom(xs.sorted)(0)

    def selectFrom(l: List[Int])(k: Int): Int = l match
        case Nil => k
        case x :: xs => 
            if k == x then selectFrom(xs)(k + 1)
            else k

    ```
- 我们可以利用分治法来在线性时间内解决这个问题：

    我们需要把list分为两个等长的sublist。然后在其中一个符合条件的sublist上继续。

- split

    假设存在某个合适的自然数b，我们可以把输入xs分成两段。

    ```scala
    val partition: [A] => (A => Boolean) => List[A] => (List[A], List[A]) = [A] => (p: A => Boolean) => xs => 
        type ACC = (List[A], List[A])
        val initAcc: ACC = (Nil, Nil)
        val step: (A, ACC) => ACC = (x: A, acc: ACC) => 
            val (as, bs) = acc
            if p(x) then (x :: as, bs)
            else (as, x :: bs)
        xs.foldRight(initAcc)(step)
    ```

    则可以证明：
    ```haskell
    [0..] \\ xs -- \\ 即 minus
    = ([0 .. b - 1] ++ [b ..]) \\ xs
    = [0 .. b - 1] \\ xs ++ [b ..] \\ xs
    = [0 .. b - 1] \\ vs \\ us ++ [b ..] \\ us \\ vs -- (us,vs) = partition (< b) xs，并且 as \\ xs = as \\ us \\ vs = as \\ vs \\ us
    = [0 .. b - 1] \\ us ++ [b ..] \\ vs -- 因为[0 .. b - 1] \\ vs == [0 .. b - 1]，[b ..] \\ us = [b ..]
    ```

- 又因为在Haskell里有：

    ```haskell
    head (xs ++ ys) = if null xs then head ys else head xs
    ```

    所以有：

    ```scala
    def select(xs: List[Int]): Int = 
        val (as, bs) = partition(i => i < b)(xs)
        val l1 = minus(LazyList.range(0, b))(LazyList.from(as))
        val l2 = minus(LazyList.from(b))(LazyList.from(bs))
        if l1.isEmpty then l2.head
        else l1.head
    ```

- 问题的关键就在于选取合适的b

    因为问题中给的是不含重复元素的list，

    所以有：

    ```haskell
    null([0 .. b - 1] \\ ys) = (length ys == b)
    ```

    第一步像之前一样，我们可以泛化一下select，得到selectFrom

    ```scala
    def selectFrom(a: Int)(xs: List[Int]): Int = 
        xs match
            case Nil => a
            case _ =>
                val (ys, zs) = partition(i => i < b)(xs)
                if ys.size == (b - a) then selectFrom(b)(zs)
                else selectFrom(a)(ys)
    ```

    如何选取b呢？b值应该是a + 1 + &#x230A; n / 2 &#x230B;。

    这样的话，如果p < b - a, 则 p &le; b - a - 1 = &#x230A; n / 2 &#x230B;。如果p = b - a，则q = n - (b - a) = n - &#x230A; n / 2 &#x230B; - 1 &le; &#x230A; n / 2 &#x230B;。

- 最终我们可以得到

    ```scala
    def selectFrom(a: Int)(n: Int, xs: List[Int]): Int = n match
        case 0 => a
        case _ =>
            val b = a + 1 + n / 2
            val (ys, zs) = partition(i => i < b)(xs)
            val l = ys.size
            if l == b - a then selectFrom(b)(zs)
            else selectFrom(a)(ys)

    def select(a: Int)(xs: List[Int]): Int = selectFrom(0)(xs.size, xs)
    ```

    该实现的复杂度递推式为： T(n) = T(n/2) + &Theta;(n)。可得T(n) = &Theta;(n)




---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}