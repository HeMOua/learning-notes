# Node2Vec

> **node2vec是一种综合考虑DFS邻域和BFS邻域的graph embedding方法**。简单来说，可以看作是deepwalk的一种扩展，是结合了DFS和BFS随机游走的deepwalk。

## 算法详解

### 优化目标

对于图中每个顶点 $u$

+ 设 $f(u)$ 是将顶点 $u$ 映射为 embedding 向量的**映射函数**
+ 定义 $N_S(u)$ 为通过采样策略 $S$ 采样出的顶点 $u$ 的**近邻顶点集合**

**node2vec 优化的目标是给定每个顶点条件下，令其近邻顶点（如何定义近邻顶点很重要）出现的概率最大。**
$$
max_f\sum_{u\in V} log Pr(N_s(U)|f(u))
$$
为了将上述最优化问题可解，文章提出两个假设：

- 条件独立性假设：假设给定源顶点下，其**近邻顶点出现的概率与近邻集合中其余顶点无关**
- 特征空间对称性假设：顶点作为源顶点和作为近邻顶点的时候共享同一套embedding向量

### 顶点序列采样策略

node2vec 依然采用随机游走的方式获取顶点的近邻序列，不同的是 node2vec 采用的是一种有偏的随机游走。

给定当前顶点 $v$ ，访问下一个顶点 $x$ 的概率为
$$
P\left(c_{i}=x \mid c_{i-1}=v\right)=\left\{\begin{array}{cc}
\frac{\pi_{v x}}{Z} & \text { if }(v, x) \in E \\
0 & \text { otherwise }
\end{array}\right.
$$
其中 $\pi_{vx}$ 是顶点 $v$ 和顶点 $x$ 之间的未归一化转移概率， $Z$ 是归一化常数。

node2vec 引入两个超参数 $p$ 和 $q$ 来控制随机游走的策略，下面讨论超参数 $p$ 和 $q$ 对游走策略的影响：

+ p，Return parameter：参数[公式]控制重复访问刚刚访问过的顶点的概率
  + 若 $p$ 较高，则访问刚刚访问过的顶点的概率会变低，反之变高。

+ q，In-out papameter：
  + $q$ 控制着游走是向外还是向内
  + 若 $q \gt 1$ ，随机游走倾向于访问和 $t$ 接近的顶点(偏向BFS)
  + 若 $q \lt 1$ ，倾向于访问远离 $t$ 的顶点(偏向DFS)

### 学习算法

采样完顶点序列后，剩下的步骤就和deepwalk一样了，**用word2vec去学习顶点的embedding向量**。 值得注意的是node2vecWalk中不再是随机抽取邻接点，而是按概率抽取，node2vec采用了Alias算法进行顶点采样。

