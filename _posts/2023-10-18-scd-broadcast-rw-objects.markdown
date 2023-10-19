---
layout: post
title:  用scd-broadcast实现一些read/write object
date:   2023-10-16 14:45:00 -0700
categories: [Fault-Tolerant-Message-Passing-Distributed-Systems]
---

教科书链接：<https://link.springer.com/book/10.1007/978-3-319-94141-7>

- 可以用scd-broadcast来实现concurrent object。实现的基础框架为：

    - 操作op：

        done<sub>i</sub>初始化为false，scd广播消息TYPE(a,b,...,i)，等done<sub>i</sub>被置为true。

        以上三步根据实现的object不同，可能会被重复1到2次。

        最后用本地变量计算出最终结果r并返回。

    - 当要scd-deliver 消息集合{MSG(...,j<sub>1</sub>), ..., MSG(...,j<sub>x</sub>), SYNC(j<sub>x+1</sub>), ..., SYNC(j<sub>y</sub>)}时，

        对于每条MSG(...,j)消息，根据不同object更新本地状态

        如果p<sub>i</sub> 发现scd-deliver了自己broadcast的消息时（&exist; l: j<sub>l</sub> = i)，把done<sub>i</sub>置为true，使得op可以返回。

- 在CAMP<sub>n,t</sub>[SCD-broadcast]里实现MWMR寄存器

    - **实现MWMR的atomic寄存器**

        - 本地变量

            - done<sub>i</sub>

            - reg<sub>i</sub>，对应atomic寄存器REG的本地值

            - tsa<sub>i</sub>，现在在reg<sub>i</sub>中的值写入时的时间戳。该时间戳是一个逻辑时间戳，<sn, x>。

        - **REG.read()**

            done<sub>i</sub>置为false，scd-broadcast消息SYNC(i)，等到done<sub>i</sub>被置为true时，返回reg<sub>i</sub>内的值。

            消息SYNC(i)用来告诉其他process，p<sub>i</sub>要刷新本地的reg<sub>i</sub>了。

        - **REG.write()**

            done<sub>i</sub>置为false，scd-broadcast消息 SYNC(i)，等到done<sub>i</sub>被置为true执行下一步。

            这一步的目的是re-synchronization。给p<sub>i</sub>一个最新的tsa<sub>i</sub>。

            再把done<sub>i</sub>置为false，scd-broadcast消息 WRITE(v <tsa<sub>i</sub> + 1, i>)，等到done<sub>i</sub>被置为true之后，写操作结束。

        - **当要scd-deliver消息集合 {WRITE(v<sub>j<sub>1</sub></sub>, <date<sub>j<sub>1</sub></sub>, j<sub>1</sub>>), ..., WRITE(v<sub>j<sub>x</sub></sub>, <date<sub>j<sub>x</sub></sub>, j<sub>x</sub>>), SYNC(j<sub>x+1</sub>), ..., SYNC(j<sub>y</sub>)}时**：

            不妨令WRITE(v,<d,w>)是收到的所有WRITE消息里时间戳最大的那一条消息。

            如果tsa<sub>i</sub>小于d，则说明有更新，p<sub>i</sub>进行更新：reg<sub>i</sub> &larr; v, tsa<sub>i</sub> &larr; <d,w>

            更新结束后，如果发现有msg是从自己scd-broadcast出来的，则把done<sub>i</sub>置为true。

        - 为什么该实现是一个MWMR的atomic寄存器：

            Lemma21证明过程略。思路是构造一个在operation上的total order。leamma 21可以REG是linearizable。-->safety

            通过上述算法描述可以简单证明如果一个正常的process 发起一个operation，则该operation可以正常结束 --> liveness

    + **实现MWMR的sequential consistent寄存器**

        实现和上面的MWMR的atomic寄存器基本一样，只需要把发SYNC消息的部分移除即可。

        因为SYNC起到的作用是让所有的读写操作好像依照真实时间中的顺序。而sequential consistent寄存器并没有这个要求

- 在CAMP<sub>n,t</sub>[SCD-broadcast]里实现atomic snapshot object

    - **Atomic MWMR snapshot object**

        - 是一个REG[1...m]数组，每个REG[i]都是一个atomic寄存器。

        - 提供两个操作：
        
            - write(r,v)： 把v值赋给REG[r]

            - snapshot()：返回REG[1...m]的在该时刻的snapshot。因为是atomic的，所以满足atomic的定义：如果snapshot和多个写concurrent，这么多写中的任意一个。

    - 实现简述：

        - snapshot实现和atomic MWMR寄存器里的read类似。区别在于snapshot返回时返回所有本地reg[1...m]。

        - write实现和atomic MWMR寄存器的write类似。区别在于snapshot object的write消息多了一个field，目标寄存器r

        - 当消息集合要被scd-deliver时的实现也和atomic MWMR寄存器里的类似。唯一区别在于现在更新时需要更新m个reg<sub>i</sub>本地变量，也同时需要更新m个时间戳tsa<sub>i</sub>。



---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}