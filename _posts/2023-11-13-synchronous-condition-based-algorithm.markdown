---
layout: post
title:  基于condition的CSMP[∅]的early stopping 算法实现简述
date:   2023-11-13 20:45:00 -0800
categories: [Fault-Tolerant-Message-Passing-Distributed-Systems]
---

教科书链接：<https://link.springer.com/book/10.1007/978-3-319-94141-7>

- 不妨令input vector I是属于x-legal condition的某个input vector。

    我们称J是I的一个view，（即J是一个和I一样的vector，但是J中最多有x个element是⊥）。

    我们可以把J当作某个process 通过收到信息所构造出的本地input vector。

    表示为**J &le; I**

- 可以通过反证法证明：

    **如果C是一个x-legal condition并且I<sub>1</sub>和I<sub>2</sub>是属于C的两个input vector。如果存在view J使得J &le; I<sub>1</sub>并且J &le; I<sub>2</sub>，则h(I<sub>1</sub>)=h(I<sub>2</sub>)**

- 基于上述定理，可以推断出：

    **如果I是一个属于x-legal condition C的input vector，并且J是I的一个view，则h(J) = h(I)**

- 基于condition的CSMP[∅]的early stopping 算法实现简述

    - 本地变量：

        - view<sub>i</sub>，input vector的本地已知值

        - v_cond<sub>i</sub>，用来保存h(I)(只要h(I)可以被知晓)

        - v_tmf<sub>i</sub>，用来保存某个值，当系统挂太多process时，不再适用函数h来decide value时，该值被作为decided value

    - 实现简介：

        process p<sub>i</sub>的行为按照所在轮数分为两部分：

        - 在第一轮中：

            process p<sub>i</sub>通过msg EST1(v<sub>i</sub>)广播它propose的值v<sub>i</sub>。

            然后在receive阶段构造view<sub>i</sub>。

            如果本轮结束时

            - equal(⊥, view<sub>i</sub>) &le; x，这说明p<sub>i</sub>已经有足够的信息，此时p<sub>i</sub>用h函数计算h(view<sub>i</sub>)，将结果存入v_cond<sub>i</sub>中。

            - 否则表明不适用函数h。p<sub>i</sub>计算view<sub>i</sub>中已知值的最大值，将其赋予v_tmf<sub>i</sub>.
        
            如果x = t，此时可以直接返回v_cond<sub>i</sub>。因为别的process已经都知道了信息了。

        - 在其他轮中：

            p<sub>i</sub>通过消息EST2(v_cond<sub>i</sub>, v_tmf<sub>i</sub>)来广播他的状态。

            广播完消息之后，
            
            - 如果v_cond<sub>i</sub>不为空，则直接返回v_cond<sub>i</sub>。

            - 如果某个v_cond<sub>j</sub>不为空，则置v_cond<sub>i</sub>为该值。

            - 取所有收到的v_tmf<sub>j</sub>的最大值，将其赋予v_tmf<sub>i</sub>.

            如果已经是执行了t + 1 - x 轮，

            则返回v_cond<sub>i</sub>若其不为空，否则返回v_tmf<sub>i</sub>。


- 可以证明：

    上述算法可以在CSMP<sub>n,t</sub>[&empty;]中实现consensus。并且每个process 都不超过t + 1 - x轮。

---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}