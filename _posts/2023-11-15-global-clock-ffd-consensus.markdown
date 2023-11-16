---
layout: post
title:  CSMP中基于global clock和fast failure detector的consensus算法
date:   2023-11-15 20:45:00 -0800
categories: [Fault-Tolerant-Message-Passing-Distributed-Systems]
---

教科书链接：<https://link.springer.com/book/10.1007/978-3-319-94141-7>


- 假定每一轮的时长bounded by D.

- Fast perfect failure detector （FFD）

    - FFD告诉p<sub>i</sub> 一个集合suspected<sub>i</sub>。

    - FFD满足以下性质：

        - **strong accuracy**： 没有一个process在挂掉以前会进入别的process的suspected集合中

        - **detection timeliness**：如果process p<sub>j</sub>在&tau;时刻挂掉，则从&tau;+d时刻起，每个其他没有挂掉的process都会suspect p<sub>j</sub>。

            其中d << D，我们称其为maximal detection time

- 如果CSMP<sub>n,t</sub>[&empty;]里

    - 加上global clock，我们称其为CLOCK，算法开始时CLOCK为0

    - 并且加上Fast perfect failure detector，

    该系统模型是CSMP<sub>n,t</sub>[CLOCK, FFD]。

    在CSMP<sub>n,t</sub>[CLOCK, FFD]上存在consensus 算法。

- 算法实现简述：

    - 两种round

        - rounds of duration D time units：同步系统模型中的round

        - rounds of duration d： FFD rounds。

    - 中心思想：

        每个FFD round 有一个coordinator，在该FFD round里只有它可以发消息。

    - 实现简述：

        在FFD round (i - 1)d时，p<sub>i</sub>当且仅当它之前的所有process都在suspected集合中时才会发送消息(est<sub>i</sub>, i).

        当process<sub>i</sub>收到消息(est<sub>j</sub>, j)时，把该元组加入到view<sub>i</sub>中

        在t*d + D时刻，所有活着的process可以从view中所有的二元组（v,k)中选出k最大的对应的二元组中的v，返回该值。

    - 值得注意的是：

        **该算法是unfair的**，因为模型最多t个process挂掉，所以t + 1后面的process 没有机会广播自己的值。

    - 可以证明：该简单算法可以在 t &times; d + D时间内实现consensus。


- 在上述算法的基础上还可以使其early stopping。

    - 在(i-1)d时刻，p<sub>i</sub>还是按照上面算法一样，如果满足要求则广播(est<sub>i</sub>, i)。

    - 当p<sub>i</sub>收到消息，如果(est<sub>j</sub>,j)大于本地max_id<sub>i</sub>，则将est<sub>j</sub>作为要返回的值。

    - 每个(k - 1)d + D的时刻，如果k 不在suspected<sub>i</sub>中，则可以early stop，直接返回est<sub>i</sub>

    - 可以证明：该算法可以在d × f + D 时间内decide。




---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}