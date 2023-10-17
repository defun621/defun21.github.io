---
layout: post
title:  CAMP[t < n/2]上的scd广播（set-constrained delivery broadcast)
date:   2023-10-16 14:45:00 -0700
categories: [Fault-Tolerant-Message-Passing-Distributed-Systems]
---

教科书链接：<https://link.springer.com/book/10.1007/978-3-319-94141-7>

- scd-broadcast和scd-deliver

    - scd-broadcast 以一条message为参数，进行广播

    - scd-deliver操作返回**一个message 集合**

- **scd-broadcast性质**

    - validity：如果一个process scd-deliver的message集合里包括m，则m是由某个process scd-broadcast的。

    - integrity：一条message m最多被每个process scd-deliver一次。

    - MS-ordering：
    
        不妨令p<sub>i</sub>先scd-deliver了集合ms<sub>i</sub>，再scd-deliver了集合ms'<sub>i</sub>。

        对于任意一对message m &isin; ms<sub>i</sub>, m' &isin; ms'<sub>i</sub>，

        不存在这样的p<sub>j</sub>会先scd-deliver包含m'的集合ms'<sub>j</sub>，再scd-deliver包含m的集合ms<sub>j</sub>。

    - Termination-1：如果一个没挂掉的process scd-broadcast了一条message m，该process会结束该scd-broadcast操作，并且会scd-deliver一个包含m的消息集合

    - Termination-2：如果一个process scd-deliver了消息m，每个没有挂掉的process都会scd-deliver包含m的消息集合。

- 注意MS-ordering性质

    - 任意两个process scd-deliver的消息集合里的东西不是相互独立的。

    - 如果令scd-deliver的消息集合大小为1，并且没有MS-order性质，这就是URB-broadcast

    - 满足SCD顺序的例子：满足SCD顺序的例子：

        - p1: {m1, m2}, {m3, m4, m5}, {m6}, {m7, m8}. 
        - p2: {m1}, {m3, m2}, {m6, m4, m5}, {m7}, {m8}.

    - 不满足SCD顺序的例子：

        - p1: {m1, m2}, {m3, m4, m5}
        - p2: {m1, m3}, {m2}

- containment性质

    由MS-ordering性质和Termination-2性质，我们可以得到

    &forall; i,j,x,y: (MS<sub>i</sub><sup>x</sup> &sube; MS<sub>j</sub><sup>y</sup>) &or; (MS<sub>j</sub><sup>y</sup> &sube; MS<sub>i</sub><sup>x</sup>)

    其中不妨令

    - ms<sub>i</sub><sup>l</sup>表示由p<sub>i</sub> scd-deliver的第l个消息集合

    - 则p<sub>i</sub>scd-deliver的消息序列即为ms<sub>i</sub><sup>1</sup>， ms<sub>i</sub><sup>2</sup>，……， ms<sub>i</sub><sup>x</sup>，

    - 则MS<sub>i</sub><sup>x</sup> = ms<sub>i</sub><sup>1</sup> &cup; ms<sub>i</sub><sup>2</sup> ... &cup; ms<sub>i</sub><sup>x</sup>

- 消息上的partial order关系

    由MS-ordering性质和integrity性质我们可以得到一个消息上的偏序关系。

    不妨令p<sub>i</sub>先scd-deliver了包含m的消息集合，再scd-deliver了包含m'的消息集合，

    则m和m'上由偏序关系：m &#x21A6;<sub>i</sub> m'， 下标i表示这个关系是表示是p<sub>i</sub>局部的。

    同时 &#x21A6; = &cup;<sub>1&le;i&le;n</sub>&#x21A6;<sub>i</sub>构成了一个消息上的total-order。

