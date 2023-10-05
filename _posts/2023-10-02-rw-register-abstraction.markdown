---
layout: post
title:  读写寄存器抽象
date:   2023-10-02 12:45:00 -0700
categories: [Fault-Tolerant-Message-Passing-Distributed-Systems]
---

教科书链接：<https://link.springer.com/book/10.1007/978-3-319-94141-7>

- **concurrent object**: 

    能同时被多个sequential process访问的object

- **register**:

    （shared register object，in short， register）寄存器R是一种抽象，可以代表一个字节，或者一个shared disk。

    R提供两种操作：R.read()和R.write()。

    第五章主要定义以下两类register：

    - regular read/write register

    - atomic register

- **regular R/W register**

    regular R/W register是一种（SWMR）寄存器（单写多读）。

    regular寄存器读操作返回的值定义为：

    - 一个读如果不和任何写是concurrent关系, 则读需要读到寄存器当前的内容(即, 上一个成功完成的写入的值)

    - 一个读如果和多个写操作是concurrent关系, 那么这个读可以返回这多个写的值里的任意一个, 或者寄存器在这些写发生前的值

- **new/old inversion**

    register上读到的值序列不一定是写入时候序列，也就是说可以先读到一个后被写入的值再读到一个老的值。

- **atomic register**

    atomic register与regular register不同，它可以是MWMR，在atomic register上不会出现new/old inversion。

    atomic寄存器读操作返回的值定义为：

    - 所有的读和写操作看起来就像它们按照顺序执行一样，这个序列不妨称其为*S*

    - 如果op<sub>1</sub>在op<sub>2</sub>开始前结束，则op<sub>1</sub>在S序列里出现在op<sub>2</sub>之前。也即S遵守时间序。

    - atomic寄存器上每个读操作返回在S序列上在他之前最近的写操作写入的值

    *S*被称为寄存器执行的线性化。（一个register execution可以存在多个linearization）。

- **sequentially consistency register**

    它是atomic register的一种弱化形式。它满足一下性质：

    - 所有的读和写操作看起来就像它们按照顺序执行一样，这个序列不妨称其为*S*

    - 如果process p<sub>i</sub>先执行op<sub>1</sub>再执行op<sub>2</sub>，则在序列S中op<sub>1</sub>必须出现在op<sub>2</sub>之前。也即S遵守进程顺序关系。

    - 每个读操作返回在S序列上在他之前最近的写操作写入的值

- **注意：atomic寄存器要求遵守全局时间序，而sequentially consistency register只要求操作需要遵守进程顺序关系**

- 如何形式化定义**Atomicity**和**Sequentially consistency**

    - 预备概念：

        - **R.op(arg)(res)**，表示寄存器R上参数为arg，返回值为res的操作op，可以是写也可以是读

        - 每个**R.op(arg)(res)**有两个相应的**event**，invocation *inv[R.op(arg) by p<sub>i</sub>]* 和reply *resp[R.op(res) by p<sub>i</sub>]*.

        - **History**:

            - H&#770; = (H, &lt;<sub>H</sub>)，其中H是所有process发生的event所构成的集合，&lt;<sub>H</sub>是在这些events上的全序关系。

            - **local history**，H&#770;\|p<sub>i</sub>，是只有p<sub>i</sub>发起的event构成的H&#770;的一个子序列。

            - 如果 &forall; p<sub>i</sub>: H&#770;\|p<sub>i</sub> = H&#770;'\|p<sub>i</sub>，则H&#770;和H&#770;'是等价的。

            - 本文中只考虑well-formed history（即invocation后面是reply，然后再接invocation……）

        - **operation上的偏序关系**

            （op &rarr;<sub>H</sub> op') &#x225D; (resp[op] &lt;<sub>H</sub> inv[op'])

            如果既没有（op &rarr;<sub>H</sub> op') 也没有（op' &rarr;<sub>H</sub> op) ，则op与op' overlap。

        - **sequential history**

            当H&#770;的第一个事件是一个invocation，并且
            1. 每个invocation之后紧跟着它对应的reply；
            2. 每个reply之后跟着一个invocation；则H&#770;是一个sequential history

        - **concurrent history**

            即一个非sequential的history。

        - **S&#770;\|R**，即在sequential history *S&#770;* 上，只由寄存器R相关的事件构成的子序列。

        - **legal history**: 在**S&#770;\|R**，每个read都只返回它之前最近的write的值。

    - **Atomicity**: 一个history H&#770;是Atomic的条件是

        - 存在这么一个由H&#770;里的事件的排列构成的序列S&#770，即H&#770;和S&#770;等价
        - S&#770;是sequential并且legal
        - &rarr;<sub>H</sub> &sube; &rarr;<sub>S</sub>

        S&#770;就是H&#770;的一个线性化。

    - **Sequentially consistency**

        一个history H&#770;是Sequentially consistency的条件是

        - 存在这么一个由H&#770;里的事件的排列构成的序列S&#770;，即H&#770;和S&#770;等价
        - S&#770;是sequential并且legal

- 一些重要结论：

    - **atomicity是composable的而Sequentially consistency不是**
    
    - **在CAMP<sub>n,t</sub>[t &ge; n/2]里构造不出atomic寄存器**
    
    - **在CAMP<sub>n,t</sub>[t &ge; n/2]里构造不出两个或更多Sequentially consistency寄存器**

---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}