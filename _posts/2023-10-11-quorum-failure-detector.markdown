---
layout: post
title:  Quorum failure detector &Sigma;
date:   2023-10-11 22:45:00 -0700
categories: [Fault-Tolerant-Message-Passing-Distributed-Systems]
---

教科书链接：<https://link.springer.com/book/10.1007/978-3-319-94141-7>

- **quorum failure detector class &Sigma;**

    &Sigma;给每个process<sub>i</sub>，提供一个sigma<sub>i</sub>

    集合{sigma<sub>i</sub>}<sub>i &le; n</sub>一起使得&Sigma;满足以下性质：

    - **Intersection**: &forall; i,j &isin; {1,...,n}: &forall; &tau;,&tau;' &isin; N: sigma<sub>i</sub><sup>&tau;</sup> &cap; sigma<sub>j</sub><sup>&tau;'</sup> &ne; &empty;

    - **Liveness**: &exist; &tau; &isin; N: &forall; &tau;' &ge; &tau;: &forall; i &isin; Correct(F): sigma<sub>i</sub><sup>&tau;'</sup> &sube; Correct(F).

    其中F表示某次execution对应的failure pattern；
    Correct(F)表示在该failure pattern里没有挂掉的process集合；
    sigma<sub>i</sub><sup>&tau;</sup>表示在&tau;时刻p<sub>i</sub>读到的sigma<sub>i</sub>的值，也就是H(p<sub>i</sub>, &tau;)

    **Intersection**性质表明在任何时刻，任意两个process读到的sigma值总有交集 （可以prevent cluster partitioning）

    **Liveness**性质表明一段时间之后，没挂掉的process读到的sigma里只有正在运行的process

- 在CAMP<sub>n,t</sub>[t&le;n/2]中&Sigma;实现简述

    - 初始化： 每个process都有一个queue，里面随机排列所有n个process。

    - 每个process都广播alive消息

    - 当p<sub>i</sub>从p<sub>j</sub>那里收到alive消息时：
        
        把j放到队列头，把队列里前&#x2308;(n+1)/2&#x2309;个process形成的set赋值给sigma<sub>i</sub>

    **注意到，该实现与在CAMP<sub>n,t</sub>[t &le; n/2]中的failure detector &Theta;的实现一样**

- **在CAMP<sub>n,t</sub>[&Sigma;]中SWSR的atomic寄存器的实现简述**

    不妨令在SWSR的atomic寄存器的实现中，写的那个process是p<sub>w</sub>，读的那个process是p<sub>r</sub>

    为了实现atomic寄存器，每个process都会参与，每个process p<sub>i</sub>都有一个REG的本地副本reg<sub>i</sub>，还有一个本地变量wsn<sub>i</sub>，目的和之前SWMR的regular寄存器实现中一样。

    - 当p<sub>w</sub>调用*REG.write(v)*时：

        wsn<sub>w</sub> 自增1，广播WRITE(v, wsn<sub>w</sub>)

        然后等到在从sigma<sub>w</sub>中的每个process都收到ACK_WRITE(wsn<sub>w</sub>)之后，*REG.write(v)*完成

    - 当p<sub>r</sub>调用*REG.read()*时：

        reqsn<sub>r</sub> 自增1，广播READ_REQ(reqsn<sub>r</sub>)。

        然后等到在从sigma<sub>r</sub>中的每个process那里都收到ACK_READ_REQ(rsn, wsn, v)之后，

        从这些ACK_READ_REQ里挑取wsn最大的那条，不妨令wsn最大的那条ACK_READ_REQ里的v值为x，

        则如果wsn<sub>r</sub>小于wsn，则把wsn赋值给wsn<sub>r</sub>，v赋值给reg<sub>r</sub>。

        最后返回reg<sub>r</sub>

    - 当任意process p<sub>i</sub>收到WRITE(wsn, v)时：

        如果wsn大于本地的wsn<sub>i</sub>，则把wsn赋值给wsn<sub>i</sub>，v赋值给reg<sub>i</sub>。

        无论wsn如何，总是回复ACK_WRITE(wsn)给p<sub>w</sub>

    - 当任意process p<sub>i</sub>收到READ_REQ(reqsn)时：

        回复ACK_READ_REQ(reqsn, wsn<sub>i</sub>, reg<sub>i</sub>)给p<sub>r</sub>






---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}