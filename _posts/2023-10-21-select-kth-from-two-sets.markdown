---
layout: post
title:  从两个list/set中select第k小元素
date:   2023-10-21 23:11:08 -0700
categories: [Algorithm-design-with-Haskell]
---

教科书链接：<https://www.cambridge.org/core/books/algorithm-design-with-haskell/824BE0319E3762CE8BA5B1D91EEA3F52>

- 如果是从两个sorted list或binar tree里select 第k个元素, 时间复杂度是多少呢？对于sorted list，select可以定义成：

    ```scala
    def select[A: Ordering](k: Int)(l1: List[A])(l2: List[A]): A = merge(l2)(l2)(k)
    ```

    问题的关键在于分治来推导下面这个

    ```merge (xs ++ [a] ++ ys) (us ++ [b] ++ vs) k```

- 假设存在关系：

    ```scala
    def <<==[A: Ordering](l1: List[A])(l2: List[A]): Boolean = 
        val ord = summon[Ordering[A]]
        val step: (Boolean, Boolean) => Boolean = (b, acc) => b && acc
        (for {
            x <- l1
            y <- l2
        } yield ord.lteq(x, y)).foldRight(true)(step)
    ```

    并且如果有```xs <<== vs``` 和 ```us <<== ys```，

    则
    ```merge (xs ++ ys) (us ++ vs) = merge(xs)(us) ++ merge (ys)(vs)```

- 又因为 ```xs <<== ys``` (因为xs ++ ys是sorted)，加上```xs <<== vs```,

    所以有 ```xs <<== merge(ys)(vs)```

    同理 ```us <<== merge(ys)(vs)```

    所以 ```merge(xs)(us) <<== merge(ys)(vs)```

- 假设 ```ys = ys1 ++ ys2```，其中ys1是使得```xs ++ [a] ++ ys1 <<== [b] ++ vs```成立的ys最长前缀，

    并且```us <<== ys2```，

    则有：

    ```merge (xs ++ [a] ++ ys1 ++ ys2) (us ++ [b] ++ vs) = merge (xs ++ [a] ++ ys1)(us) ++ merge(ys2)([b] ++ vs)```

- 同理，存在该情况的对偶情况：

    假设```us == us1 ++ us2```，其中us2是使得```xs ++ [a] <<== us2 ++ [b] ++ vs```的us的最长后缀，
    
    并且```us1 <<== ys```，

    则有：

    ```merge (xs ++ [a] ++ ys) (us1 ++ us2 ++ [b] ++ vs) = merge (xs ++ [a])(us1) ++ merge(ys)(us2 ++ [b] ++ vs)```

- 有了上述observation，我们可以对k进行分类讨论，这里只考虑 a &le; b 的情况。a &ge; b 的情况与下述结论对偶。

    - k &le; p + q （p、q是l1，l2的某段长度）

        ```haskell
        (merge (xs ++ [a] ++ ys) (us ++ [b] ++ vs)) !! k
        = (merge (xs ++ [a] ++ ys1 ++ ys2) (us ++ [b] ++ vs)) !! k
        = (merge (xs ++ [a] ++ ys1)(us) ++ merge(ys2)([b] ++ vs)) !! k
        = merge(xs ++ [a] ++ ys1)(us) !! k -- 因为k <= p + q
        = (merge(xs ++ [a] ++ ys1)(us) ++ merge(ys2) ([])) !! k -- 因为后面merge[] 没有影响，而且k <= p + q，所以无论后面++什么都不影响
        = merge(xs ++ [a] ++ ys)(us) !! k -- merge decomposition rule 反过来应用
        ```

    - k &gt; p + q

        ```haskell
        (merge (xs ++ [a] ++ ys) (us ++ [b] ++ vs)) !! k
        = (merge (xs ++ [a] ++ ys) (us1 ++ us2 ++ [b] ++ vs)) !! k
        = (merge (xs ++ [a])(us1) ++ merge (ys)(us2 ++ [b] ++ vs)) !! k -- merge decomposition rule
        = merge (ys)(us2 ++ [b] ++ vs) !! (k - p - q1 - 1) -- k > p + q, q1 是us1 的长度
        = (merge([])(us1) ++ merge(ys)(us2 ++ [b] ++ vs)) !! (k - p - 1)
        = merge(ys)(us ++ [b] ++ vs) !! (k - p - 1)
        ```


- 有了上述分治结论, 对于sorted list上的select，我们可以给出定义：

    ```scala
    def select[A: Ordering](k: Int)(l1: List[A])(l2: List[A]): A = (l1, l2) match
        case (Nil, _) => l2(k)
        case (_, Nil) => l1(k)
        case _ => 
            val p = l1.size / 2
            val q = l2.size / 2
            (xs, a :: ys) = l1.splitAt(p)
            (us, b :: vs) = l2.splitAt(q)
            val ord = summon[Ordering[A]]
            if (ord.lteq(a, b) && k <= (p + q)) then select(k)(l1)(us)
            else if (ord.lteq(a, b) && k >= (p + q)) then select(k - p - 1)(ys)(l2)
            else if(ord.gt(a, b) && k <= (p + q)) then select(k)(xs)(l2)
            else select(k - q - 1)(l1)(vs)
    ```

    该实现的时间复杂度是多少？

    T(m, n) = Math.max(T(m, n /2), T(m / 2, n)) + &Theta;(m + n)

    可以得到T(m, n) = &Theta;(m + n)

- 对于set或者binary search tree，select可以定义为：


    ```scala
    enum Tree[A]:
        case Null
        case Node(x: A, h: Int, left: Tree[A], right: Tree[A])

    val size: [A] => Tree[A] => Int = [A] => (t: Tree[A]) => t match
        case Null => 0
        case Node(_, h, _, _) => h

    val index: [A] => Tree[A] => Int => A = [A] => (t: Tree[A]) => k => t match
        case Null => throw new IllegalStateException
        case Node(x, h, l, r) => 
            val p = size(l)
            if k < p then index(l)(k)
            else if k == p then x
            else index(r)(k - p - 1)

    def select[A: Ordering](k: Int)(t1: Tree[A])(t2: Tree[A]): A = (t1, t2) match
        case (Null, _) => index(t2)(k)
        case (_, Null) => index(t1)(k)
        case (Node(a, h1, l1, r1), Node(b, h2, l2, r2)) => 
            val ord = summon[Ordering[A]]
            val p = size(l1)
            val q = size(l2)
            if(ord.lteq(a, b) && k <= (p + q)) then select(k)(t1)(l2)
            else if(ord.lteq(a, b) && k >= (p + q)) then select(k - p - 1)(r1)(t2)
            else if(ord.lteq(b, a) && k <= (p + q)) then select(k)(l1)(t2)
            else select(k - q - 1)(t1)(r2)

    ```

    该实现的时间复杂度是多少？

    T(m, n) = Math.max(T(m, n /2), T(m / 2, n)) + &Theta;(1)

    可以得到T(m, n) = &Theta;(logm + logn)

    

---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}