---
layout: post
title:  用scd-broadcast实现counter和lattice agreement
date:   2023-10-23 20:45:00 -0700
categories: [Fault-Tolerant-Message-Passing-Distributed-Systems]
---

教科书链接：<https://link.springer.com/book/10.1007/978-3-319-94141-7>

- **Counter object**

    一个concurrent counter object有三个操作：

    - increase
    - decrease
    - read

- **用scd-broadcast实现counter object**实现简述

    - increase：
        
        done<sub>i</sub>置为false。

        scd广播plus(i)消息，等到done<sub>i</sub>值被置为true时，increase 操作结束

    - decrease：

        和increase一样，区别在scd广播的是消息MINUS(i)

    - read：

        done<sub>i</sub>置为false。

        scd广播SYNC(i)消息，等到done<sub>i</sub>值被置为true时，返回本地counter<sub>i</sub>的值。

    - 当p<sub>i</sub>要scd-deliver消息集合 {PLUS(j<sub>1</sub>), ..., MINUS(j<sub>x</sub>), ..., SYNC(j<sub>y</sub>), ...}时：

        不妨令u是消息集合中PLUS的消息个数，v是消息集合中MINUS消息的个数，

        更新counter<sub>i</sub>为counter<sub>i</sub> + u - v

        如果在消息集合中有本地scd-broadcast的消息时，done<sub>i</sub>置为true。

- 可以证明：上述算法可以实现一个atomic counter。证明思路和之前用scd-broadcast实现atomic register类似。

- sequentially consistent counter

    - 上述实现只需要把done<sub>i</sub>改成一个本地计数器就可以实现一个sequentially consistent counter。

        在该实现中，increase和decrease在消息broadcast完后能马上返回，不需要等待某个值。

    - **实现简述**

        - increase

            本地lsc<sub>i</sub>自增1，广播PLUS(i)。结束

        - decrease

            和increase类似，区别在于scd广播的是MINUS(i)。

        - read：

            等到本地lsc<sub>i</sub>的值归0之后，返回counter<sub>i</sub>的值。

        - 当p<sub>i</sub>要scd-deliver消息集合 {PLUS(j<sub>1</sub>), ..., MINUS(j<sub>x</sub>), ...}时：

            不妨令u是消息集合中PLUS的消息个数，v是消息集合中MINUS消息的个数，w是集合里从本地发出的消息的个数。

            更新counter<sub>i</sub>为counter<sub>i</sub> + u - v。

            更新lsc<sub>i</sub>为lsc<sub>i</sub> - w.

- lattice agreement

    - **定义**

        不妨令S是一个集合，&le;是S上的偏序关系。

        给定S' &sube; S，**S'的上界**就是一个属于S的元素x，使得 &forall; y &isin; S': y &le; x

        **S'的最小上界z**，就是一个这么样的元素，使得对于所有S'的上界y都有 z &le; y。

        如果S的所有有限子集都有一个最小上界，则S被称做一个**semilattice**。


    - **lattice agreement task**

        - 定义：

            p<sub>i</sub>有操作propose。

            不妨令 p<sub>i</sub>调用了propose(in<sub>i</sub>)返回了属于semilattice S的元素z，其中in<sub>i</sub>也是semilattice S中的元素。

            我们称p<sub>i</sub> propose in<sub>i</sub>，decide z。

        - 性质：

            - LA-validity：
            
                如果p<sub>i</sub> decide了 out<sub>i</sub>，则有in<sub>i</sub> &le; out<sub>i</sub> &le; lub({in<sub>1</sub>, ..., in<sub>n</sub>}),

                其中lub(S)表示S的最小上界。

            - LA-containment:
            
                如果p<sub>i</sub> decide了out<sub>i</sub>， p<sub>j</sub> decide了 out<sub>j</sub>，

                则有out<sub>i</sub> &le; out<sub>j</sub> 或者 out<sub>j</sub> &le; out<sub>i</sub>

            - LA-termination:

                如果一个没有挂的process propose了一个值，则它会decide 一个值

- lattice agreement实现简述：

    - propose(in<sub>i</sub>)：

        done<sub>i</sub>置为false。广播消息MSG(i, in<sub>i</sub>)。等到done<sub>i</sub>为true时，返回本地lub(rec<sub>i</sub>)。

    - 当要scd-deliver消息集合{MSG(j<sub>1</sub>, v<sub>j<sub>1</sub></sub>), ..., MSG(j<sub>x</sub>, v<sub>j<sub>x</sub></sub>)}时：

        rec<sub>i</sub>更新为 rec<sub>i</sub> &cup; {v<sub>j<sub>1</sub></sub>, ..., v<sub>j<sub>x</sub></sub>}

        如果有从本地发出的MSG，则done<sub>i</sub>置为true。



---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}