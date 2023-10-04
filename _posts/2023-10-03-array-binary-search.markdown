---
layout: post
title:  array和二分法
date:   2023-10-03 21:54:08 -0700
categories: [Algorithm-design-with-Haskell]
---

教科书链接：<https://www.cambridge.org/core/books/algorithm-design-with-haskell/824BE0319E3762CE8BA5B1D91EEA3F52>

- Array
    在Haskell的Data.Array里，array是immutable的。

    构造函数定义为
    ```haskell
    array :: Ix i => (i,i) -> [(i,e)] -> Array i e
    ```

    接受一个表示bound的pair，一个由(i,e)构成的association list。

    在Scala里，我们有Array，well，虽然它的Ix类型固定为Nat。

- 一维上的二分法
    一维上的二分要解决的问题可以抽象为：给定一个严格单调递增的函数f，和目标数t，我们在数域上要找到x，使得f(x) = t。

    对于我们常见的Int数组上的二分，f即```num.apply```
    ```haskell
        search :: (Nat -> Nat) -> Nat -> [Nat]
    ```
    返回值要么是一个空list，要么是一个singleton。

    线性的搜索的话，search的实现为：
    ```scala
        val search : (Int => Int) => Int => List[Int] = (f: Int => Int) => t => (0 to t).filter(i => f(i) == t).toList
    ```

    二分的话，首先因为f单调递增，
    可以先定义出一个bound函数，通过2的幂次去找p，使得f(2<sup>p - 1</sup>) &lt; t &le; f(2<sup>p</sup>)
    ```scala
        val bound: (Int => Int) => Int => (Int, Int) = 
            (f: Int => Int) => 
                t => 
                    val until: [A] => (A => Boolean) => (A => A) => A => A = [A] => (p: A => Boolean) => f => x => if p(x) then x else until(p)(f)(f(x))
                    val b: Int = until(x => t <= f(x))(x => x * 2)(1)
                    if t <= f(0) then (-1, 0) else (b / 2, b)
    ```
    然后
    在定义smallest函数，通过二分去找到该bound中最小的值x，使得t &le; f(x)。
    ```scala
        val smallest: (Int, Int) => (Int => Int) => Int => Int = (a: Int, b: Int) => f => t => 
            if a + 1 == b then b
            else 
                val m = (a + b) / 2
                if t < f(m) then smallest(a, m)(f)(t)
                else smallest(m, b)(f)(t)
    ```

    有了这两个之后，我们可以定义search为：
    ```scala
        val search: (Int => Int) => Int => List[Int] = (f: Int => Int) => t => 
            val (a, b) = bound(f)(t)
            val x = smallest(a, b)(f)(t)
            if f(x) == t then List(x) else Nil
    ```

- 在二维上的二分

  二维上的二分要解决的问题可以抽象为
  
  给定一个函数f, 他接受一对Nat作为参数，返回值为Int。函数f在每个argument 上都严格递增。给定目标t，求所有的(x,y)使得f(x,y) = t。

  &Theta;(n<sup>2</sup>)的实现很简单：

  ```scala
    val search: ((Int, Int) => Int) => Int => List[(Int, Int)] = (f: (Int, Int) => Int) => t => 
        for {
            x <- (0 to t).toList
            y <- 0 to t
            if f(x, y) == t
        } yield (x, y)
        
  ```

  我们来一步一步优化：

  1. 从矩阵的top-left而不是bottom-left开始搜索

  ```scala
    val search: ((Int, Int) => Int) => Int => List[(Int, Int)] = (f: (Int, Int) => Int) => t => 
        for {
            x <- (0 to t).toList
            y <- t to 0 by -1
            if f(x, y) == t
        } yield (x, y)
  ```

  2. make search interval explicit
  ```scala
    val searchIn: (Int, Int) => ((Int, Int) => Int) => Int => List[(Int, Int)] => (a: Int, b: Int) => f => t => 
        for {
            x <- (a to t).toList
            y <- b to 0 by -1
            if f(x, y) == t
        } yield (x, y)
  ```

  3. saddleback search显然有：
     - 如果f(a, b) < t, 则a + 1
     - 如果f(a, b) > t, 则b - 1
     - 如果f(a, b) == t， 则a + 1, b - 1

     ```scala
        val search: ((Int, Int) => Int) => Int => List[(Int, Int)] = f => t =>
            def searchIn(x: Int, y: Int): List[(Int, Int)] = 
                if x > t || y < 0 then Nil
                else
                    val z = f(x, y)
                    if z < t then searchIn(x + 1, y)
                    else if z == t then（x, y) :: searchIn(x + 1, y - 1)
                    else searchIn(x, y - 1)
     ```

    Saddleback search 的时间复杂度为&Theta;(m + n) + &Theta;(t)。

    search里的起点(t,0)，（t,0)选的不好，如果将
    
    - p定义为 smallest (-1, t) (&lambda;y.f(0,y)) t
    - q定义为 smallest (-1, t) (&lambda;x.f(x,0)) t

    则此时saddleback search 的时间复杂度为&Theta;(logt) + &Theta; (p + q)。

    下一篇讲书里怎么定义在二维上正确的进行binary search。

---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}