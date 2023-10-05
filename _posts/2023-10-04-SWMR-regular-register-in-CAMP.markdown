---
layout: post
title:  在CAMP[t < n/2]中实现SWMR的regular寄存器
date:   2023-10-04 21:45:00 -0700
categories: [Fault-Tolerant-Message-Passing-Distributed-Systems]
---

教科书链接：<https://link.springer.com/book/10.1007/978-3-319-94141-7>

- 所有实现shared read/write register *REG*的在构成架构上都是相同的。组成部分包括：

    - n 个process，每个process相互之间发送、接收消息；并且每个process维护自己本地的数据结构

        本地的数据结构包括：

        - 一个本地的reg<sub>i</sub>，用来存放p<sub>i</sub>已知的上一次写入*REG*的值

        - 一组控制变量，用来保证p<sub>i</sub>调用*REG.read()*的时候能读到正确的值。

    - 每个process既担任client（实现 *REG.read()和REG.write()*的本地算法部分），又担任server（负责处理收到的消息）。

- 回顾regular寄存器的定义

    regular寄存器由以下性质定义：

    - **Safety**: 

        - 如果*REG.read()*执行完成并且该次读操作没有和任何写操作concurrent，则它返回最近被写入*REG*中的值。

        - 如果该次读操作与某些写操作concurrent，则它可以返回这些写操作要写入的值或者这些写操作之前最近被写入*REG*中的值。

        这两条在上一篇中有给出，这里回顾一下，并且告诉我们regular寄存器可能返回什么样的值。

    - **Liveness**

        无论操作是*REG.read()*还是*REG.write()*， 如果调用它的process p<sub>i</sub>没有挂，则该操作总能完成。

- 在CAMP<sub>n,t</sub>[n &lt; n/2]中实现一个SWMR regular寄存器

    - 实现思路简述（不妨令p<sub>w</sub>是那一个写的process）：

        一方面，p<sub>w</sub>给每个写操作都赋一个sequence number，并且广播该pair（新写入的值和它对应的sequence number）。

        另一方面，每个其他process p<sub>i</sub>都把sequence number最高的那个pair记在本地。

    - **local variables**: 每个p<sub>i</sub>都维护以下本地变量：

        - reg<sub>i</sub>，用来存p<sub>i</sub>已知的*REG*的值

        - wsn<sub>i</sub>，用来存reg<sub>i</sub>里的值对应的sequence number

        - reqsn<sub>i</sub>，用来存放p<sub>i</sub>最近一次读到*REG*值时对应的sequence number

    - 实现简述:

        - **REG.write(v)** (因为是SWMR，所以只有p<sub>w</sub>会调用*REG.write()*)

            wsn<sub>i</sub>加一，然后广播pair *<v, wsn>*，等到从多数process那里都收到*ack_write(wsn<sub>w</sub>)*的时候从方法中返回

        - **REG.read()**

            reqsn<sub>i</sub>加一，广播READ_REQ(reqsn<sub>i</sub>)。
            
            如果别的process p<sub>j</sub>收到READ_REQ(reqsn<sub>i</sub>)时，p<sub>j</sub>会返回ACK_READ_REQ(reqsn<sub>i</sub>, wsn<sub>j</sub>, reg<sub>j</sub>)给p<sub>i</sub>

            当p<sub>i</sub>从大多数process那里收到reqsn<sub>i</sub>对应的ACK_READ_REQ之后，他从中挑选wsn最大的那个值 *v* ，并将 *v* 值返回。

        - 当process<sub>i</sub> **收到write(val, wsn)** 时

            process<sub>i</sub>会对比wsn是否比本地wsn大，如果比本地wsn<sub>i</sub>的值大，则把val存进*reg<sub>i</sub>*中。

            然后总是发送ack_write(wsn)给p<sub>w</sub>。

    - **cost**

        一次read或者write，会产生2 &times; n 条message。

    - **注意：该实现假定process之间能互通，构成完全图，并且process之间channel时reliable**


- 如果write process 挂了：

    - 如果p<sub>w</sub>当没在写时挂掉，没影响，别的进程能读到最近写的值。

    - 如果p<sub>w</sub>在执行写操作时挂掉，可能部分process能读到新值，部分process读到老值。

- 如果read process挂掉：对其他process没影响。




---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}