---
layout: post
title:  atomic寄存器与urb广播
date:   2023-10-13 14:45:00 -0700
categories: [Fault-Tolerant-Message-Passing-Distributed-Systems]
---

教科书链接：<https://link.springer.com/book/10.1007/978-3-319-94141-7>


- 如果D是一个failure detector class，并且在CAMP<sub>n,t</sub>[D]里可以构造出atomic register，不妨令A是该构造atomic register的算法，

    则总可以用A构造出算法，模拟出&Sigma;,使得每个process p<sub>i</sub>得到simga<sub>i</sub><sup>&tau;</sup>

    **结论：只要有failure detector class &Sigma;，我们就可以在一个任意process可能crash的异步系统里构造出atomic寄存器**

- 对比failure detector class &Theta; 和 &Sigma;

     - 只要有&Theta;，在CAMP<sub>n,t</sub>[-FC, t &ge; n/2]里就可以构造出URB 广播

     - 只要有&Sigma;，就可以在CAMP里构造出atomic 寄存器

     - 并且两者的实现代码相同，那是否两者的computation power也一样呢？

     **结论：在任何t &ge; n/2的系统里，&Sigma;严格地强于&Theta;。即&Theta;可以在CAMP<sub>n,t</sub>[&Sigma;]里被构造，而&Sigma;无法在CAMP<sub>n,t</sub>[&Theta;]里被构造出来**

     - 推理简述：
     
        回忆两者的properties，它们的liveness都是一样的，区别在于

        - &Sigma; 上要求 **intersection**: 在任何时刻，任意两个process读到的sigma值总有交集 （可以prevent cluster partitioning）

        - &Theta; 上要求**correctness**：在任何时间，任何trusted<sub>i</sub>至少包含一个正常运行的process。

        - 当t &lt; n/2时，从&Sigma;**liveness**和**intersection**结合起来看，可以得出&Theta;的**correctness**。

        - 当t &lt; n/2时，却可以通过反证法得到&Sigma;无法在CAMP<sub>n,t</sub>[&Theta;]里被构造出来。

- 可以用atomic寄存器构造出URB广播

    构造算法简述

    - 算法由一组REG[i..n]组成，每个REG[i]都是一个SWMR的atomic register。每个REG[i]都允许p<sub>i</sub>写，允许所有process读。

    - 每个p<sub>i</sub>的本地变量有：

        - sent<sub>i</sub>

        - reg<sub>i</sub>[1..n]

        这些local variable里都放着一串message

    - **urb-broadcast**:

        首先把msg concatenate到sent<sub>i</sub>。

        然后p<sub>i</sub>调用REG[i].write(sent<sub>i</sub>).

    - **urb-deliver**:

        urb-deliver由一个background task实现：

        这个task是一个while true loop：
        
        在loop里面p<sub>i</sub>会从每个REG[x]里读取msg，然后放到本地reg<sub>i<sub>[x]里。对于reg<sub>i</sub>[x]里每条没有被urb-deliver的msg，去urb-deliver该msg。
    
- 从上面结论中我们可以看到，无论t的值，&Theta;都可以在CAMP[&Sigma;]或者CAMP[-FC, &Sigma;]里被构造，而反过来不行。

    同时&Sigma;又是满足可以被加到CAMP[&empty;]中后能无论t的值都能实现atomic寄存器的the weakest failure detector class。

    这就意味着**从failure detector class的角度来看，atomic寄存器strictly stronger than URB广播**



---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}