---
layout: post
title:  在CAMP[-FC]中的Quiescent可靠广播
date:   2023-09-29 22:59:00 -0700
categories: [Fault-Tolerant-Message-Passing-Distributed-Systems]
---

教科书链接：<https://link.springer.com/book/10.1007/978-3-319-94141-7>

- 引入failure detector 的意义

    t &lt; N/2这个假设很强, 也很不灵活, 而如果能提供必要的信息供系统使用, 就可以隔离关于系统模型的假设； 对于failure detector来说, 算法只要依赖于failure detector提供的关于failure的信息就可以解决问题的话, 对于任意failure detector的实现, 只要能满足它定义的属性, 它们实现的细节是分布式算法不用关心的. 即分布式算法只依赖于这些属性, 而不依赖于具体实现和具体环境的细节. 这就使得算法和系统设计更加模块化。

    同时, 使用failure detector可以可以使我们直接研究一个系统需要对 failure 的观察"准确和全面"到何等程度, "最少"需要多少对于failure的信息, 一个分布式系统的问题才能被解决. 这是学术上的failure detector的意义所在。

- failure detector 正式定义

    - **Failure pattern**数学上，Failure pattern是一个映射*F : N -> 2<sup>&Pi;</sup>*，其中
    
      - N是自然数集（代表时间
        
      - 2<sup>&Pi;</sup>代表所有process的幂集
      
      那么，*F(&tau;)* 就是在&tau;时刻，CAMP模型中，挂掉process的集合。

    - **Faulty(F)** 和 **Non-Faulty(F)**：表示在某次execution中，按照Failure pattern *F*挂掉的process的集合。
        
        则有*Faulty(F) = F(&tau;<sub>max</sub>)*，&tau;<sub>max</sub>代表了该次执行的结束时间。

        显然，*Faulty(F) = &cup;<sub>1 &le; &tau; &lt; +&infin;</sub>F(&tau;) = lim<sub>&tau; &rarr; +&infin;</sub>F(&tau;)*

        而*Non-Faulty(F) = &Pi; - Faulty(F)*，Non-Faulty(F)在本书中也被称为Correct(F)

    - **Failure detector history with range R**：*R*表示failure detector提供给process的某类信息。

        *failure detector history*是一个函数*H: &Pi; &times; N &rarr; R*，
        
        *H(p<sub>i</sub>, &tau;)* 代表了failure detector在&tau;时刻给p<sub>i</sub>的信息。

    - **Failure detector class FD with range R**: FD是一个将Failure pattern F映射到某个failure detector history with range R集合上的函数。

- CAMP<sub>n,t</sub>[-FC, &Theta;]上的URB

    - failure detector *&Theta;* 的定义

        - **accuracy**：&forall; i &isin; &Pi;: &forall; &tau; &isin; N: (trusted<sub>i</sub><sup>&tau;</sup> &cap; Correct(F)) &ne; &empty;
        
          这代表着在任何时间，任何trusted<sub>i</sub>至少包含一个正常运行的process。

        - **Liveness**: &exist; &tau; &isin; N: &forall; &tau;' &ge; &tau;: &forall; i &isin; Correct(F): trusted<sub>i</sub><sup>&tau;'</sup> &sube; Correct(F)

          这表示在某个时间点之后，任何正常运行的process上的trusted集合里只包括正常运行的process。

    - **CAMP<sub>n,t</sub>[-FC, t &lt; n/2]上构造Failure detector &Theta;** 实现简述

        - 初始化：构造队列queue，队列包含乱序的所有集合，trusted集合中可以包含任意&#x2308;(n+1)/2&#x2309;个process。

        - 运行时：
            - 广播alive消息
            - 当收到从别的process过来的alive消息时，把pk移到队列queue头，把队列中前 &#x2308;(n+1)/2&#x2309; 个process加入到trusted。

    - 在 **CAMP<sub>n,t</sub>[-FC, &Theta;]** 上的URB实现简述

        和之前 **CAMP<sub>n,t</sub>[-FC]** 的URB一样，唯一的区别在于urb-deliver的条件

        - **CAMP<sub>n,t</sub>[-FC]** 上的条件是 rec_by[m]中process的数量大于等于t + 1

        - **CAMP<sub>n,t</sub>[-FC, &Theta;]** 上的条件变成 trusted<sub>i</sub> &sube; rec_by[m]

    - 引入failure detector &Theta;的**意义**：

        有了 **&Theta;**， 即使是t &ge; n/2，也可以实现URB。

- quiescent uniform reliable broadcast

    之前所有的在CAMP[-FC]上的URB实现中，Diffuse(m)都需要无限broadcast消息。
    
    原因就是在于无法区分一个process是死了, 还是消息丢失了, 所以只要有一个process没有ack之前就死了, 其他process就需要一直向其发送msg, 永不停止。

    - **quiescent 属性**

        是指算法为了发送一条msg只需要有限次消息传递即可。

