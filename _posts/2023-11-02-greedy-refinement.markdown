---
layout: post
title:  list上greedy algorithm用refinement reasoning
date:   2023-11-02 21:54:08 -0700
categories: [Algorithm-design-with-Haskell]
---

教科书链接：<https://www.cambridge.org/core/books/algorithm-design-with-haskell/824BE0319E3762CE8BA5B1D91EEA3F52>

- 如果minWith cost不能有unique candidate，要么换cost function，要么找方法使得greedy condition成立

    ```haskell
    gstep x (minWith cost (candidates xs)) = minWith cost (map (gstep x) (candidates xs))
    ```

- 如果对于minimum cost存在多个candidate，

    必须要有如下性质：

    对于任意c 和c'，如果 cost c &le; cost c' &hArr; cost (gstep x c) &le; cost (gstep x c')

- 该式问题在于，现实情况中很难保证它被满足

    我们需要一种别的方式使得我们在如下单调关系成立的时候可以使用fusion law

    cost c &le; cost c' &rArr; cost (gstep x c) &le; cost (gstep x c') 

- 书里引入了 &larr;

    x &larr; MinWith cost xs

    即从xs的cost最小的元素构成的集合中选取任意一个，

    用set membership代替了equation reasoning

- 更一般的，&larr; 还可以表示集合包含

    即E<sub>1</sub> &larr; E<sub>2</sub>就是表示 v &larr; E<sub>1</sub> &rArr; v &larr; E<sub>2</sub>

- 此外，x &larr; E(E<sub>1</sub>)表示 存在y，使得 y &larr; E<sub>1</sub>并且 x &larr; E(y)

- 通过上述3个定义，所有的expression在refinement的定义下是monotonic的。

- 如何证明 gstep x (MinWith cost (candidates xs)) &larr; MinWith cost (map (gstep x) (candidates xs))?

    左边即表示

    对于某个c', c' &larr; MinWith cost (candidates xs)，并且c = gstep x c'，则有 c &larr; gstep x (MinWith cost (candidates xs))

    因为cost c &le; cost c' &rArr; cost (gstep x c) &le; cost (gstep x c') 

    所以c' 作为candidates xs中cost最小的元素之一，也同样是MinWith cost (map (gstep x) (candidates xs))中的cost最小元素之一。

    所以c &larr; MinWith cost (map (gstep x) (candidates xs))

    所以gstep x (MinWith cost (candidates xs)) &larr; MinWith cost (map (gstep x) (candidates xs))

- 如何证明 MinWith cost (concat xss) &larr; MinWith cost (map (MinWith cost) xss)?

    左边表示有 x &larr; MinWith cost (concat xss)

    不妨令xss = [xs<sub>1</sub>, ..., xs<sub>n</sub>]

    是所有xss中cost最小的元素之一。

    同时，也存在某个i，使得x 是xs<sub>i</sub>中的元素。

    不妨令y<sub>i</sub> &larr; MinWith cost xs<sub>i</sub>。

    则 [y1,.., y<sub>i-1</sub>, x, y<sub>i+1</sub>,.., yn] &larr; map (MinWith cost) xss 

    又因为x是最小的之一，所以x &larr; [y1,.., y<sub>i-1</sub>, x, y<sub>i+1</sub>,.., yn]

    所以x &larr; MinWith cost (map (MinWith cost) xss)

- 如何证明 MinWith cost (map (MinWith cost) xss) &larr; MinWith cost (concat xss)?

    不妨令xs &larr; map (MinWith cost) xss， 其中xs = [x<sub>1</sub>, .., x<sub>n</sub>]

    也就是说对于任意i，有x<sub>i</sub> &larr; MinWith cost xs<sub>i</sub>。

    不妨令x<sub>k</sub> &larr; MinWith cost xs

    所以对于任意i，有cost x<sub>k</sub> &le; cost x<sub>i</sub>。

    又因为x<sub>i</sub> &larr; MinWith cost xs<sub>i</sub>

    所以x<sub>k</sub> &larr; MinWith cost (concat xss)

- 上面两个综合，我们有MinWith cost (map (MinWith cost) xss) = MinWith cost (concat xss)。

    也就是说我们从集合包含的角度证明了分布律

- 如何证明 foldr gstep c<sub>0</sub> xs &larr; MinWith cost (foldr fstep [c<sub>0</sub>] xs)?

    induction part:
    ```haskell
    foldr gstep c0 (x : xs)
    = gstep x (foldr gstep c0 xs)  -- foldr 定义
    <- gstep x (MinWith cost (foldr fstep [c0] xs)) -- induction
    <- MinWith cost (fstep x (foldr fstep [c0] xs)) -- fusion law
    = MinWith cost (foldr fstep [c0] (x : xs))
    ```

- 综上

    我们可以得到

    mcc xs &larr; MinWith cost (candidates xs)

- 所以，当我们从equality reasoning 做不了的时候，可以尝试从refinedment reasoning角度入手。


---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}

    