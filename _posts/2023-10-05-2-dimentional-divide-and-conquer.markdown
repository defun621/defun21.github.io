---
layout: post
title:  二维里的二分
date:   2023-10-05 20:54:08 -0700
categories: [Algorithm-design-with-Haskell]
---

教科书链接：<https://www.cambridge.org/core/books/algorithm-design-with-haskell/824BE0319E3762CE8BA5B1D91EEA3F52>

上一篇我们讲到了saddleback search。这一篇来学习two dimentional divide and conquer。问题还是同一个问题，

给定一个函数f, 他接受一对Nat作为参数，返回值为Int。函数f在每个argument 上都严格递增。给定目标t，求所有的(x,y)使得f(x,y) = t。

- 假设矩阵左上角是(x<sub>1</sub>, y<sub>1</sub>), 右下角是(x<sub>2</sub>, y<sub>2</sub>)。

    我们考察f(x, y), 其中 x = &#x230A;(x<sub>1</sub> + x<sub>2</sub>)/2&#x230B; ， y = &#x230A;(y<sub>1</sub> + y<sub>2</sub>)/2&#x230B;

    如果f(x, y) &lt; t，则x,y左下角那块可以剔除；如果f(x,y) &gt; t，则(x,y)右上角那块可以剔除。如果f(x, y) = t，则这两块都可以剔除。

    如果我们这样写的话，时间复杂度的递推式为：

    T(m, n) = 1 + T(m / 2, n / 2) + T(m / 2, n)

    最后会得到T(m, n) = 2<sup>logm</sup>(logn - (logm) / 2 + 1) - 1 &le; mlog(2n / &radic;m)

    这并不是最优。

- 正确的二分做法是，

    我们先考察中间的那行 r = &#x230A;(y<sub>1</sub> + y<sub>2</sub>)/2&#x230B;，在这行上找到这么一个x，使得

    ```scala
        x = smallest(x1 - 1, x2)(x => f(x, r))(t) //smallest实现参考上一篇
    ```

    对比t和f(x, r)， 如果f(x,r) &lt; t, 

    用Scala来实现的话就是：

    ```scala
        
        val search: ((Int, Int) => Int) => Int => List[(Int, Int)] = (f: (Int, Int) => Int) => t =>
            def from(x1: Int, y1: Int)(x2: Int, y2: Int): List[(Int, Int)] = 
                val c = (x1 + x2) / 2
                val r = (y1 + y2) / 2
                if (x1 > x2 || y1 < y2) then Nil
                else if y1 - y2 <= x2 - x1 then
                    val x = smallest(x1 - 1, x2)(i => f(i, r))(t)
                    val z = f(x, r)
                    if z < t then from(x1, y1)(x2, r + 1)
                    else if z == t then (x, r) :: (from(x1, y1)(x - 1,r + 1) ++ from(x + 1, r - 1)(x2, y2))
                    else from(x1, y1)(x - 1,r + 1) ++ from(x + 1, r - 1)(x2, y2)
                else
                    val y = smallest(y2 - 1, y1)(i => f(c, i))(t)
                    val z = f(c, y)
                    if z < t then from(c + 1, y1)(x2, y2)
                    else if z == t then (c, y) :: (from(x1, y1)(c - 1, y + 1) ++ from(c + 1, y - 1)(x2, y2))
                    else from(x1, y1)(c - 1, y + 1) ++ from(c + 1, y - 1)(x2, y2)

            val p = smallest(-1, t)(i => f(0, i))(t)
            val q = smallest(-1, t)(i => f(i, 0))(t)

            from(0, p)(q, 0)
    ```

    该实现在最坏情况下，

    T(m, n) = logn + 2T(m/2, n/2)，最终可以得到一个logT = &Omega;(mlog(1 + n/m) + nlog(1 + m/n))的下界。

- **注意：该下界表明，当m = n时，时间复杂度起码是&Omega;(m + n)，所以当m = n时，saddleback search是最优**
===

---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}