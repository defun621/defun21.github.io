---
layout: post
title:  在CAMP[t < n/2]中实现MWMR的sequentially consistent寄存器
date:   2023-10-09 21:45:00 -0700
categories: [Fault-Tolerant-Message-Passing-Distributed-Systems]
---

教科书链接：<https://link.springer.com/book/10.1007/978-3-319-94141-7>

- 之前提到过**sequentially consistency不是composable的**。所以实现composite sequentially consistent寄存器的一种可能是系统提供额外的条件。

    本章中，作者介绍了两种：

    - 加上TO-broadcast
    - logic time

- **TO-broadcast**回顾

    Total order broadcast的性质满足

    - TO-validity：如果一个process TO-deliver一条msg，则msg之前被TO-broadcast过。

    - TO-integrity： 一个processTO-deliver某条msg最多1次。

    - TO-order：如果一个process在TO-deliver m1之前TO-deliver了m2，所有process都要在TO-deliver m1之前TO-deliver m2.

    - TO-causal precedence：如果m &rarr;<sub>M</sub> m' (表示m在因果关系上是m'的前驱)，则每个process都要先TO-deliver m，再TO-deliver m'

    - TO-termination: 如果一个没挂的process TO-broadcast 了m，或者一个process TO-deliver了m，则所有没挂的process都TO-deliver了m。

    可以看到TO-broadcast的性质就是Causal order broadcast 性质加上了TO-order性质。

    而且：

    - 每个没挂掉的process都TO-deliver相同的message 序列

    - 每个挂掉的process都TO-deliver上述序列的某个前缀

- **CAMP<sub>n,t</sub>[t &lt; n/2, TO-broadcast]中的MWMR sequentially consistent寄存器实现简述**

    - **with fast read operation**

        - 任意寄存器X读操作的时候，p<sub>i</sub>直接返回本地的值x<sub>i</sub>

        - 任意寄存器X写操作的时候，先TO-broadcast 消息 SEQ_CONS(i, X, v) （意思是p<sub>i</sub>往寄存器X里写了值v），然后等到消息TO-deliver到自己之后，再返回。

        - 任意process p<sub>i</sub> TO-deliver任意消息 SEQ_CONS(j,X,v)时

            把本地的x<sub>i</sub>置为v。然后如果i == j，告诉自己可以从写操作返回了。

    - **with fast write operation**

        每个process除了有对应于每个register的本地变量外还有一个计数器nb_write<sub>i</sub>，用来标记从自己这里TO-broadcast但还没有TO-deliver的消息数。

        - 任意寄存器X读操作时：得等到nb_write<sub>i</sub>归零之后才能返回x<sub>i</sub>

        - 任意寄存器X写操作时：nb_write<sub>i</sub>自增1，广播消息SEQ_CONS(i, X, v)。结束操作。

        - 任意process p<sub>i</sub> TO-deliver任意消息 SEQ_CONS(j,X,v)时：

            把本地x<sub>i</sub>置为v，如果该消息是由自己TO-broadcast(i == j)的，则nb_write<sub>i</sub>自减1


- **基于logic clock的MWMR sequentially consistent寄存器实现简述**

    - 每个process p<sub>i</sub>的本地变量

        - lc<sub>i</sub>，p<sub>i</sub>的本地logic clock。当p<sub>i</sub>上发生读或写时，自增1.

        - reqsn<sub>i</sub>，p<sub>i</sub>上的每次操作所对应的sequence number

        - done<sub>i</sub>。每次p<sub>i</sub>由新的操作时被置为false，当该操作满足条件可以终止时，被置为true

        - rr<sub>i</sub>，系统假定每个register都有index。rr<sub>i</sub>表示p<sub>i</sub>上一次读的寄存器所对应的index。

        - reg<sub>i</sub>[X]，p<sub>i</sub>中对应的寄存器X的本地变量

        - tst<sub>i</sub>[X]，用来记录p<sub>i</sub>中当reg<sub>i</sub>[X]被更新时的逻辑时间

        - val<sub>i</sub>，p<sub>i</sub>中当前读操作要返回的值

    - p<sub>i</sub>有X.write(v)时：

        先初始：reqsn<sub>i</sub>自增1，lc<sub>i</sub>自增1。done<sub>i</sub>设为false。
        
        然后构造逻辑时间戳ts: <lc<sub>i</sub>, i>，广播消息WRITE(reqsn<sub>i</sub>, lc<sub>i</sub>, X, ts, v)。

        一直等到done<sub>i</sub>被设为true之后，操作结束。

    - p<sub>i</sub>收到从p<sub>j</sub>来的WRITE(reqsn, lc, Y, ts, v)时：

        先调整lc<sub>i</sub>，将其设为lc<sub>i</sub>与lc 取大加1。

        然后比较消息里的时间戳ts与tst<sub>i</sub>[Y]，如果ts &lt; tst<sub>i</sub>[Y]，则更新tst<sub>i</sub>[Y]与reg<sub>i</sub>[Y]

        最后向p<sub>j</sub>发送ACK_WRITE(reqsn, lc<sub>i</sub>)

    - 当p<sub>i</sub>收到来自p<sub>j</sub>的ACK_WRITE(reqsn, lc)时：

        如果reqsn == reqsn<sub>i</sub>，则调整lc<sub>i</sub>为lc<sub>i</sub>与lc 取大加1。否则discard该条消息

        一直到从大多数process那里收到reqsn<sub>i</sub>相应的ACK_WRITE时，done<sub>i</sub>设为true，并且reqsn<sub>i</sub>自增1.

    - p<sub>i</sub>有X.read()时：

        先初始：reqsn<sub>i</sub>自增1，lc<sub>i</sub>自增1。done<sub>i</sub>设为false。

        rr<sub>i</sub>置为X，广播READ_REQ(reqsn<sub>i</sub>, lc<sub>i</sub>, X)。

        等到done<sub>i</sub>值为true时，返回val<sub>i</sub>。

    - p<sub>i</sub>收到从p<sub>j</sub>来的READ_REQ(reqsn, lc, Y)时：

        调整lc<sub>i</sub>为lc<sub>i</sub>与lc 取大加1

        给p<sub>j</sub>发送ACK_READ_REQ(rsn, lc<sub>i</sub>, reg<sub>i</sub>[Y], tst<sub>i</sub>[Y])

    - p<sub>i</sub>收到ACK_READ_REQ(rsn, lc, v, tst)时：

        如果reqsn == reqsn<sub>i</sub>，则调整lc<sub>i</sub>为lc<sub>i</sub>与lc 取大加1。否则discard该条消息
        
        如果已从大多数process中收到reqsn<sub>i</sub>对应的ACK_READ_REQ，不妨令这些ACK_READ_REQ消息中tst最大的那条里的tst为mst，v为val。val<sub>i</sub>的值置为val。

        则reqsn<sub>i</sub>自增1，把reg<sub>i</sub>[rr<sub>i</sub>]设为val<sub>i</sub>，tst<sub>i</sub>[rr<sub>i</sub>]设为mst。

        然后广播WRITE(reqsn<sub>i</sub>, lc<sub>i</sub>, rr<sub>i</sub>, val<sub>i</sub>, mst).

    - **可以观察到，该基于logic clock的MWMR sequentially consistent寄存器实现里的读操作有两阶段，和ABD算法一样；而读操作只有一阶段**




---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}