---
layout: post
title:  consensus 的变体：simultaneous consensus
date:   2023-11-27 10:45:00 -0800
categories: [Fault-Tolerant-Message-Passing-Distributed-Systems]
---

教科书链接：<https://link.springer.com/book/10.1007/978-3-319-94141-7>

- 在CSMP<sub>n,t</sub>[&empty;]里，simultaneous consensus （SC）的目标是：

    - processes在同一轮中decide

    - 该轮越早越好
    
    - 无论有多少process crash


- SC要满足的性质有：

    - SC validity：decide出来的值是某个被proposed的值

    - SC data agreement：不存在两个decide出不同value的process

    - SC round agreement：不存在两个在不同轮中完成decide的process

    - SC termination：每个正确运行的process都可以decide出值

- 之前的non early deciding consensus算法可以使得正确运行的process在第t + 1轮完成decide。

    但是他的问题在于不够efficient

- **思路： 当y个process在轮r中挂掉，则剩余的process可以在该轮结束时了解到这y个process 已经挂掉，则可以省下y - 1轮**


- 一些定义：

    - failure pattern F：在SC的语境里，F是一个由最多t个三元组<j, k<sub>j</sub>, b<sub>j</sub>>构成的list。

        其中
        
        - j是process id，

        - k<sub>j</sub>是某轮

        - b<sub>j</sub>是一个process的集合。

        该三元组可以表示p<sub>j</sub>在k<sub>j</sub>轮挂掉，b<sub>j</sub>是在该轮中没有收到由p<sub>j</sub>发出的消息的process的集合。


    - Failure discovery：我们说p<sub>j</sub>的failure在第r轮发现当r是满足以下条件的首轮：

        有一个process p<sub>i</sub>

        - 没有在r轮中收到p<sub>j</sub>的消息

        - 并且在第r轮中没有挂掉


    - waste：

        - 不妨令C[r,F]（或者缩写成C[r]，因为failure pattern F是implicit）是在第r轮结束时由没有挂掉的process观察到的挂掉的process的数量

        - 对于任意轮r，不妨令d<sub>r</sub> = max(0, \|C[r]\|-r)。则d<sub>r</sub>代表着可以省下的轮数

        - 给定某个failure pattern，不妨令D<sub>r</sub> = max<sub>r&ge;0</sub>(d<sub>r</sub>)。这个值就对应了给定F的情况下，能省下的最多的轮数


    - clean round：
        
        如果在某轮r中没有process 被观察到首次为faulty，则该轮r被称为clean round。即C[r - 1] = C[r]

        如果令r是一个clean round，并且P是进到r+1轮时正确运行的process的集合。则在第r轮时P从某个集合Q中收到消息并且P&sube;Q。

    - horizon of a round：

        给定某个process p<sub>i</sub>和某个轮数r &ge; 1，令x是p<sub>i</sub>在第1轮和第r-1轮之间观察到的挂掉process的数量最大值（按照定义，当r为1时，x=1）。

        则我们称h<sub>i</sub>(r)=r + t - x为轮r时p<sub>i</sub>的horizon。按照定义有h<sub>i</sub>(1) = t + 1。

        有以下定理成立：

        **如果p<sub>i</sub>完成轮r并且可以知道上述x值，则存在一个clean round y使得r &le; y &le; h<sub>i</sub>(r)=r+t-x**


---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}