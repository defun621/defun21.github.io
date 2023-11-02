---
layout: post
title:  early deciding
date:   2023-11-01 22:45:00 -0700
categories: [Fault-Tolerant-Message-Passing-Distributed-Systems]
---

教科书链接：<https://link.springer.com/book/10.1007/978-3-319-94141-7>

- early decision

    虽然 t + 1轮中最坏情况是达到interactive consistency的轮数，
    
    但当在execution中，挂掉的process数f相比模型CSMP<sub>n,t</sub>[&empty;]里的t小的时候，需要的轮数也会相应的更小。

    当一个process decide了一个值，结束propose操作，也就是说不再往别的process广播message。

    这叫做early dicision/stopping。

- 问题的关键还是分布式系统的老问题：

    如果p<sub>i</sub>没有收到从p<sub>j</sub>那里发来的消息，p<sub>i</sub>无法判断p<sub>j</sub>是挂了还是决定不发了。

- 为了能early stopping，要求process遵循以下行动准则：

    - process直到挂掉或者decide为止每轮都要发消息

    - 每条消息都可以indicate发送者是否在该轮发送完之后就会decide 值

- 这样的话，假设p<sub>i</sub>在第r轮第一次没有从p<sub>j</sub>收到消息。

    说明p<sub>j</sub>在第r - 1轮要么挂掉了，要么decide出value了，或者是在第r轮中往p<sub>i</sub>发出消息之前挂掉。

    如果p<sub>j</sub>成功decide出值来，则显然在r - 1轮之前的所有轮（包括r - 1）中都往p<sub>i</sub>发送过 pair <k, v>。

- 不妨令：

    - UP<sup>r</sup>是第r轮时活着的process

    - R<sub>i</sub><sup>r</sup>是第r轮中，p<sub>i</sub>收到的所有消息的发送方构成的集合

    - 不失一般性，令R<sub>i</sub><sup>0</sup>是所有n个process构成的集合。

    如果process按照之前的行动规则执行并且process挂掉之后不会recover，则有

    - &forall; r &ge; 1: R<sub>i</sub><sup>r</sup> &sube; UP<sup>r</sup> &sube; R<sub>i</sub><sup>r - 1</sup>

- **当R<sub>i</sub><sup>r</sup>=R<sub>i</sub><sup>r-1</sup>时** ，p<sub>i</sub>可以early decision。

    因为此时已经知道r轮知道的新信息不会更多，未来只会更少。

- 此外p<sub>i</sub>还得通知别的process p<sub>j</sub>它early decide了。

    所以假设在第r轮p<sub>i</sub>已经可以early stopping，还需要额外一轮来forward自己在第r轮得到的所有<k,v>。

    而p<sub>j</sub>可以在第r + 2轮decide，因为在第r + 1轮收到消息后还需要同样额外一轮来forward。

- early stopping算法

    - 本地变量

        - view<sub>i</sub>[1..n]，和interactive consistency的实现中的view<sub>i</sub>一样

        - new<sub>i</sub>，和interactive consistency的实现中的new<sub>i</sub>一样

        - early<sub>i</sub>，当local early decision predicate *R<sub>i</sub><sup>r</sup>=R<sub>i</sub><sup>r-1</sup>* 被满足时，该值被置为true

        - nbr<sub>i</sub>[0..n]，其中nbr<sub>i</sub>[r] 即为 \|R<sub>i</sub><sup>r</sup>\|。

            所以local early decision predicate 即为 nbr<sub>i</sub>[r - 1] = nbr<sub>i</sub>[r]

        - decide<sub>i</sub>，当从p<sub>j</sub>收到消息知道early<sub>j</sub>为true时，该值被置为true。

    - propose()实现简述

        首先初始化view<sub>i</sub>，new<sub>i</sub>，nbr<sub>i</sub>和early<sub>i</sub>。

        开始t + 1轮synchronous round，在每一轮中：

        先广播消息EST(new<sub>i</sub>, early<sub>i</sub>)。

        再把收到所有new<sub>j</sub> (j &isin; {1,...,n} \ {i})，放入recfrom<sub>i</sub>数组中。

        如果此时early<sub>i</sub>为true，则直接返回view<sub>i</sub>。

        否则计算nbr<sub>i</sub>[r]，计算decide<sub>i</sub> &larr; &#x22C1;(early<sub>j</sub>)，

        然后开始计算new<sub>i</sub>，即recfrom<sub>i</sub>[j]不为空并且如果view<sub>i</sub>[k]为空值，则view<sub>i</sub>[k] &larr; v并且 new<sub>i</sub> &larr; new<sub>i</sub> &cup; {<k,v>}。

        如果此时满足nbr<sub>i</sub>[r - 1] = nbr<sub>i</sub>[r]或者decide<sub>i</sub>是true，则置early<sub>i</sub>为true。

        最后如果已经是第t + 1轮， 则返回view<sub>i</sub>.




---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}