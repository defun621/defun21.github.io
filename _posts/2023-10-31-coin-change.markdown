---
layout: post
title:  找零问题
date:   2023-10-31 21:54:08 -0700
categories: [Algorithm-design-with-Haskell]
---


教科书链接：<https://www.cambridge.org/core/books/algorithm-design-with-haskell/824BE0319E3762CE8BA5B1D91EEA3F52>

- 找零问题如何套入list上贪婪的范式 ```minWith cost . candidates```呢？

- 因为不同国家有不同的货币单位

    所以我们可以首先定义货币计价单位：

    ```scala
    type Denom = Int
    type Tuple = List[Int] 

    val usds: List[Denom] = List(100, 50, 25, 10, 5, 1) //1刀、5角、25美分、一角、5美分和1美分
    val ukds: List[Denom] = List(200, 100, 50, 20, 10, 5, 2, 1)
    ```

- 有了货币单位之后，可以给出amount函数

    ```scala
    val amount: List[Denom] => Tuple => Int = (ds: List[Denom]) => cs => 
        ds.zip(cs).map(t => t._1 * t._2).sum
    ```

- 找零问题就可以定义为：

    ```scala
    val mkChange: List[Denom] => Int => Tuple = (ds: List[Denom]) => amount => 
        minWith(t => t.sum)(mktuples(ds)(amount))
    ```

- 其中mktuple函数生成所有的candidates

    生成式的定义可以是：

    ```scala
    val mkTuples: List[Denom] => Int => List[Tuple] = (ds: List[Denom]) => n => ds match
        case List(1) => List(List(n))
        case x :: xs => 
            for {
                c <- (0 to n / x).toList
                cs <- mkTuples(xs)(n - c * x)
            } yield c :: cs
    ```

    这样的枚举出来中间结果很耗时间。

- 此外，minWith(t => t.sum) 可能不唯一，所以我们需要改变目标函数。

    把目标函数改成 maxWith id。这样的话，mkChange得到的是唯一的lexically最大的tuple。

    该tuple是否是硬币数最少的tuple取决于货币单位。

- 为了以后推到方便，我们需要重写一下mktuples。

    ```scala
    val mkTuples: List[Denom] => Int => List[Tuple] = (ds: List[Denom]) => n => ds match
        case List(1) => List(List(n))
        case x :: xs => 
            def extend(us: List[Denom])(v: Int): List[Tuple] = mkTuples(us)(n - x * v).map(vs => v :: vs)
            (0 to n/x).toList.flatMap(c => extend(xs)(c))
    ```

- 又因为存在以下关系：

    ```haskell
    maximum (concatMap f xs) = maximum (map (maximum . f) xs)
    maximum (map (x:) xs) = x : maximum xs
    ```

- 所以有：

    ```haskell
    mkchange (d : ds) n
    = maximum (mktuples (d : ds) n)
    = maximum (concatMap (extend ds) [0. .m]) -- m = n div d
    = maximum (map (maximum · extend ds) [0. .m]) -- 上面第一条规则
    ```

    ```haskell
    maximum (extend ds c)
    = maximum (map (c:) (mktuples ds (n - c * d)))
    = c : maximum (mktuples ds (n - c * d)) -- 上面第二条rule
    = c : mkchange ds (n - c * d) -- mkchange 定义
    ```

- 综上有：

    ```haskell
    maximum (map (maximum . extend ds) [0. .m])
    = m: mkchange ds (n - m * d)
    ```

- 所以最后我们可以得到简化后的省略产生中间结果的最终mkChange

    ```scala
    val mkChange: List[Denom] => Int => Tuple = (ds: List[Denom]) => n => ds match
        case List(1) => List(n)
        case x :: xs => 
            val c = n / x
            c :: mkChange(xs)(n - c * x)
    ```

- 除了生成式来算mkTuples以外，我们还可以用foldr来得到mkTuples

    ```scala
    val mkTuples: List[Denom] => Int => List[Tuple] = (ds: List[Denom]) => n => 
        type ACC = List[(Tuple, Int)]
        val init: ACC = List((Nil, n))
        def extend(d: Denom, p: (Tuple, Int)): ACC = 
            for {
                c <- (0 to p._2 / d).toList
            } yield (p._1 ++ List(c), p._2 - c * d)
        def step(d: Denom, acc: ACC): ACC = acc.flatMap(a => extend(d, a))
        ds.reverse.foldRight(init)(step).filter(pair => pair._2 == 0).map(_._1)
    ```

- 有了foldr 我们就可以尝试用fusion law 和maximum来避免产生中间结果。

    回顾foldr 的fusion law：
    
    对于有限长度的list xs来说，下式成立

    ```haskell
    h (foldr f e xs) = foldr g e' xs
    ```
    
    其中```e' = h e```，此外还要满足fusion condition ```h (f x y) = g x (h y)```
    
- 现在 

    h 为 ```maximum```，
    
    xs是```reverse ds```，
    
    e是```[([], n)]```，

    f是```concatMap . extend```

    则h e 即为 ```([], n)```。

    h y 即为所有(cs, r)两元组的list中第一个element lexically maximum的。

    为了保证这个特性，易得

    ```haskell
    g d (cs, r) = (cs ++ [c], r - c * d)
        where c = r div d
    ```

- 所以通过fusion law，我们可以得到：

    ```scala
    val mkChange: List[Denom] => Int => Tuple = (ds: List[Denom]) => n => 
        type ACC = (Tuple, Int)
        val init: ACC = (Nil, n)
        def step(d: Denom, acc: ACC): ACC = 
            val c = acc._2 / d
            (acc._1 ++ List(c), acc._2 - c * d)
        ds.reverse.foldRight(init)(step)._1
    ```

---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}