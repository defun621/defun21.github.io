---
layout: post
title:  在CAMP[t < n/2]中实现SWMR以及MWMR的atomic寄存器
date:   2023-10-06 14:45:00 -0700
categories: [Fault-Tolerant-Message-Passing-Distributed-Systems]
---

- **Regular register不能保证atomicity**

    比如下面这个例子里，红线表示某次读在各个process的时间点。read1比read2早发生，但read1可以读到新的值，该新值的sequence number是15。而read2能只能读到老的值。
    这就发生了new/old inversion.

    ![例子](/assets/images/regularity_not_atomicity.png)

- 如何防止new/old inversion

    光是在读的时候让process从大多数process中收ACK_READ_REQ是不够的。还需要额外的步骤。

    假设<v, wsn>是p<sub>i</sub>从ACK_READ_REQ里收到的所有元组里sequence number最高的一个元组。
    
    为了达到atomicity，我们还需要让p<sub>i</sub>在发起一次write请求。

    这保证了当读操作完成时，别的大多数process在本地也能有至少像那样“新”的值。

- **CAMP<sub>n,t</sub>[t &lt; n/2]中SWMR的atomic寄存器实现简述**

    - *REG.read()*

        reqsn<sub>i</sub>加1，广播READ_REQ(reqsn<sub>i</sub>)。

        等从大多数process中收到ACK_READ_REQ(reqsn<sub>i</sub>, _, _)，从中选取最大的(v, wsn)。

        广播WRITE(v, wsn)。

        等从大多数process中收到ACK_WRITE(wsn)之后才返回值v

    - *REG.write()* 和SWMR的regular寄存器一样

    - 收到*WRITE(v, wsn)*消息时，和SWMR的regular寄存器的实现大致一样。
    
        唯一要注意的区别
        
        - 在于SWMR的regular寄存器只有一个p<sub>w</sub>在广播WRITE消息；
        
        - 而在CAMP<sub>n,t</sub>[t &lt; n/2]中SWMR的atomic寄存器实现中，每个process都会广播WRITE消息。

    - 收到READ_REQ(reqsn)消息时，和SWMR的regular寄存器的实现一样


- 在CAMP<sub>n,t</sub>[t &lt; n/2]上，从SWMR的atomic寄存器到MWMR的atomic寄存器 （**ABD算法**）

    - **timestamp**，一个timestamp是一个元组<sn, i>，其中sn是逻辑时戳，i是process identity。

    - timestamp上可以定义一个**全序关系**：<sn1, i> &lt; <sn2, j> &equiv; ((sn1 &lt; sn2) &#x22C1; (sn1 = sn2 &#x22C0; i &lt; j))

    - **ABD算法实现简述**
    
        - 相比SWMR的atomic寄存器，每个process需要维护的新的本地变量

            -lw：用来记录哪个process发出的写操作最近写入了本地reg<sub>i</sub>中的值

        - *REG.write(v)*时：

            reqsn<sub>i</sub>自增1，
            
            然后process<sub>i</sub>会尝试拿到系统当前状态：

                - 先广播WRITE_REQ(reqsn<sub>i</sub>)
                
                - 等从大多数process处收到ACK_WRITE_REQ(reqsn<sub>i</sub>, msn)之后，找到最大的msn。

            然后更新全局系统状态：
                
                - 广播WRITE(reqsn<sub>i</sub>, v, msn + 1, i)

                - 等从大多数process处收到ACK_WRITE(reqsn<sub>i</sub>)

            最后返回

        - *REG.read()*时：

            reqsn<sub>i</sub>自增1，

            然后process<sub>i</sub>尝试拿到全局系统当前状态：

                - 先广播READ_REQ(reqsn<sub>i</sub>)

                - 等从大多数process处收到ACK_READ_REQ(reqsn<sub>i</sub>, msn, mlw, v)之后，找到拥有最大的timestamp<msn, mlw>的那个消息里的值v

            - 更新全局系统状态

                - 广播WRITE(reqsn<sub>i</sub>, v, msn, mlw)

                - 等从大多数process处收到ACK_WRITE(reqsn<sub>i</sub>)

            最后返回v

        - 收到WRITE(reqsn, v, sn, lw)时：

            如果(sn, lw) &ge; (wsn<sub>i</sub>, lw<sub>i</sub>)，则更新reg<sub>i</sub>，wsn<sub>i</sub>，lw<sub>i</sub>

        - 收到READ_REQ(reqsn)时

            回复ACK_READ_REQ(rsn, wsn<sub>i</sub>, lw<sub>i</sub>, reg<sub>i</sub>)给发送方

        - 收到WRITE_REQ(reqsn)时

            回复ACK_WRITE_REQ(rsn, wsn<sub>i</sub>)给发送方

- 可以看到ABD算法在write和read的时候都分为两步：先尝试获取系统状态，在更新系统状态。拥有这样的算法structure的算法，叫做*two-phase algorithms*。

---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}