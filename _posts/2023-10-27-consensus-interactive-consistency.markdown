---
layout: post
title:  consensus和interactive consistency定义
date:   2023-10-27 21:45:00 -0700
categories: [Fault-Tolerant-Message-Passing-Distributed-Systems]
---

教科书链接：<https://link.springer.com/book/10.1007/978-3-319-94141-7>

- consensus问题在process crash failure模型里可以定义为一个distributed agreement abstraction

    - propose()操作

        如果一个process p<sub>i</sub> 调用了propose(v<sub>i</sub>)，最后propose(v<sub>i</sub>)返回了w<sub>i</sub>。

        我们说p<sub>i</sub> propose了v<sub>i</sub>，p<sub>i</sub> decide了w<sub>i</sub>。

    - 性质：

        - **CC-validity**：一个被decide的值肯定是一个被某个process proposed的值

        - **CC-agreement**：不存在两个decide出不同值的process。

        - **CC-termination**：每个正确的process都能decide 一个值出来。

        其中CC表示在crash failure model下的consensus。

        CC-validity和CC-agreement保证了safety。
        
        CC-validity说明decided value是从proposed value中选出的。

        CC-agreement保证了最后会只有一个值。

        CC-termination体现了liveness。没挂掉的process上的propose操作总会terminate。

- uniform consensus

    通过CC-agreement和CC-termination保证了如果一个process了一个值，再crash掉，他不会产生不同的值。

- non-uniform consensus

    是比uniform consensus更弱的模型。

    它有性质：**Non-uniform CC-agreement**：不存在两个没挂的process会decide出不同的结果。

    也就是说，在non-uniform consensus里允许process挂掉并decide出和别的process不同的值。

- **binary consensus 和 multivalued consensus**

    不妨令V是可以被proprose的值的集合。

    如果|V|=2，那么这个consensus是binary的，通常认为V = {0, 1}。

    如果|V| &gt; 2，则这个consensus是multivalued。V可以是一个有限集合也可以是一个无限的集合。

- interactive consistency （也叫vector consensus）

    和consensus不同，interactive consistency不是只得到一个decided value，而是得到一个decided vector。所以叫做vector consensus。

    - **性质**：

        - **ICC-validity**：

            不妨令D<sub>i</sub>[1..n]是由p<sub>i</sub> decide出的vector。

            则有 &forall; j &isin; [1..n]: D<sub>i</sub>[j] &isin; {v<sub>j</sub>, &#x22A5;}，

            其中v<sub>j</sub>是由p<sub>j</sub>propose的值。
            
            如果p<sub>j</sub>没挂，则D<sub>i</sub>[j] = v<sub>j</sub>。

        - **ICC-agreement**：不存在能decide出不同值的两个process

        - **ICC-termination**： 每个没有挂掉正常运行的process都可以最后decide出同一个vector。

    - 值得注意的是：

        如果D<sub>i</sub>[j] = &#x22A5;，则说明p<sub>i</sub>知道p<sub>j</sub>挂了。

        反之，如果D<sub>i</sub>[j] &ne; &#x22a5;，则说明p<sub>i</sub>并不能下判断p<sub>j</sub>是否正常运行。

- 由ICC 可以得到CC，而从CC不能得到ICC，所以ICC是在CSMP<sub>n,t</sub>[&empty;]中比CC更强的抽象。


---
本文章属于以下分类：
{% for category in page.categories %}
- {{ category }}
{% endfor %}