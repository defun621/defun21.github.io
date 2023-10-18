---
layout: post
title:  桶排序和基数排序
date:   2023-10-17 21:24:08 -0700
categories: [Algorithm-design-with-Haskell]
---

教科书链接：<https://www.cambridge.org/core/books/algorithm-design-with-haskell/824BE0319E3762CE8BA5B1D91EEA3F52>

- **discriminator**：一个从元素中“抽取”特征的函数，抽出来的特征可以用来排序。

    - 例如：

        - 十进制k位数可以有k个discriminator，把每一个位的digit都提取出来

- 有了一个discriminator list之后，可以判断两个参数是否有序：

    ```scala
    def ordered[A,B](fs: List[A => B])(x: A)(y: A)(using ord: Ordering[B]): Boolean = fs match
        case Nil => true
        case d :: ds => 
            val dx = d(x)
            val dy = d(y)
            if ord.compare(dx, dy) < 0 then true
            else if ord.compare(dx, dy) == 0 then ordered(ds)(x)(y)
            else false
    ```

- 实现桶排序要用到数据结构是rose tree。可以这样定义：

    ```scala
    enum Tree[+A]:
        case Leaf(a: A)
        case Node(children: List[Tree[A]])
    ```

- 给定一系列discriminator和数组，我们用以下方法构造rose tree

    ```scala
    def mkTree[A,B](fs: List[A => B])(xs: List[A])(using ord: Ordering[B], e: Enum[B]): Tree[List[A]] = fs match
        case Nil => Leaf(xs)
        case d :: ds => 
            val internal: List[List[A]] => List[Tree[A]] = l => l.map(a => mkTree(ds)(a))
            Node(ptn(d)(xs).map(internal))
    ```

    其中Enum是scalaz里的一个typeclass，有succ和pred。在Scala中还有min和max。

- 如何定义ptn函数，按照discriminator d把list分割成buckets？

    一个最简单的定义就是：

    ```scala
    def ptn[A, B](d: A => B)(xs: List[A])(using ord: Ordering[B], e: Enum[B]): List[List[A]] = 
        val rng = e.min |-> e.max
        for {
            m <- rng
            sub <- xs.filter(i => d(i) == m)
        } yield sub
    ```

    这个定义很不效率，一方面e.min 到e.max 的范围可能十分巨大，另一方面对于每个m都要重复遍历一次rng。如何优化？

    可以先把b的typecless 收小，使它变成Ix。（在scala貌似只有Int)。

    ```scala
    def ptn[A](d: A => Int)(xs: List[A]): List[List[A]] = 
        val zipped = xs.map(d).zip(xs)
        zip.groupMap(_._1)(_._2).values.toList
    ```

- 有了rose tree之后我们需要把它flatten，按照我们之前```sort = flatten . mkTree```的思路

    ```scala
    val flatten: [A] => Tree[A] => List[A] = [A] => (t: Tree[A]) => t match
        case Leaf(a) => List(a)
        case Node(children) => children.flatMap(flatten)
    ```

- 最后我们可以给出桶排序的定义：

    ```scala
    val bucketSort: [A] => List[A => Int] => List[A] => List[A] = [A] => (ds: List[A => Int]) => xs => flatten(mkTree(ds)(xs))
    ```

    尝试消去中间结果：

    ```haskell
    bsort (d:ds) xs
    = flatten (mkTree (d:ds) xs)
    = flatten (Node(map (mkTree ds) (ptn d xs)))
    = concatMap flatten (map (mkTree ds) (ptn d xs))
    = concatMap (flatten . mkTree ds) (ptn d xs)      -- concatMap f . map g = concatMap (f.g)
    = concatMap (bsort ds) (ptn d xs)
    ```

    所以bucketSort又可以定义为：

    ```scala
    val bucketSort: [A] => List[A => Int] => List[A] => List[A] = [A] => (ds: List[A => Int]) => xs => ds match
        case Nil => xs
        case d :: ds => ptn(d)(xs).flatMap(bucketSort(ds))
    ```

- 基数排序

    由于```map (bsort ds) (ptn d xs) = ptn d (bsort ds xs)```

    所以bsort的最后一步```concatMap (bsort ds) (ptn d xs)```还可以进一步化为：```concat (ptn d (bsort ds xs))```

    这就是基数排序的定义：

    ```scala
    val radixSort: [A] => List[A => Int] => List[A] => List[A] = [A] => (ds: List[A => Int]) => xs => ds match
        case Nil => xs
        case d :: ds => ptn(d)(radixSort(ds)(xs)).flatten
    ```

- 这章中，所有的算法都由一个基础思想```sort = flatten . mkTree```变化而得。这也是divide and conquer算法思想的体现。



---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}