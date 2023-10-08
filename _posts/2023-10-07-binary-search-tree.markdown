---
layout: post
title:  binary search tree
date:   2023-10-07 23:54:08 -0700
categories: [Algorithm-design-with-Haskell]
---

教科书链接：<https://www.cambridge.org/core/books/algorithm-design-with-haskell/824BE0319E3762CE8BA5B1D91EEA3F52>

- BST里Tree的ADT定义和之前RAList不同。第一版BST的Tree的ADT定义为

    ```scala
        enum Tree[+A]:
            case Null
            case Node(v: A, left: Tree[A], right: Tree[A])
    ```

    为什么说这是第一版，因为到后面我们发现在Node里光有这些信息不够使其保持平衡。

- 一些基本工具函数

    - size

        ```scala
        val size: [A] => Tree[A] => Int = [A] => (t: Tree[A]) => t match
            case Null => 0
            case Node(_, l, r) => 1 + size(l) + size(r)
        ```

    - flatten
    
        ```scala
        val flatten: [A] => Tree[A] => List[A] = [A] => (t: Tree[A]) => t match
            case Null => Nil
            case node(v, l, r) = flatten(l) ++ (v :: flatten(r))
        ```

        注意：这个算法的时间复杂度很糟糕，因为因为每次都要++的关系。

        像之前RAList的toList一样，我们可以引入一个函数来优化时间复杂度。

        ```scala
        val flatcat: [A] => Tree[A] => List[A] => List[A] = [A] => (t: Tree[A]) => l => t match
            case Null => l
            case Node(v, left, right) => flatcat(left)(v :: flatcat(right)(l))
        ```

        通过flatcat，通过递归消除了++。flatten就可以定义为：
        ```scala
        val flatten: [A] => Tree[A] => List[A] = [A] => (t: Tree[A]) => flatcat(t)(Nil)
        ```

    - height

        ```scala
        val height: [A] => Tree[A] => Int = [A] => (t: Tree[A]) => t match 
            case Null => 0
            case Node(v, l, r) => 1 + Math.max(height(l), height(r))
        ```

- 一种通用的抽象是认为Tree[A]里的A是一种record，每个record有其对应的unique key。这样的话可以给出search的定义

    ```scala
    def search[A, K](key: A => K)(k: K)(t: Tree[A])(using ord: Ordering[K]): Option[A] = t match 
        case Null => None
        case Node(v, l, r) => 
            ord.compare(key(v), k) match
                case 0 => Some(v)
                case x if x < 0 => search(key)(k)(l)
                case _ => search(key)(k)(r)
    ```

- 显然为了让binary search tree搜索时间最优，我们要让binary search tree平衡，所以构造函数List[A] => Tree[A]要尽可能的把list平均二分。
    
    为了达到这个目的，我们需要稍微修改一下我们的Tree ADT。

    ```scala
    enum Tree[+A]:
        case Null
        case Node(height: Int, a: A, left: Tree[A], right: Tree[A])
    ```

    引入height是为了在构造的时候更方便做平衡。

- 在新ADT的基础上，height是一个O(1)的函数。

    ```scala
    val height: [A] => Tree[A] => Int = [A] => (t: Tree[A]) => t match
        case Null => 0
        case Node(h, _, _, _) => h
    ```

- 一颗平衡的BST可以这么构造：

    ```scala
    def mkTree[A](xs: List[A])(using ord: Ordering[A]): Tree[A] = 
        val step: (A, Tree[A]) => Tree[A] = (a: A, t: Tree[A]) => insert(a)(t)
        xs.foldRight(Null)(step)
    ```

    insert定义为：
    ```scala
    def insert[A](a: A)(t: Tree[A])(using ord: Ordering[A]): Tree[A] = t match
        case Null => Tree(a, Null, Null) //这里Tree是一个smart constructor，等于Node(1 + Math.max(height(l), height(r)), a, l, r)
        case n @ Node(h, x, l, r) =>
            ord.compare(a, x) match
                case 0          => n
                case i if i < 0 => balance(insert(a)(l))(x)(r)
                case _          => balance(l)(x)(insert(a)(r))
    ```

    问题在于balance 函数的定义，这里就和AVL树一样了，需要旋转。

- balance函数

    - 如果左子树和右子树高度差不超过1，则Node(1 + Math.max(height(l), height(r)), x, l, r)

    - 如果左子树高度比右子树高2，则

        - 如果左子树l的左子树ll的高度不小于l的右子树rl，则我们有

            height r = height l - 2 = height ll - 1 &le; height rl &le; height ll

            此时我们只需要变左子树的root为新的root，做一次右旋

        - 如果ll的高度小于rl，则我们有：

            height r = height l - 2 = height rl - 1 = = height ll = max(height lrl, height rrl)

            ,其中lrl为rl的左子树，rrl为rl的右子树

            此时我们需要变rl的root为新的root，先做一次左旋再做一次右旋

    - 右子树高的情况与上述对称。

    - 写成scala就是:

        ```scala
        val bias: [A] => Tree[A] => Int = [A] => (t: Tree[A]) => t match
            case Null => 0
            case Node(_, _, l, r) => height(l) - height(r)

        val rotr: [A] => Node[A] => Node[A] = [A] => (n: Node[A]) => n match
            Node(_, x, Node(_, y, ll, rl), r) => Tree(y, ll, Tree(x, rl, r))

        val rotl: [A] => Node[A] => Node[A] = [A] => (n: Node[A]) => n match
            Node(_, y, ll, Node(_, z, lrl, rrl)) => Tree(z, Tree(y, ll, lrl), rrl)

        val rotateR: [A] => Tree[A] => A => Tree[A] => Tree[A] = [A] => (l: Tree[A]) => a => r =>
            if bias(l) >= 0 then rotr(Tree(a, l, r))
            else rotr(Tree(a, rotl(t1), r))

        val rotateL: [A] => Tree[A] => A => Tree[A] => Tree[A] = [A] => (l: Tree[A]) => a => r =>
            if bias(r) <= 0 then rotl(Tree(a, l, r))
            else rotl(Tree(a, l, rotr(r)))

        val balance: [A] => Tree[A] => A => Tree[A] => Tree[A] = [A] => (l: Tree[A]) => a => r => 
            val h1 = height(l)
            val h2 = height(r)
            if Math.abs(h1 - h2) <= 1 then Tree(a, l, r)
            else if h1 == h2 + 2 then rotateR(l)(a)(r)
            else if h2 == h1 + 2 then rorateL(l)(a)(r)
            else throw new IllegalStateException
        ```

            


---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}