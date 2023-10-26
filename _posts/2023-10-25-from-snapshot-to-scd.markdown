---
layout: post
title:  用SWMR snapshot objects实现scd-broadcast
date:   2023-10-25 22:45:00 -0700
categories: [Fault-Tolerant-Message-Passing-Distributed-Systems]
---

教科书链接：<https://link.springer.com/book/10.1007/978-3-319-94141-7>

- **实现简述**

    - shared object

        - SWMR snapshot object：SENT[1..n]。

            初始为[&empty;,...,&empty;]。SENT[i]表示p<sub>i</sub> scd广播出去的message。

        - SWMR snapshot object: SETS_SEQ[1,...,n].

            初始为[&epsilon;,...,&epsilon;]。SETS_SEQ[i]表示p<sub>i</sub> scd-deliver过的消息集合的sequence。

    - 本地变量

        - sent<sub>i</sub>。SENT在p<sub>i</sub>的本地copy

        - sets_seq<sub>i</sub>。SETS_SEQ在p<sub>i</sub>的本地copy

        - to_deliver<sub>i</sub>。用来存放p<sub>i</sub>要scd-deliver的下个消息集合。

    - 算法简述

        - 每个p<sub>i</sub>都有一个后台线程T一直循环执行方法progress。保证每条message 都会被scd-deliver。

        - p<sub>i</sub>调用scd-broadcast(m):

            把m加入到本地sent<sub>i</sub>中。p<sub>i</sub>往SENT[i]里写sent<sub>i</sub>。

            然后调用progress方法。

        - progress方法：

            因为scd-broadcast和线程T都会调用progress方法、access本地变量，所以progress第一步就是调用互斥锁。

            调用catchup方法。目的是为了让p<sub>i</sub>可以scd-deliver那些已经被scd-broadcast但还没本地scd-deliver的消息。

            等从catchup方法退出之后，开始scd-deliver那些别的process scd-broadcast的消息。

            p<sub>i</sub>首先从SENT读snapshot出来，然后存到sent<sub>i</sub>。

            然后计算要scd-deliver的消息集合，放入to_deliver<sub>i</sub>.
            to_deliver<sub>i</sub> &larr; &cup;<sub>1 &le; j &le; n</sub>sent<sub>i</sub>[j] \ member(sets_seq<sub>i</sub>[i])。

            如果to_deliver<sub>i</sub>不为空，说明有消息要deliver。

            这时，把to_deliver<sub>i</sub>加入到sets_seq<sub>i</sub>[i]中，接着往SETS_SEQ里写sets_seq<sub>i</sub>[i]。写完后scd-broadcast to_deliver<sub>i</sub>.

            最后退出互斥锁。

        - catchup方法：

            p<sub>i</sub>先从SETS_SEQ里读一个snapshot，存入sets_seq<sub>i</sub>.

            然后对每个包含了没有被p<sub>i</sub> scd-deliver过的消息的message set，如果有这样的process p<sub>j</sub> , 即
            &exist; j, set: set = sets_seq<sub>i</sub>[j].head: set &#x2288; member(sets_seq<sub>i</sub>[i])，

            p<sub>i</sub>把set里没发过的消息（不在member(sets_seq<sub>i</sub>[i])都加到to_deliver<sub>i</sub>里。

            把to_deliver<sub>i</sub>加入到sets_seq<sub>i</sub>[i]中，接着往SETS_SEQ里写sets_seq<sub>i</sub>[i]。写完后scd-broadcast to_deliver<sub>i</sub>.

            结束循环后退出catchup方法。

- 可以证明上述实现可以保证
    
    - validity
    - integrity
    - termination-1 和termination-2
    - MS-ordering

    即，保证了scd broadcast的性质。（证明略，看书）

- 那既然scd-broadcast可以实现snapshot，snapshot也可以实现scd broadcast，也就是说**scd broadcast 和 snapshot在computability power上是等价的**


---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}