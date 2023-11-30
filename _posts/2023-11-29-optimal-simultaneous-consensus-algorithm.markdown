---
layout: post
title:  CSMP下最优simultaneous consensus算法实现简述
date:   2023-11-29 20:45:00 -0800
categories: [Fault-Tolerant-Message-Passing-Distributed-Systems]
---

教科书链接：<https://link.springer.com/book/10.1007/978-3-319-94141-7>


- 每个process p<sub>i</sub>本地变量：

    - est<sub>i</sub>，存着p<sub>i</sub>在r轮结束时认为的可能的decision value。初始化为v<sub>i</sub>

    - f<sub>i</sub>[r]，表示p<sub>i</sub>在r轮没有从其收到message的process集合。

        则<text style="text-decoration:overline">f<sub>i</sub>[r]</text> = &Pi; \ f<sub>i</sub>[r] 则表示f<sub>i</sub>[r]的补集，也就是收到从其发来消息的process的集合。

    - f'<sub>i</sub>[r-1]，是一个p<sub>i</sub>在第r轮计算出的一个值。

        它等于&cup;<sub>p<sub>j</sub>&isin;<text style="text-decoration:overline">f<sub>i</sub>[r]</text></sub>f<sub>j</sub>[r-1]

        从它的定义我们可以看到，它就是那些被p<sub>i</sub>在第r轮收到从其发来的消息的中至少一个认为到r-1轮为止挂掉的process。

        因为p<sub>i</sub>也会对自身发消息，所以有 f<sub>i</sub>[r-1] &sube; f'<sub>i</sub>[r-1]

    - bh<sub>i</sub>[r]，存的是p<sub>i</sub>在r轮中得出的最小的horizon value。按照定义bh<sub>i</sub>[0] = t + 1


- 算法实现简述：

    初始化est<sub>i</sub>, bh<sub>i</sub> 以及f<sub>i</sub>[0]（置为&empty;).

    在每一轮synchronous 轮中：

    p<sub>i</sub>首先广播消息EST(est<sub>i</sub>，f<sub>i</sub>[r-1])。

    然后根据收到的消息计算f'<sub>i<sub>[r-1]。

    收发消息阶段结束后，我们就可以得到f<sub>i</sub>[r]。

    根据本轮收到的所有消息中的est，p<sub>i</sub>取最小值赋予est<sub>i</sub>。

    然后计算h<sub>i</sub>[r], 它的值为(r-1)+(t+1-\|f'<sub>i</sub>[r-1]\|)，并且更新bh<sub>i</sub>[r]，取h<sub>i</sub>[r]与上一轮的bh值中较小者。

    最后如果r就是bh<sub>i</sub>[r]，那说明我们已经得到了最小的horizon。

    算法终止。

- 可以证明，上述算法满足所有SC性质。





---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}