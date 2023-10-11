---
layout: post
title:  更多关于二叉搜索树
date:   2023-10-10 21:54:08 -0700
categories: [Algorithm-design-with-Haskell]
---

教科书链接：<https://www.cambridge.org/core/books/algorithm-design-with-haskell/824BE0319E3762CE8BA5B1D91EEA3F52>

- 如果树的左子树、右子树仍是平衡的，但树的左子树和右子树的高度差超过2，也可以balance

    - 考察h1 &gt; h2 + 2的情况, h1是左子树的高度，h2是右子树的高度，x是根节点的值

        沿着左子树的右子树r1、左子树的右子树的右子树r2、……一直找下去，我们总能找到这么一颗子树r使得 0 &le; height(r) - height(t2) &le; 1。

        这是因为沿着右边找下去，每次子树r<sub>k</sub>的高度最多下降2.

        此外，不妨令l是r对应的左侧子树，则有abs(height(l) - height(Tree(x,r,t2))) &le; 2

        这是因为左子树t1是平衡的且abs(height(l) - height(r)) &le; 1。所以我们可以用上一篇中的balance方法来实现

        ```scala
        val balanceR: [A] => Tree[A] => A => Tree[A] => Tree[A] = [A] => (t1: Tree[A]) => x => t2 => t1 match
            case Node(_, y, l, r) => 
                if height(r) >= height(t2) + 2 then balance(l)(y)(balanceR(r)(x)(t2))
                else balance(l)(y)(Tree(x, r, t2))
        ```
    
    - h2 &gt; h1 + 2的的情况与上面对偶。

        ```scala
        val balanceL: [A] => Tree[A] => A => Tree[A] => Tree[A] = [A] => (t1: Tree[A]) => x => t2 => t2 match
            case Node(_, y, l, r) => 
                if height(l) >= height(t1) + 2 then balance(balanceL(t1)(x)(l))(y)(r)
                else balance(Tree(x, t1, l))(y)(r)
        ```

    - 把上面两种情况与之前的合并，我们就有一个新的方法gbalance

        ```scala
        val gbalance: [A] => Tree[A] => A => Tree[A] => Tree[A] = [A] => (l: Tree[A]) => x => r => 
            val h1 = height(l)
            val h2 = height(r)
            h1 - h2 match
                case i if i < -2 => balanceR(l)(x)(r)
                case i if -2 <= i && i <= 2 => balance(l)(x)(r)
                case _ => balanceL(l)(x)(r)
        ```

        balance的时间复杂度为&Theta;(1)，但balanceR和balanceL因为要沿着子树一直找下去，所以这个gbalance的时间复杂度为&Theta;(nlogn)。

- BST也可以用来代表Set.

    ```scala
        type Set[A] = Tree[A]
    ```

- 一些Set的工具函数

    ```scala
    def member[A](a: A)(s: Set[A])(using ord: Ordering[A]): Boolean = s match
        case Null => false
        case Node(_, x, l, r) => ord.compare(a, x) match
            case 0 => true
            case i if i < 0 => member(a)(l)
            case _ => member(a)(r)
    ```

    - 为了能够定义出Set/Tree上的remove操作，我们需要先研究怎么把两个set组合到一起（写出combine）。假定左边的Set里的值都小于右边Set里的值。

        - 一种方法是在较大那个里把最小值摘出来，然后讲两个set与该值合并成一个set，思路很直观，写起来也很简单。

            ```scala
            val deleteMin: [A] => Set[A] => (A, Set[A]) = [A] => (s: Set[A]) => s match
                case Node(_, x, Null, r) => (x, r)
                case Node(_, x, l, r) => 
                    val (v, t) = deleteMin(l)
                    (v, balance(t)(x)(r))

            val combine: [A] => Set[A] => Set[A] => Set[A] = [A] => (s1: Set[A]) => s2 => (s1, s2) match
                case (Null, _) => s2
                case (_, Null) => s1
                case _ => 
                    val (v, t) = deleteMin(s2)
                    balance(s1)(v)(t) //这里可以用balance因为s1与s2是s删掉root 之后的两个子树

            def delete[A](a: A)(s: Set[A])(using ord: Ordering[A]): Set[A] = s match
                case Null => Null
                case Node(_, x, l, r) => ord.compare(a,x) match
                    case i if i < 0 => balance(delete(a)(l))(x)(r)
                    case 0 => combine(l)(r)
                    case i if i > 0 => balance(l)(x)(delete(a)(r))
            ```

        - 此外，如果用我们先前定义的gbalance代替combine里的balance，我们就可以combine任意大小的两个set。


    - split，即给定一个值x，把Set分为两部分，一部分中的所有值都不大于x，另一部分中的所有值都大于x

        为了能计算出split，书里先给了一个ADT的定义Piece

        ```scala
        enum Piece[+A]:
            case LP(l: Set[A], x: A)
            case RP(x: A, r: Set[A])
        ```

        一个Piece就是一颗树缺了左子树或者右子树。

        我们首先需要把Set分成多个pieces

        ```scala
        def pieces[A](x: A)(s: Set[A])(using ord: Ordering[A]): List[Piece[A]] = 
            def addPiece(t: Set[A])(ps: List[Piece[A]]): List[Piece[A]] = t match
                case Null => ps
                case Node(_, y, l, r) => 
                    if x < y then addPiece(l)(new RP(y, r) :: ps)
                    else addPiece(r)(new LP(l, y) :: ps)

            addPiece(s)(Nil)
        ```

        分成多个pieces之后我们可以把list fold成我们想要的两部分
        ```scala
        val sew: [A] => List[Piece[A]] => (Set[A], Set[A]) = [A] => (l: List[Piece[A]]) => 
            type ACC = (Set[A], Set[A])
            val step: (ACC, Piece[A]) => ACC = (acc, p) => 
                val (t1, t2) = acc
                p match
                    case RP(x, t) => (t1, gbalance(t2)(x)(t))
                    case LP(t, x) => (gbalance(t)(x)(t1), t2)
            l.foldLeft((Null, Null))(step)
        ```
        显然split就是把pieces的结果sew在一起

        ```scala
        def split[A](a: A)(s: Set[A])(using ord: Ordering[A]): (Set[A], Set[A]) = sew(pieces(a)(s))
        ```

---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}