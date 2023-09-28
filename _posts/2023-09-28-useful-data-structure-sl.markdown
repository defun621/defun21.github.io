---
layout: post
title:  Useful Data structure - Symmetric List
date:   2023-09-28 10:54:08 -0700
categories: [Algorithm-design-with-Haskell]
---

教科书链接：<https://www.cambridge.org/core/books/algorithm-design-with-haskell/824BE0319E3762CE8BA5B1D91EEA3F52>

+ list在cons, head, tail上的时间复杂度是O(1)，但snoc,last,init的时间复杂度是O(n)。而Symmetric List可以使两端操作都能达到均摊O(1)。
  
+ 定义：
  ```scala
    type SymList[A] = (List[A], List[A])
  ```

+ invariant
  + 如果pair里有一个为空，则另一个最多只有一个element
    ```haskell
    null xs => null ys || single ys
    null ys => null xs || single xs
    ```

+ 一些基本函数：
  + fromSL
    ```scala
    val fromSL: [A] => SymList[A] => List[A] = [A] => (sl: SymList[A]) => sl._1 ++ sl._2.reverse
    ```
  
  + consSL
    ```scala
    val consSL: [A] => A => SymList[A] => SymList[A] = [A] => (a: A) => sl => sl match 
      case (xs, Nil) => (List(a), xs)
      case (xs, ys) => (a :: xs, ys)
    ```

  + snocSL
    ```scala
    val snocSL: [A] => A => SymList[A] => SymList[A] = [A] => (a: A) => sl => sl match
      case (Nil, ys) => (ys, List(a))
      case (xs, ys) => (xs, a :: ys) 
    ```

  + headSL
    ```scala
    val headSL: [A] => SymList[A] => A = [A] => (sl: SymList[A]) => sl match 
      case (Nil, ys) => ys.head
      case (xs, ys) => xs.head
    ```

  + lastSL
    ```scala
    val lastSL: [A] => SymList[A] => A = [A] => (sl: SymList[A]) => sl match
      case (xs, Nil) => xs.head
      case (_, ys) => ys.head
    ```

  + nilSL
    ```scala
    def nilSL[A]: SymList[A] = (Nil, Nil)
    ```

  + tailSL
    ```scala
    val tailSL: [A] => SymList[A] => SymList[A] = [A] => (sl: SymList[A]) => sl match
      case (Nil, ys) => if ys.nonEmpty then nilSL else throw new RuntimeException
      case (List(x), ys) => 
        val (us, vs) = ys.splitAt(ys.size / 2)
        (vs.reverse, us)
      case (xs, ys) => (xs.tail, ys)
    ```
  
  + initSL
    ```scala
    val initSL: [A] => SymList[A] => SymList[A] = [A] => (sl: SymList[A]) => sl match 
      case (xs, Nil) => if xs.nonEmpty then nilSL else throw new RuntimeException
      case (xs, List(_)) => 
        val (us, vs) = xs.splitAt(xs.size / 2)
        (us, vs.reverse)
      case (xs, ys) => (xs, ys.tail)
    ```

  + nullSL
    ```scala
    val nullSL: [A] => SymList[A] => Boolean = [A] => (sl: SymList[A]) => sl._1.isEmpty && sl._2.isEmpty
    ```

  + singleSL
    ```scala
    val singleSL: [A] => SymList[A] => Boolean = [A] => (sl: SymList[A]) => sl match
      case (Nil, List(_)) => true
      case (List(_), Nil) => true
      case _ => false
    ```

  + dropWhileSL
    ```scala
    val dropWhileSL: [A] => (A => Boolean) => SymList[A] => SymList[A] = [A] => (p: A => Boolean) => sl => sl match 
      case _ if nullSL(sl) => nilSL
      case _ if p(headSL(sl)) => dropWhileSL(p)(tailSL(sl))
      case _ => sl
    ```

  + lengthSL
    ```scala
    val lengthSL: [A] => SymList[A] => Int = [A] => (sl: SymList[A]) => sl._1.size + sl._2.size
    ```

  + initsSL
    ```scala
    val initsSL: [A] => SymList[A] => SymList[SymList[A]] = [A] => (sl: SymList[A]) => sl match
      case _ if nullSL(sl) => snocSL(sl)(nilSL)
      case _ => snocSL(sl)(initsSL(initSL(sl)))
    ```

+ 有了SymList之后，可以给出一个inits的实现使得length . inits 的时间复杂度为O(n)
  ```haskell
  inits = map fromSL . scanl (flip snocSL) nilSL
  ```

+ 当然，其实上面的秘诀在scanl，所以就算没有SymList，我们可以给出一个inits的实现使得length . inits 的时间复杂度为O(n)
  ```haskell
  inits = map reverse . scanl (flip (:)) []
  ```