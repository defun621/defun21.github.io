---
layout: post
title:  huffman树（上）
date:   2023-11-11 22:54:08 -0800
categories: [Algorithm-design-with-Haskell]
---

教科书链接：<https://www.cambridge.org/core/books/algorithm-design-with-haskell/824BE0319E3762CE8BA5B1D91EEA3F52>

- huffman树的目的在于得到一个满足如下条件的encoding：

    如果字符c<sub>i</sub>，1 &le; i &le; n，其出现频率为p<sub>i</sub>，它对应的encoding长度为l<sub>i</sub>，则&Sigma;<sub>i=1</sub><sup>n</sup>l<sub>i</sub>p<sub>i</sub>要最小

- 这里仅考察如何构造huffman树。问题输入定义为：

    给定一列元组[(c<sub>1</sub>, w<sub>1</sub>), ..., (c<sub>n</sub>, w<sub>n</sub>)]

    其中c<sub>i</sub>是字符，w<sub>i</sub>是weight。不妨假定w<sub>1</sub> &le; w<sub>2</sub> &le; ... &le; w<sub>n</sub>


- 然后是定义cost：

    ```scala
    type Weight = Int
    type Elem = (Char, Weight)
    type Cost = Int

    val depths: [A] => Tree[A] => List[Int] = [A] => (t: Tree[A]) =>
        def from(n: Int)(r: Tree[A]): List[Int] = r match
            case Leaf(_) => List(n)
            case Node(u, v) => from(n + 1)(u) ++ from(n + 1)(v)

        from(0)(t)

    val cost: Tree[Elem] => Cost = (t: Tree[Elem]) => 
        val flatten = fringe(t)
        val depths = depths(t)

        flatten.zip(depths).map(p => p._1._2 * p._2).sum
    ```

- 所以综上，套用之前mct的例子，我们可以得到huffman的定义：

    huffman &larr; MinWith cost . mktrees

    其中mktrees负责根据给定的fringe生成所有的candidate trees

- 然而这个specification过于strong，给定的list不一定要是树的fringe。

    所以我们这里的mktrees的目标是生成unordered binary tree。


- 要定义新的mktrees，我们需要先定义combine。

    combine从forest里取出一组tree，构造一个新的树，然后按照weight重新插入list中。

    - picks：要定义combine，我们先要有函数能从list中取出一对candidate。

        ```scala
        val picks: [A] => List[A] => List[(A, List[A])] = [A] => (l: List[A]) => l match
            case Nil => Nil
            case x :: xs => 
                val tail = for {
                    (y, ys) <- picks(xs)
                } yield (y, x :: ys)
                (x, xs) :: tail

        val pairs: [A] => List[A] => List[((A, A), List[A])] = [A] => (l: List[A]) => 
            for {
                (x, xs) <- picks(l)
                (y, ys) <- picks(xs)
            } yield ((x, y), ys)
        ```

    - 其次要定义函数insert，负责把构造出的node按照weight重新插入list

        ```scala
        val weight: Tree[Elem] => Int = (t: Tree[Elem]) => t match
            case Leaf(elem) => elem._2
            case Node(u, v) => weight(u) + weight(v)

        val insert: Tree[Elem] => Forest[Elem] => Forest[Elem] = (t: Tree[Elem]) => ts => ts match
            case Nil => List(t)
            case t1 :: tail => if weight(t) < weight(t1) then t :: ts else t1 :: insert(t)(tail)
        ```

    - 有了上述函数之后，我们可以定义combine

        ```scala
        val combine: Forest[Elem] => List[Forest[Elem]] = (f: Forest[Elem]) => 
            for {
                ((t1, t2), ts) <- pairs(f)
            } yield insert(Tree(t1, t2))(ts)
        ```

- 有了combine，mktrees可以定义为：

    ```scala
    val mkforests: List[Tree[Elem]] => List[Forest[Elem]] = (ts: List[Tree[Elem]]) =>
        val init: List[Forest[Elem]] = List(ts)
        until(init)(fs => fs.allMatch(f => f.size == 1))(fs => fs.flatMap(combine(_)))

    val mktrees: List[Elem] => List[Tree[Elem]] = (l: List[Elem]) => 
        mkforests(l.map(Tree(_))).map(_.head)
    ```

- 到此，huffman的目标变成：

    找到function gstep，使得

    **unwrap (until single gstep ts) &larr; MinWith cost (map unwrap (mkforests ts))**



---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}