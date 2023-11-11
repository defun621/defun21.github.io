---
layout: post
title:  基于condition的CSMP[∅]的early stopping 算法中的概念
date:   2023-11-10 22:45:00 -0800
categories: [Fault-Tolerant-Message-Passing-Distributed-Systems]
---

教科书链接：<https://link.springer.com/book/10.1007/978-3-319-94141-7>

- I[1..n]代表每个process p<sub>i</sub> propose 一个值I[i]

- 基于condition的CSMP[∅]的early stopping 算法思想：

    **是否存在方法可以描述一组输入向量，使得无论故障模式如何，进程始终可以在不到（t + 1）轮内做出决策？**

- 一个condition就是一组input vector。

- 不妨令C[x], 0 &le; x &le; t，是一组/类（class）condition，该组condition可以满足一下条件：

    在最多f<sub>t</sub>(x)轮里可以完成consensus，其中f<sub>t</sub>(x) &le; t + 1 并且 f<sub>t</sub>(x + 1) &lt; f<sub>t</sub>(x)

    x 被叫做该类condition的degree。

- 一些准备概念：

    - V： 可以被proposed 的值的集合

    - equal(a, I): 在input vector I中值a出现的次数

    - dist(I<sub>1</sub>, I<sub>2</sub>)：I<sub>1</sub>和I<sub>2</sub>之间的Hamming distance

- **legality**

    一个condition C 如果满足条件：

    存在映射 h: C &#x21A6; V ，并且该映射满足条件：

    - &forall; I &isin; C: #<sub>h(I)</sub>(I) &gt; x

    - &forall; I1, I2 &isin; C: (h(I1) &ne; h(I2)) &rArr; (dist(I1, I2) &gt; x)

    则我们说condition C是x-legal。

    即函数h从input vector I中取出一个值，作为decided value。函数h代表了consensus。

    此时我们要求h满足一些条件：

    1. h(I)得到的值v，要在原来的input vector中出现足够多次（即多余x次）

    2. 如果h从不同的input vector I1, I2得到不同的值时，这两个input vector必须区别足够大。这是为了防止不同的process对于同一input vector观测到不同view时得出不同decided value。

- **所有x-legal的condition构成的类别即为C[x]**

- 假设V中的值可以有一个全序关系。

    - C<sub>max</sub><sup>x</sup>表示对于给定的x，h函数总是会取proposed值中最大的那个的condition。

    - max(I)代表input vector I中的最大值

    则C<sub>max</sub><sup>x</sup>可以按照下面方式被定义：

    C<sub>max</sub><sup>x</sup> &#x225D; {I \| equal(a, I) &gt; x where a = max(I)}

- 可以证明：**C<sub>max</sub><sup>x</sup>是一个x-legal condition**

- 同理 **C<sub>min</sub><sup>x</sup>也是一个x-legal condition**

- **一个x-legal condition C如果满足以下条件，则称C为maximal**:

    如果往C中加入一个input vector，则C不再是x-legal。

- C<sub>first</sub><sup>x</sup>

    它是取input vector 中最frequent值为decided value的condition。

    **C<sub>first</sub><sup>x</sup>也是x-legal的**。

- C<sub>first</sub><sup>x</sup>不是maximal的，但是C<sub>min</sub><sup>x</sup>和C<sub>max</sub><sup>x</sup>是maximal。

---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}