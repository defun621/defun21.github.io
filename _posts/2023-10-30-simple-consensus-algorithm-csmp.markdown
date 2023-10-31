---
layout: post
title:  CSMP[∅]中的consensus algorithm
date:   2023-10-30 22:45:00 -0700
categories: [Fault-Tolerant-Message-Passing-Distributed-Systems]
---

教科书链接：<https://link.springer.com/book/10.1007/978-3-319-94141-7>

- CSMP<sub>n,t</sub>[&empty;]中consensus 算法实现一：

    - 思想：因为最多t个process会挂，所以任取一个大小为t+1的process集合，集合里总能找到一个process可以decide出一个value。

    - propose实现简述：

        p<sub>i</sub>初始化本地变量est<sub>i</sub>为v<sub>i</sub>。

        然后所有process 同步执行t + 1轮如下操作，每一轮中有一个process作为coordinator。

        不妨令第r轮的coordinator是p<sub>r</sub>。

        如果在第r轮中，如果i == r，即自身是coordinator的话， 广播消息 EST(est<sub>i</sub>)。

        如果在第r轮中，拿到了从coordinator发过来的EST消息，则把消息里的值赋予est<sub>i</sub>。

        t + 1轮结束之后，p<sub>i</sub>返回本地的est<sub>i</sub>为decide的值。

    - 从代码中我们可以直接看出上述实现满足CC-validity 和CC-termination。

        因为执行了t + 1轮， 所以总有一轮里面的coordinator是正常的process。
        
        broadcast后，没挂的process都会update 由该coordinator发出来的值。

        所以能得到CC-agreement。

    - 该实现是不公平的：

        因为v<sub>j</sub>，其中j &isin; {t + 1, ..., n}，根本没有机会参与到决策中。

        可以在所有轮之前加一轮随机shuffle process的propose value，这可以使得该算法公平。

- CSMP<sub>n,t</sub>[&empty;]中consensus 算法实现二：

    - 思想：

        每个process存下每轮看到的value。在最后一轮之后，根据某个rule来返回最后decide了的value。

    - propose实现简述：

        p<sub>i</sub>有两个本地变量：est<sub>i</sub>和prev_est<sub>i</sub>。

        est<sub>i</sub>初始化为v<sub>i</sub>. prev_est<sub>i</sub>初始化为⊥。

        算法一共t + 1轮，在每一轮中：

        如果p<sub>i</sub>在上一轮中看到了更小的值（即est<sub>i</sub> &ne; prev_est<sub>i</sub>)，p<sub>i</sub>广播 EST(est<sub>i</sub>)。

        不妨令recval<sub>i</sub>是p<sub>i</sub>在某轮中收到的所有values组成的集合。

        先更新prev_est<sub>i</sub>为est<sub>i</sub>。

        然后更新est<sub>i</sub> 为 集合revcals<sub>i</sub> &cup; {est<sub>i</sub>} 中的最小值。

        t + 1轮之后，返回est<sub>i</sub>

    - 可以得到，上述算法也满足CC-validity、CC-termination和CC-agreement。

    - 该实现是公平的

- 基于实现二，我们可以得到CSMP<sub>n,t</sub>[&empty;]里vector consensus的算法

    - propose实现简述：

        p<sub>i</sub>有如下本地变量：

        view<sub>i</sub>是一个array，其中view<sub>i</sub>[k]表示p<sub>i</sub>知道的p<sub>k</sub> propose的值。

        也就是说如果view<sub>i</sub>[k] = ⊥，则表示p<sub>i</sub>还不知道p<sub>k</sub> propose的值。

        如果view<sub>i</sub>[k] = v，表示p<sub>i</sub>已经知道p<sub>k</sub> propose了v。

        算法中用两元组<k, v>表示p<sub>k</sub> propose了值v。

        在某轮结束之后，本地变量new<sub>i</sub>表示p<sub>i</sub>在本轮新了解到的值。

        一开始new<sub>i</sub>初始化为{<i, v<sub>i</sub>>}。

        算法一共t + 1轮，每一轮总是先send，后receive，然后再计算：

        发的时候，在每一轮中，如果new<sub>i</sub> 有值，则p<sub>i</sub>广播消息EST(new<sub>i</sub>)。

        收的时候，把从p<sub>j</sub>那里收到的new<sub>j</sub>赋予recfrom<sub>i</sub>[j]，如果没有从p<sub>j</sub>那里收到EST消息，recfrom<sub>i</sub>[j]置为⊥。

        计算时，对于每个有值的recfrom<sub>i</sub>[j]，对于该值集合中的每个二元组<k, v>，如果view<sub>i</sub>[k]是空，

        则更新view<sub>i</sub>[k] &larr; v，并且更新new<sub>i</sub> &larr; new<sub>i</sub> &cup; {<k, v>}。

        t + 1轮之后，返回view<sub>i</sub>

    - 可以证明上述算法实现满足ICC-validity、ICC-termination和ICC-agreement。


- 可以证明 t + 1是在CSMP<sub>n,t</sub>[&empty;]里的consensus算法的最低轮数。





---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}