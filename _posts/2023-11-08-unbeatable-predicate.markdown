---
layout: post
title:  其他的early decision predicates
date:   2023-11-08 10:45:00 -0800
categories: [Fault-Tolerant-Message-Passing-Distributed-Systems]
---

教科书链接：<https://link.springer.com/book/10.1007/978-3-319-94141-7>

- knowledge based unbeatable predicate 目标：

    使得processes能尽快decide一个preferred的值。（在value是binary的情况下，我们认为该preferred的value是0）。

- 直觉上，我们有：

    - process p<sub>i</sub>可以直接decide 0，只要它知道其他每个正确运行的process已经获知preferred value 0已经被propose。

    - process p<sub>i</sub>可以直接decide 1，只要它知道没有一个活着的process收到过包含0已经被某个process propose的消息。

- 定义：

    - process p<sub>j</sub> 在第r轮被 **revealed** 给 process p<sub>i</sub>是指：

        - 要么p<sub>i</sub>在第r轮开始时已经知道p<sub>j</sub>已经知道的所有values

        - 要么p<sub>i</sub>知道p<sub>j</sub>在第r轮前已经挂掉

    - 某轮 r 被 **revealed** 给 process p<sub>i</sub> 是指 每个process p<sub>j</sub> 在第r轮都被revealed给p<sub>i</sub>。 
    
        这意味着p<sub>i</sub>在第r轮开始时已经知道了所有的value。

- **PREF0**

    PREF0 是0值的early decision predicate。它的定义是：

    PREF0 &#x225D; correct0(i,r) &or; revealed0(i,r)

    其中，

    - correct0(i,r)代表predicate：p<sub>i</sub>知道至少有一个没挂的process知道在第r轮0被propose了
    
    - revealed0(i,r)代表predicate：存在某一轮r'，r' &le; r，该轮r'被revealed给p<sub>i</sub>

- 可以证明：**PREF0强于DIFF**

    也就是说：

    - 给定某个execution，不妨令r为使得PREF0(i,r)为true的第一轮。则有DIFF(i,r) &rArr; PREF0(i,r)

    - 对于某个execution，不妨令r是使得DIFF(i,r)为真的第一轮。则存在某个failure pattern使得 PREF0(i,r) &and; &not;DIFF(i,r)

- 算法CGM

    算法CGM是基于PREF0的可以early decision的binary consensus算法

- 算法CGM实现中p<sub>i</sub>本地变量

    - vals<sub>i</sub>：存有p<sub>i</sub>已知的被proposed的值

    - knew0<sub>i</sub>：如果在前一轮结束时，0 &isin; vals<sub>i</sub>，则knew0<sub>i</sub>为true

    - correct0<sub>i</sub>：表示在当前轮r，correct0(i,r)是否被满足

    - revealed<sub>i</sub>：表示在当前轮r，revealed(i,r)是否被满足

    - lg<sub>i</sub>：一个本地维护的有向图。在该有向图中，节点是pair <process id, round number>。

        算法开始时，图只有节点<i, 0>。在每轮中，算法会根据收到的信息添加边和节点。

        如果有一条边从<j, r>到<k, r+1>，说明p<sub>i</sub>知道了p<sub>k</sub>在第r+1轮收到了从p<sub>j</sub>广播的消息。

- 算法CGM第一部分负责commuticate和本地状态的更新

    每个同步轮开始时，

    算法先广播本地状态，即由vals<sub>i</sub>和lg<sub>i</sub>构成的二元组。

    如果此时发现early<sub>i</sub>已经为true，则p<sub>i</sub>可以直接early decide 值0

    如果early<sub>i</sub> 为false并且 0 &isin; vals<sub>i</sub>，则置knew0<sub>i</sub>为true。

    然后p<sub>i</sub>根据本轮r中收到的消息来更新本地状态：

    - vals<sub>i</sub> 更新为 &cup;(vals<sub>j</sub>)

    - 令n0<sub>i</sub> 表示本轮中收到的满足0 &isin; vals<sub>j</sub>的消息的个数

    - 令nf<sub>i</sub> 表示本轮中p<sub>i</sub>没有从对方收到消息的process的个数

    - 更新lg<sub>i</sub>，按照上述规则添加边和节点

- 算法CGM的第二部分试着decide出一个值：

    - 先计算correct0(i, r):

        - 如果 0 &isin; vals<sub>i</sub> 并且 knew0<sub>i</sub>为true，这表示p<sub>i</sub>已经知道所有没挂掉的process已经有0被propose的知识。

            此时置correct0(i,r)为true

        - 如果0 &isin; vals<sub>i</sub>并且knew0<sub>i</sub>为false，这表示p<sub>i</sub>在第r轮才知道0被propose。

            如果t - nf<sub>i</sub> &le; n0<sub>i</sub>，则置correct0(i,r)为true。

            因为此时起码n - nf<sub>i</sub> + 1个process（包括p<sub>i</sub>自己）知道0被propose。

            因为最多有t个process会挂掉，而 n - nf<sub>i</sub> + 1 > t，所以至少有一个正确的process知道0被propose。

    - 再计算 revealed(i,r)：即

        &exist; r' &lt; r: &forall; p<sub>j</sub>: 
        
        - <j,r'> &isin; vertices(lg<sub>i</sub>) 或者
        
        - &exist; <l,r'> &isin; vertices(lg<sub>i</sub>) : (<j,r'-1>, <l,r'>) &notin; edges(lg<sub>i</sub>)

    - 最后p<sub>i</sub>根据correct0(i,r)和revealed(i,r)来看是否能early stopping。

        - 如果correct0(i,r)为true，则decide 0

        - 如果correct0(i,r)不满足，0 &notin; vals<sub>i</sub>，但revealed(i,r)为true，则可以decide 1

        - 如果correct0(i,r)不满足，revealed(i,r)为true，并且 0 &isin; vals<sub>i</sub>，p<sub>i</sub>置early<sub>i</sub>为true。


- 可以证明：**CGM算法在CSMP<sub>n,t</sub>[&empty;]实现了binary consensus。并且证明一个process最多执行min(f+2,t+1)轮。**



---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}