- **在CAMP<sub>n,t</sub>[t &lt; n/2]上SCD广播实现简述**

    - 该实现基于FIFO-broadcast

    - 每个process p<sub>i</sub>维护以下本地变量：

        - buffer<sub>i</sub>： 
        
            一个初始为空的buffer，buffer里面存有四元组qdplt = <qdplt.msg, qdplt.sd, qdplt.sn, qdplt.cl>，这些四元组里的msg都是已经被FIFO-delivered但属于一个还没被scd-deliver的消息集合。

            qdplt里

            - msg是消息m

            - sd是发消息的process id

            - sn是消息发送方发送该条消息的sequence number或local date。所以<qdplt.sn, qdplt.sd>构成了逻辑时间戳

            - cl是一个长度为n的array，array里的每个元素初始为无穷大。cl[x]里存放p<sub>x</sub>广播FORWARD消息时对应于消息m的sequence number。


        - to_deliver<sub>i</sub>

            一组qdplt集合，每个qdplt里包含将要被scd-deliver的消息

        - sn<sub>i</sub>

            本地逻辑时钟，初始值为0。每次操作自增1。每个由p<sub>i</sub> scd-broadcast的消息的id就是<i, sn<sub>i</sub>>

        - clock<sub>i</sub>[1...n]

            由逻辑时间戳构成的数组。clock<sub>i</sub>[j]表示由p<sub>i</sub> scd-deliver的id为<j, x>的消息的中的时间戳x的最大值

        - p<sub>i</sub>scd广播消息m时：

            构造FORWARD消息 FORWARD(m, sn<sub>i</sub>, i, sn<sub>i</sub>, i)。
            
            第一对pair sn<sub>i</sub>, i表示消息的identifier。
            
            第二对pair sn<sub>i</sub>, i表示消息的时间戳。

            把FORWARD发给自己

            等到：条件buffer<sub>i</sub>中没有sender是自身的四元组满足后，调用结束。

        - 当p<sub>i</sub> FIFO-deliver消息FORWARD(m, sd, sn<sub>sd</sub>, f, sn<sub>f</sub>)之后，

            调用函数forward(m, sd, sn<sub>sd</sub>, f, sn<sub>f</sub>)

            再调用函数try_deliver

        - 函数forward(m, sd, sn<sub>sd</sub>, f, sn<sub>f</sub>)：

            - 如果sn<sub>sd</sub> &le; clock<sub>i</sub>[sd]，说明p<sub>i</sub>已经scd-deliver了消息

            - 否则

                - 如果buffer<sub>i</sub>里没有包含消息m的四元组qdplt（即qdplt.sd = sd并且qdplt.sn = sn<sub>sd</sub>)，

                    初始化一个新的qdplt，（cl[f] &larr; sn<sub>f</sub>)。

                    把这个新的qdplt加到buffer<sub>i</sub>中去

                    fifo-broadcast 消息forward(m, sd, sn<sub>sd</sub>, i, sn<sub>i</sub>)

                    sn<sub>i</sub>自增1。

                - 否则更新qdplt.cl[f] &larr; sn<sub>f</sub>。
                
                    表明已经知道有消息m并且它被p<sub>f</sub>在它本地时间sn<sub>f</sub>转发过。

        - 函数try_deliver():

            p<sub>i</sub>会首先计算buffer<sub>i</sub>中有多少消息已经被大多数process见过（即该qdplt中的cl数组中有超过半数的元素不是+&infin;)。

            为了保证MS-ordering性质，p<sub>i</sub>需要从to-deliver<sub>i</sub>中将满足如下条件的qdplt删除：

            &exist; qdplt &isin; to_deliver<sub>i</sub>, &exist; qdplt' &isin; buffer<sub>i</sub> \ to_deliver<sub>i</sub>: \|{f: qdplt.cl[f] &lt; qdplt'.cl[f]}\| &le; n/2

            如果剔除完之后to_deliver<sub>i</sub>不为空：

            - 则遍历每个qdplt，更新clock<sub>i</sub>[qdplt.sd] &larr; max(clock<sub>i</sub>[qdplt.sd], qdplt.sn).

                把to_deliver<sub>i</sub>从buffer<sub>i</sub>里移除，

                把to_deliver<sub>i</sub>里的消息拿出来构造集合ms，

                最后SCD-deliver(ms)。



---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}