- 3种达成Quiescent的方法

    - 基于**perfect failure detector P**的URB

        perfect failure detector P维护的集合是suspected<sub>i</sub>,其中包含了p<sub>i</sub>认为已经挂掉的process。

        - **perfect failure detector P** 定义

            - **Completeness**: &forall; &tau; &isin; N: &forall; &tau;' &ge; &tau;: &forall; i &isin; Correct(F), &forall; j &isin; Faulty(F): j &isin; suspected<sub>i</sub><sup>&tau;'</sup>

                这表示总有某个时刻&tau;，在其之后，所有的挂掉的process都已经进到了每个正常的process的suspected集合里去。

            - **Strong accuracy**: &forall; &tau; &isin; N: &forall; i, j &isin; Alive(&tau;): j &notin; suspected<sub>i</sub>

                其中Alive(&tau;) = &Pi; \ F(&tau;)。

                这表示任何时候，活着的process都不会出现在suspected集合中。

        - 在 **CAMP<sub>n,t</sub>[-FC, &Theta;, P]** 上的URB实现简述

            P提供suspected, 则&Theta; 的trusted即为 &Pi; \ suspected.

            - **broadcast**：发送给包括自己在内的所有process

            - **收到从p<sub>k</sub>发来的msg时**：

                - 如果是第一次收到：初始化rec_by[m]，启动Diffuse(m) 任务
                - 否则更新rec_by[m]

                然后发送ack(m)给p<sub>k</sub>。

            - **收到p<sub>k</sub>发来的ack(m)时**，更新rec_by[m]。

            - **Diffuse(m)**: 直到 rec_by<sub>i</sub>[m] &cup; suspected<sub>i</sub> = &Pi; 为止都执行循环：对于不在rec_by且不在suspected中的每个process，发送m。

            - **urb-deliver条件**：
            
                同 **CAMP<sub>n,t</sub>[-FC, &Theta;]** 上的URB：

                (trusted<sub>i</sub> &sube; rec_by<sub>i</sub>[m]) &and; (p<sub>i</sub> has not yet urb-delivered m)

        - **注意：因为可以完美检测错误, 那么被检测到crash的不用再发信息就好了, 而没crash的好的process一定会最终ack, 所以不但可以实现Quiescent, 算法也可以终止**

    - 基于**eventually perfect failure detector &#x25C7;P**的URB

        - **定义**

            - **completeness**：&exist; &tau; &isin; N: &forall; &tau;' &ge; &tau;: &forall; i &isin; Correct(F), &forall; j &isin; Faulty(F): j &isin; suspected<sub>i</sub><sup>&tau;'</sup>

                这和perfect failure detector P一样。

            - **Eventual strong accuracy**：&exist; &tau; &isin; N: &forall; &tau;' &ge; &tau;: &forall; i, j &isin; Alive(&tau;'): j &notin; suspected<sub>i</sub><sup>&tau;'</sup>

                注意这里和和perfect failure detector P的区别，这里的条件更弱，只需要某个时刻之后就可以。

        - **在CAMP<sub>n,t</sub>[-FC, &Theta;, &#x25C7;P]上URB**的实现简述：

            与 **CAMP<sub>n,t</sub>[-FC, &Theta;, P]上的URB** 实现唯一不同的在于Diffuse(m)的循环终止条件： rec_by<sub>i</sub>[m] = &Pi;

        - **注意：算法无法停止(需要不断观察&#x25C7;P的检测结果变化), 但是至少可以达成Quiescent效果(由于可以检测到crash, 所以不会向一个已经crash的process发无限的msg)**

    - 基于**heartbeat failure detector HB**的URB

        - **heartbeat failure detector HB 定义**

            failure detector HB维护一个数据结构HB<sub>i</sub>[1,...,n]

            - **Completeness**： &forall; i &isin; Correct(F), &forall; j &isin; Faulty(F): &exist; K: &forall; &tau; &isin; N: HB<sub>i</sub><sup>&tau;</sup>[j] < K

                这表示p<sub>i</sub>的HB计数器中对应挂掉的process的值不会增长

            - **Liveness**

                1. &forall; i, j &isin; &Pi; &forall; &tau; &isin; N: HB<sub>i</sub><sup>&tau;</sup>[j] &le; HB<sub>i</sub><sup>&tau;+1</sup>[j]
                2. &forall; i,j &isin; Correct(F): &forall; K: &exist; &tau; &isin; N: HB<sub>i</sub><sup>&tau;</sup>[j] > K

                这两条表明HB单调递增，且如果p<sub>i</sub>,p<sub>j</sub>都正常，它的值HB<sub>i</sub>[j]就会一直增长。

        - **在CAMP<sub>n,t</sub>[-FC, &Theta;, HB]上URB**的实现简述：

            - **broadcast**：把msg发送给包括自己在内的所有process

            - **收到从p<sub>k</sub>发来的msg时**：

                - 如果是第一次收到：初始化并更新rec_by[m], prev_hb[m]和cur_hb[m]，启动任务Diffuse(m)
                - 否则就更新rec_by[m]

                最后向p<sub>k</sub>发送ack

            - **收到p<sub>k</sub>发来的ack(m)时**，更新rec_by[m]

            - **Diffuse(m)**: 直到 rec_by<sub>i</sub>[m] = &Pi; 为止都执行循环：将当前HB的值赋给cur_hb[m]，只对不在rec_by[m]并且prev_hb[m][j] &lt; cur_hb[m][j]的process<sub>j</sub>发送m，最后把cur_hb[m]赋给prev_hb[m]。

            - **urb-deliver条件**：(trusted<sub>i</sub> &sube; rec_by<sub>i</sub>[m]) &and; (p<sub>i</sub> has not yet urb-delivered m)

            - **注意:由于不知道什么时候有"die"的process会重新发来heartbeat, 所以算法无法终止, 但是真的crash了的process不会再发来heartbeat, 所以不会再向其发msg(可以达成Quiescent)**



---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}