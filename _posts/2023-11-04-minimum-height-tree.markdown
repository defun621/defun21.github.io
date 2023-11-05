---
layout: post
title:  最小高度树（上）
date:   2023-11-04 20:54:08 -0700
categories: [Algorithm-design-with-Haskell]
---

教科书链接：<https://www.cambridge.org/core/books/algorithm-design-with-haskell/824BE0319E3762CE8BA5B1D91EEA3F52>

- 这里讨论树上贪婪算法时候用的树的类型是leaf-labeled tree：

    ```scala
    enum Tree[+A]:
        case Leaf(a: A)
        case Node(l: Tree[A], r: Tree[A])

    object Tree:
        def apply[A](a: A): Tree[A] = Leaf(a)
        def apply[A](l: Tree[A], r: Tree[A]): Tree[A] = Node(l, r)
    ```

- 回顾一下基本util函数

    - size：

        ```scala
        val size: [A] => Tree[A] => Int = [A] => (t: Tree[A]) => t match
            case Leaf(_) => 1
            case Node(l, r) => size(l) + size(r)
        ```

    - height

        ```scala
        val height: [A] => Tree[A] => Int = [A] => (t: Tree[A]) => t match
            case Leaf(_) => 0
            case Node(l, r) => 1 + Math.max(height(l), height(r))
        ```

    - fringe / flatten

        ```scala
        val fringecat: [A] => Tree[A] => List[A] => List[A] = [A] => (t: Tree[A]) => l => t match
            case Leaf(a) => a :: l
            case Node(l, r) => fringecat(l)(fringecat(r)(l))
        val fringe: [A] => Tree[A] => List[A] = [A] => (t: Tree[A]) => t match
            case Leaf(a) => List(a)
            case Node(l, r) => fringecat(t)(Nil)
        ```

    - mkTree

        ```scala
        val mkTree: [A] => List[A] => Tree[A] = [A] => (l: List[A]) => unwrap(until(single)(pairWith(Tree.apply.uncurried))(l.map(Tree(_))))
        ```

- 最小高度树问题：给定一列自然数，能否有一个线性时间算法以最小cost构造出fringe为该list的一棵树，其中cost为：

    ```scala
    val cost: Tree[Int] => Int = (t: Tree[Int]) => t match
        case Leaf(x) => x
        case Node(l, r) => 1 + Math.max(cost(l), cost(r))
    ```

    该问题还可以理解为：给定一列tree和它们的高度构成的pair，能否给出一个在线性时间内将这组tree合并成一颗高度最低的树，但仍保持tree的fringe。

- 算法的签名是：

    ```scala
    val mct: List[Int] => Tree[Int]
    ```

    它的refinement为：

    mct &larr; MinWith cost (mktrees xs)

    其中mktrees 负责生成所有的candidates.

- mktrees 可以这么定义：

    ```scala
    val extend: [A] => A => Tree[A] => List[Tree[A]] = [A] => (a: A) => t => t match
        case Leaf(_) => Tree(Tree(a), t)
        case Node(l, r) => Tree(Tree(a), t) :: extend(a)(l).map(u => Tree(u, r))

    val mkTrees: [A] => List[A] => List[Tree[A]] = [A] => (l: List[A]) => l match
        case List(x) => List(Tree(x))
        case x :: xs => 
            val base = mkTrees(xs).flatMap(t => extend(x)(t))
    ```

    为了能套入foldr fusion law和greedy condition，

    ```scala
    def foldrn[A, B](l: List[A])(f: (A, B) => B)(g: A => B): B = l match
        case List(x) => g(x)
        case x :: xs => f(x, foldrn(xs)(f)(g))

    val mkTrees: [A] => List[A] => List[Tree[A]] = [A] => (l: List[A]) => 
        type TL[A] = List[Tree[A]]
        val f: (A, TL[A]) => TL[A] = (x: A, tl: TL[A]) => tl.flatMap(t => extend(x)(t))
        val g: A => TL[A] = (a: A) => List(Tree(a))
        foldrn(l)(f)(g)
    ```

- mktrees 也可以这么定义：

    - 先定义forest，也就是a list of trees

        ```scala
        type Forest[A] = List[Tree[A]]
        ```

    - forest可以roll成一棵树：

        ```scala
        val rollup: [A] => Forest[A] => Tree[A] = [A] => (ts: Forest[A]) => ts match
            case List(t) => t
            case x :: y :: xs => xs.foldLeft(Tree(x, y))(Tree.apply)
        ```

    - spine 是rollup的反函数：

        ```scala
        val spinecat: [A] => Tree[A] => Forest[A] => Forest[A] = [A] => (t: Tree[A]) => ts => t match
            case Leaf(_) => t :: ts
            case Node(l, r) => spinecat(l)(r :: ts)

        val spine: [A] => Tree[A] => Forest[A] = [A] => (t: Tree[A]) => spinecat(t)(Nil)
        ```

    - 所以成立： ```spine (rollup ts) = ts```

    - mkforests

        ```scala
        val extend: [A] => A => Forest[A] => List[Forest[A]] = [A] => (x: A) => ts => 
            for {
                k <- (1 to ts.size).toList
                tail = ts.drop(k)
                head = rollup(ts.take(k))
            } yield Tree(x) :: head :: tail

        val mkforests: [A] => List[A] => List[Forest[A]] = [A] => (l: List[A]) => 
            type FL[A] = List[Forest[A]]
            val f: (A, FL[A]) => FL[A] = (a: A, fl: FL[A]) => fl.flatMap(ts => extend(a)(ts))
            val g: A => FL[A] = (a: A) => List(List(Tree(a)))
            foldrn(l)(f)(g)
        ```

    - 最后，有了mkforests，我们可以定义mktrees

    ```scala
    val mkTrees: [A] => List[A] => List[Tree[A]] = [A] => (l: List[A]) => mkforests(l).map(f => rollup(f))
    ```

    下一篇继续学习如何在上述两个定义下进行refine从而得到最终的贪婪算法


    
---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}