---
layout: post
title:  以插入排序为例看list上的贪婪算法抽象
date:   2023-10-28 18:54:08 -0700
categories: [Algorithm-design-with-Haskell]
---

教科书链接：<https://www.cambridge.org/core/books/algorithm-design-with-haskell/824BE0319E3762CE8BA5B1D91EEA3F52>

- list上的贪婪法可以抽象为：

    ```haskell
    mcc :: [Component] -> Candidate
    mcc = minWith cost . candidates
    ```

- minWith 可以定义为：

    ```scala
    def minWith[A, B](f: A => B)(xs: List[A])(using ord: Ordering[B]): A = 
        val step: (A, A) => A = (x: A, acc: A) => 
            val cost1 = f(x)
            val cost2 = f(acc)
            if ord.lteq(cost1, cost2) then x else acc
        xs.tail.foldRight(xs.head)(step)
    ```

    因为在minWith里用的是ord.lteq，所以如果存在多个最小cost的candidate的话，上述minWith返回的第一个candidate。

    如果用的是ord.lt，则得到的是最后一个candidate。


- candidates函数可以定义为：

    ```haskell
    candidates :: [Component] -> [Candidate]
    candidates xs = foldr step [c0] xs
                    where step x cs = concatMap (extend x) cs
    ```

    extend 函数的类型是：

    ```haskell
    extend :: Component -> Candidate -> [Candidate]
    ```

    extend 函数即根据当前的component和candidate，返回一个包含“下一步”的所有candidate的list。

- 但是生成所有的candidates实在是有点costy。这里可以用fusion law来优化。

    对于foldr，对于所有有限长list xs，对于函数满足```h (f x y) = g x (h y)```，存在fusion law：
    ```h (foldr f e xs) = foldr g (h e) xs```

    对于上述贪婪算法抽象，有```h = minWith cost```和```f = step```，我们要求出g。

    也就是说：

    ```minWith cost (step x cs) = g x (minWith cost cs)```

    我们从左往右推理：

    ```haskell
    minWith cost (step x cs)
    = minWith cost (concatMap (extend x) cs) -- definition of step
    = minWith cost (map (minWith cost . extend x) cs) -- distributive law
    = minWith cost (map (g x) cs) -- 不妨令 g x = minWith cost . extend x
    = g x (minWith cost cs) -- 等式的右边
    ```

- 最终，我们可以得到：

    如果有 ```minWith cost (map (g x) cs) = g x (minWith cost cs)```，

    则我们有

    ```haskell
    mcc = foldr g c0
            where g x = minWith cost . extend x
    ```

- 对于排序，如果套用上面的mcc模型，一种做法是：

    ```scala
    def sort[A: Ordering](xs: List[A]): List[A] = minWith(ic)(perms(xs))
    ```

    其中，ic表示candidate中不满足序的要求的pair的数量。

    - ic可以定义为：

        ```scala
        def ic[A](xs: List[A])(using ord: Ordering[A]): Int = pairs(xs).filter((x, y) => ord.gt(x, y)).size

        val pairs: [A] => List[A] => List[(A, A)] = [A] => (l: List[A]) => 
            for {
                x :: ys <- l.tails.toList
                y :: zs <- ys.tails
            } yield (x, y)
        ```

    - perms 可以定义为：

        ```scala
        val extend: [A] => A => List[A] => List[List[A]] = [A] => (x: A) => l => l match
            case Nil => List(List(x))
            case (y :: xs) => (x :: y :: xs) :: extend(x)(xs).map(us => y :: us)

        val perms : [A] => List[A] => List[List[A]] = [A] => (l: List[A]) => 
            val init: List[List[A]] = List(Nil)
            val step: (A, List[List[A]]) => List[List[A]] = (a: A, acc: List[List[A]]) =>
                acc.flatMap(ll => extend(a)(ll))
            l.foldRight(init)(step)
        ```

    - 套入之前的g x 定义：```g x = minWith cost . extend x```

        我们有

        ```scala
        def gstep[A: Ordering](a: A)(l: List[A]): List[A] = minWith((xs: List[A]) => ic(xs))(extend(a)(l))
        ```

    但是，这样不满足我们的greedy condition：minWith cost (map (g x) cs) = g x (minWith cost cs)

- 为了满足greedy condition。我们需要更换cost function。

    除了看ic以外，还有一个很直接的cost，即id，

    也就是说：

    ```haskell
    sort = minWith id . perms = minimum . perms
    ```

    即：求lexically least permutation。

    - 此时，greedy condition为：

        ```minimum (map (g x) (perms xs)) = g x (minimum (perms xs))```

        ，其中 ```g x xs = minimum (extend x xs)```

    - 因为minimum是唯一的，并且如果xs是sorted， 则gstep x xs也是sorted。

        所以可以：

        ```scala
        def gstep[A: Ordering](x: A)(xs: List[A]): List[A] = xs match
            case Nil => List(x)
            case y :: ys => 
                val ord = summon[Ordering[A]]
                if ord.lteq(x, y) then x :: y :: ys else y :: gstep(x)(ys)
        ```

    定义了gstep之后，带入到fusion law之后的，

    ```scala
    def sort[A: Ordering](xs: List[A]): List[A] = xs.foldRight(List.empty[A])((a, acc) => gstep(a)(acc))
    ```

    这里，我们就得到了insertion sort。


    
    
---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}