---
layout: post
title:  其他的early decision predicates
date:   2023-11-06 21:45:00 -0700
categories: [Fault-Tolerant-Message-Passing-Distributed-Systems]
---

教科书链接：<https://link.springer.com/book/10.1007/978-3-319-94141-7>

- 之前的early stopping 算法中用到的predicate是：

    *nbr<sub>i</sub>[r] - nbr<sub>i</sub>[r - 1] = 0*

    我们这里称之为DIFF(i, r)

- 不妨令 faulty<sub>i</sub>[r] = n - nbr<sub>i</sub>[r]，即在第r轮，p<sub>i</sub>认为挂掉的process的数量。

- 则 COUNT(i,r) &equiv; (faulty<sub>i</sub>[r] &le; r) 也可以是一个正确的early decision predicate

    因为：

    当COUNT(i,r)被满足时，现在的轮数大于p<sub>i</sub>挂掉的process的个数，
    
    也即在p<sub>i</sub>看来，存在这么一轮r'，1 &le; r' &le; r，在该轮中没有process 挂掉。

    那在该r'轮结束时，p<sub>i</sub>里的view<sub>i</sub>已经有当时能够知道的所有<k,v>元组并且由于不存在recover，以后也不会知道更多的<k,v>.

- DIFF(i,r) 和 COUNT(i,r) 孰优孰劣？

    可以证明：

    - 给定一个execution，不妨令 r &ge; 2是满足COUNT(i,r)的第一轮。则存在COUNT(i,r) &rArr; DIFF(i,r)

    - 给定一个execution，不妨令 r &ge; 2是满足DIFF(i,r)的第一轮。则存在failure pattern，使得DIFF(i,r) &and; &not; COUNT(i,r)。

- 第一条是因为当COUNT(i,r)满足时，也就是说foulty<sub>i</sub>[r] &lt; r 并且 faulty<sub>i</sub>[r - 1] &ge; r - 1。

    则有faulty<sub>i</sub>[r] - faulty<sub>i</sub>[r - 1] &lt; 1

    也即nbr<sub>i</sub>[r - 1] - nbr<sub>i</sub>[r] &lt; 1

    又因为nbr<sub>i</sub>[r - 1] - nbr<sub>i</sub>[r] &ge; 0

    所以 nbr<sub>i</sub>[r - 1] - nbr<sub>i</sub>[r] = 0。

    也就是说COUNT(i,r) &rArr; DIFF(i,r)

- 第二条可以反证：

    不妨令在某次execution中一开始就有2 &le; x &le; t个process 挂掉，之后没有一个process挂掉。

    则COUNT(i,r)在第x + 1轮变为真。

    而从第2轮起，nbr<sub>i</sub>[r - 1]和nbr<sub>i</sub>[r]之间的差即为0.

    所以对于2到x中的任意轮，DIFF(i,r) &and; &not; COUNT(i,r)

- 上面两个定理说明：**DIFF(i,r)优于COUNT(i,r)**。

    因为存在某些failure pattern使得用DIFF(i,r)的算法可以在min(f + 2, t + 1)轮之前就终止。

    其根本原因是DIFF(i,r)可以把本轮中挂掉的process数也加入到考量中。

- 从CSMP<sub>n,t</sub>[&empty;]上的early stopping的interactive consistency算法可以很直接得到early stopping 的consensus算法。

    因为是consensus，所以不需要view，只需要一个est.

    此处略过不表。






---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}