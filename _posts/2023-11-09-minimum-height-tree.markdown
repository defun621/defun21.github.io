---
layout: post
title:  最小高度树（下）
date:   2023-11-09 15:54:08 -0800
categories: [Algorithm-design-with-Haskell]
---

- 继续推理，为了能得到最优mct，我们先看第一个definition

    ```scala
    val extend: [A] => A => Tree[A] => List[Tree[A]] = [A] => (a: A) => t => t match
        case Leaf(_) => Tree(Tree(a), t)
        case Node(l, r) => Tree(Tree(a), t) :: extend(a)(l).map(u => Tree(u, r))

    def foldrn[A, B](l: List[A])(f: (A, B) => B)(g: A => B): B = l match
        case List(x) => g(x)
        case x :: xs => f(x, foldrn(xs)(f)(g))

    val mkTrees: [A] => List[A] => List[Tree[A]] = [A] => (l: List[A]) => 
        type TL[A] = List[Tree[A]]
        val f: (A, TL[A]) => TL[A] = (x: A, tl: TL[A]) => tl.flatMap(t => extend(x)(t))
        val g: A => TL[A] = (a: A) => List(Tree(a))
        foldrn(l)(f)(g)
    ```


- 回忆context-sensitive fusion law：

    为了foldrn f<sub>2</sub> g<sub>2</sub> xs &larr; M (foldrn f<sub>1</sub> g<sub>1</sub> xs) 对于任意优先非空list xs成立，

    我们要有:

    - g<sub>2</sub> &larr; M (g<sub>1</sub> x)

    - f<sub>2</sub> x (M (foldrn f<sub>1</sub> g<sub>1</sub> xs)) &larr; M (f<sub>1</sub> x (foldrn f<sub>1</sub> g<sub>1</sub> xs))

    其中

    - M = MinWith cost

    - f<sub>1</sub> = concatMap . extend

    - g<sub>1</sub> = wrap . leaf

- 因为 Leaf x = MinWith cost [Leaf x]

    所以 g<sub>2</sub> = Leaf

- 所以我们这里主要求：

    gstep x (MinWith cost (mktrees xs)) &larr; MinWith cost (concatMap (extend x) (mktrees xs))

    在之前研究list上的贪婪法时，我们知道他的前提是得满足：

    cost t &le; cost t' &rArr; cost (gstep x t) &le; cost (gstep x t')

- 不存在这样的gstep使得上式满足， 所以我们又得更换cost 函数

    我们来看lexical order

    可以将根据lexical order得到的lcost定义为：

    ```scala
    val lcost : Tree[Int] => List[Int] = (t: Tree[Int]) => 
        val costs = spine(t).map(cost(_))
        costs.tail.scanLeft(costs.head)((x, y) => 1 + Math.max(x, y)).reverse
    ```

- 用lcost重写fusion condition，我们有：

    gstep x (MinWith lcost (mktrees xs)) &larr; MinWith lcost (concatMap (extend x) (mktrees xs))

    此时可以证明：

    lcost t &le; lcost t' &rArr; lcost(gstep x t) &le; lcost (gstep x t')

    其中

    gstep x ts &larr; MinWith lcost (extend x ts)

- 通过分析，gstep x ts的目标是 **(x max c<sub>j</sub>) &lt; cost t<sub>j + 1</sub>，如果这样的j不存在，则j = n.**

    ```scala
    val join: Int => List[Tree[Int]] => List[Tree[Int]] = (x: Int) => ts => ts match
        case List(u) => List(u)
        case u :: v :: us => if Math.max(x, cost(u)) < cost(v) then u :: v :: us else join(x)(Node(u,v) :: ts)

    val add: Int => List[Tree[Int]] => List[Tree[Int]] = (x: Int) => ts => Tree(x) :: join(x)(ts)
        
    val gstep : Int => Tree[Int] => Tree[Int] = (x: Int) => t => rollup(add(x)(spine(t)))

    val mct: List[Int] => Tree[Int] = (l: List[Int]) => foldrn(l)(gstep)(x => Tree(x))
    ```

- 但是在上述实现中，每一步都要rollup。我们可以再通过一次fusion law来把rollup 拉出来

    ```scala
    val mct: List[Int] => Tree[Int] = (l: List[Int]) => rollup(foldrn(l)(add)(x => List(Tree(x))))
    ```

- 此外我们可以把cost与tree pair起来，用tupling的技巧避免重复计算cost

    所以最后我们可以得到：

    ```scala
    type Pair = (Tree[Int], Int)

    val leaf: Int => Pair = (x: Int) => (Tree(x), x)

    val node: Pair => Pair => Pair = (u: Pair) => v => (Tree(u._1, v._1), 1 + Math.max(u._2, v._2))

    val join: Int => List[Pair] => List[Pair] => (x: Int) => ts => ts match
        case List(u) => List(u)
        case u :: v :: us => if Math.max(x, u._2) < v._2 then ts else join(x)(node(u)(v) :: us)

    val hstep: Int => List[Pair] => List[Pair] => (x: Int) => ts => leaf(x) :: join(x)(ts)
 
    val mct: List[Int] => Tree[Int] = (l: List[Int]) => rollup(foldrn(l)(hstep)(x => List(leaf(x))).map(_._1))
    ```

---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}