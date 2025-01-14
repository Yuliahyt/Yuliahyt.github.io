---
title: '最短路算法'
toc: true
tags:	
  -图算法

---



本文总结了最短路的几个经典算法。由于笔者在网上看到的很多文章都对算法流程做了详细的介绍，但笔者发现大部分文章对算法正确性并没有给出证明，而这些证明对于初次接触的人并不是显然的，因此笔者决定撰写该文章也增进自己学习理解。

大部分内容参考自 [OI-wiki](https://oi-wiki.org/graph/shortest-path/)，且由于OI-Wiki写的较为详细，本文较为简略，但部分内容和证明笔者自认为比OI-Wiki写的更全面和直观，



## 最短路的重要性质

下面两个性质是基本所有最短路算法都会用到的，故列在前面。

#### 性质1.1：三角不等式与松弛

松弛操作，是所有最短路算法的核心。

松弛操作，首先来自于最短路的三角不等式，该不等式的成立由于最短距离的定义是显然的

三角形不等式：$d(v) \leq d(u) + w(u, v)$。

而松弛操作$relax(u,v)$ 操作指：$d(v) = min(d(v), d(u) + w(u, v))$.

#### 性质1.2

对于从$s-t$如果存在最短路径$s-x_1-x_2...x_n-t$,那么如果按照$(s,x_1),(x_1,x_2),...,(x_n,t)$的顺序对边依次进行松弛（该顺序不需要连续），那么可以找到$s-t$的最短路径。

该性质对于所有最短路算法都极为重要。

证明：归纳法证明。只要证明，按照该顺序松弛，路径上的每一个结点$u$都找到了其$d(u)$.



## 正权图单源最短路：Dijkstra

我们知道，使用BFS可以在无权图上寻找到最短路，其证明也是显然的。

无权图本质是边权均为1的正权图，而Dijkstra算法可以看作是BFS算法在正权图上的推广，其使用的贪心思想和BFS算法想通过，都是不断拓展距离原点最近的结点。

### 正确性证明

主要利用了性质1.2. 其他部分和BFS的正确性证明类似，可以由归纳法说明Dijkstra的贪心过程按照$d(u)$由小到大的顺序计算出了每个结点的$d(u)$,并且按照其最短路径松弛。较为直观，从略。

### 时间复杂度

简单的版本采用最小堆/最小优先队列，复杂度来源于：

* 松弛操作，更改堆元素的值
* 从堆中取出最小值

总复杂度为$O(ElgV)$

采用斐波那契堆可以拥有更小的复杂度。



## 负权图单源最短路：Bellman-Ford

Dijkstra算法只适用于正权图，而对于带负权的图需要另外一个算法，该算法同时可以判断最短路的存在性（也即判断图中的负环，因为有负环时沿着负环走最短路可以达到$-\infty$,故不存在最短路）。

Bellman-Ford算法，在每一轮中尝试对每条边进行松弛，总共执行$V-1$次松弛操作。

在无负环的情况下，该算法执行后所有的最短路径都被找出来了。

若执行后，还能被松弛，说明存在负环（可以无限松弛下去）。

### 正确性证明

同样依托于性质1.2，由于每一轮都尝试对边进行松弛，说明必然存在性质1.2中按照该顺序松弛的条件。

### 队列优化版本：SPFA

由于有效的松弛操作仅针对于上一轮被松驰过的结点，因此可以用一个队列记录这些结点。

该优化技术也称为SPFA。

### 时间复杂度

共$O(V)$轮，每轮执行$O(E)$次松弛，总复杂度为$O(VE)$.



## 多源最短路：Floyd

### 正确性证明

FLoyd算法基于DP。

首先将结点从$1..V$排序，依次考虑第$k$个结点。

定义$d_k(s,t)$为从$s$到$t$只经过$1...k$中的结点的最短距离，则$d_k$ 要么经过了第$k$个结点(也即为$d_{k-1}(s,k)+d_{k-1}(k,t)$)，要么不经过（退化为$d_k-1$),则：

$ d_k(s,t) = \min(d_{k-1}(s,t) ,d_{k-1}(s,k)+d_{k-1}(k,t))$ 

因此经过上述DP转移公式可以计算出所有结点对的最短路。

### 时间复杂度

时间复杂度为$O(V^3)$。



## 稀疏图上的多源最短路：Johnson

JohnSon尝试将带负权的多源最短路转化为不带负权的多源最短路，然后使用$V$次Dijkstra算法求解。

证明部分同样参考自：[OI-wiki](https://oi-wiki.org/graph/shortest-path/)

### 正确性证明

#### 性质4.1：最短距离与势能

在重新标记后的图上，从 $s$ 点到 $t$ 点的一条路径 $s \to p_1 \to p_2 \to \dots \to p_k \to t$ 的长度表达式如下：

$(w(s,p_1)+h_s-h_{p_1})+(w(p_1,p_2)+h_{p_1}-h_{p_2})+ \dots +(w(p_k,t)+h_{p_k}-h_t)$

化简后得到：

$w(s,p_1)+w(p_1,p_2)+ \dots +w(p_k,t)+h_s-h_t$

无论我们从 $s$ 到 $t$ 走的是哪一条路径，$h_s-h_t$ 的值是不变的，这正与势能的性质相吻合（势能的绝对值往往取决于设置的零势能点，但无论将零势能点设置在哪里，两点间势能的差值是一定的）。

因此，我们把 $h_i$ 称为 $i$ 点的势能。

#### 性质4.2

Johnson算法对于每一条从 $u$ 点到 $v$ 点，边权为 $w$ 的边，则我们将该边的边权重新设置为 $w+h_u-h_v$。

性质1.1告诉我们，这样子重新赋值后并没有改变结点的势能，而下面我们再证明，该赋值将使所有边权变为非负。

证明：由三角不等式（性质1.1）

根据三角形不等式，图上任意一边 $(u,v)$ 上两点满足：$h_v \leq h_u + w(u,v)$。这条边重新标记后的边权为 $w'(u,v)=w(u,v)+h_u-h_v \geq 0$。这样我们证明了新图上的边权均非负。

#### 性质4.3

Johnson算法可以求解多源最短路。

证明：根据性质4.1，我们证明了重新标注边权后图上的最短路径仍然是原来的最短路径。根据性质4.2，我们证明了新图中所有边的边权非负，而在非负权图上，Dijkstra 算法能够保证得出正确的结果。

这样，我们就证明了 Johnson 算法的正确性。

### 时间复杂度

该算法的时间复杂度为$O(V)$次Dijkstra算法的复杂度，也即$O(VElgV)$.

在很多现实场景下（稀疏图），该算法优于Floyd算法。