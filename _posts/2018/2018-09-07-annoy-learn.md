---
layout:     post
title:      "Annoy"
subtitle:   "Annoy(Approximate Nearest Neighbors Oh Yeah)学习"
date:       2018-09-07
author:     "msg"
header-img: "img/posts/02.jpg"
header-mask: 0.3
catalog:    true

tags:
    - 算法
    - 相似度
    - Annoy
---

> 工作中用到$$Annoy$$，学习的过程中记录一下。

一旦文档经过$$word2vec$$或者$$paragraph2vec$$变成***稠密向量***形式，如何从海量文本中快速查找出相似的$$TopN$$文本呢？

### 建立索引过程

$$Annoy$$的目标是建立一个数据结构，使得查询一个点的最近邻点的时间复杂度是次线性。$$Annoy$$通过建立一个二叉树来使得每个点查找时间复杂度是$$O(logn)$$。随机选择两个点，以这两个节点为初始中心节点，执行聚类数为$$2$$的$$kmeans$$过程，最终产生收敛后两个聚类中心点。这两个聚类中心点之间连一条线段（灰色短线），建立一条垂直于这条灰线，并且通过灰线中心点的线（黑色粗线）。这条黑色粗线把数据空间分成两部分。在多维空间的话，这条黑色粗线可以看成等距垂直超平面。

![分割](/img/posts/annoy/split.png)

在划分的子空间内进行不停的递归迭代继续划分，直到每个子空间最多只剩下$$K$$个数据节点。
![最终分割](/img/posts/annoy/split2.png)

通过多次递归迭代划分的话，最终原始数据会形成二叉树结构。二叉树底层是叶子节点记录原始数据节点，其他中间节点记录的是分割超平面的信息。$$Annoy$$建立这样的二叉树结构是希望满足这样的一个假设:相似的数据节点应该在二叉树上位置更接近，一个分割超平面不应该把相似的数据节点分在二叉树的不同分支上。
![树状结构](/img/posts/annoy/split3.png)

### 查询过程

完成节点索引建立过程后如何进行对一个数据点进行查找相似节点集合呢？查找的过程就是不断看查询数据节点在分割超平面的哪一边。从二叉树索引结构来看，就是从根节点不停的往叶子节点遍历的过程。通过对二叉树每个中间节点（分割超平面相关信息）和查询数据节点进行相关计算来确定二叉树遍历过程是往这个中间节点左子节点走还是右子节点走。

#### 存在问题

上述描述存在两个问题：

1) 查询过程最终落到叶子节点的数据节点数小于我们需要的$$TopN$$相似邻居节点数目怎么办？

2）两个相近的数据节点划分到二叉树不同分支上怎么办？

#### 解决方法

1）如果分割超平面的两边都很相似，那可以两边都遍历；下面是是个示意图：

![示意图](/img/posts/annoy/01.png)

如果一个划分的两边“靠得足够近”（量化方式在后面介绍），我们就两边都遍历。这样就不只是遍历一个节点的一边，我们将遍历更多的点。

我们可以设置一个阈值，用来表示是否愿意搜索划分“错”的一遍。如果设置为0，我们将总是遍历“对”的一片。但是如果设置成0.5，就按照上面的搜索路径。

这个技巧实际上是利用优先级队列，依据两边的最大距离。好处是我们能够设置比0大的阈值，逐渐增加搜索范围。

2) 建立多棵二叉树树，构成一个森林，每个树建立机制都如上面所述那样。

![森林](/img/posts/annoy/tree.jpg)

用一个优先级队列，同时搜索所有的树。这样有另外一个好处，搜索会聚焦到那些与已知点靠得最近的那些树——能够把距离最远的空间划分出去

每棵树都包含所有的点，所以当我们搜索多棵树的时候，将找到多棵树上的多个点。如果我们把所有的搜索结果的叶子节点都合在一起，那么得到的最近邻就非常符合要求。

3）采用优先队列机制：采用一个优先队列来遍历二叉树，从根节点往下的路径，根据查询节点与当前分割超平面距离进行排序。

### 返回最终近邻节点

每棵树都返回一堆近邻点后，如何得到最终的$$TopN$$相似集合呢？
首先所有树返回近邻点都插入到优先队列中，求并集去重, 然后计算和查询点距离， 最终根据距离值从近距离到远距离排序， 返回$$TopN$$近邻节点集合。

### tips

1.距离计算，采用归一化的欧氏距离：$$vectors = \sqrt{(2-2 \cdot cos(u, v))}$$

2.向量维度较小$$(<100)$$,即使维度到达1000表现也不错

3.内存占用小

4.索引创建与查找分离（特别是一旦树已经创建，就不能添加更多项）

5.有两个参数可以用来调节$$Annoy$$树的数量$$n_{trees}$$和搜索期间检查的节点数量$$search_k$$

* $$n_{trees}$$在构建时提供，并影响构建时间和索引大小。 较大的值将给出更准确的结果，但更大的索引。

* $$search_k$$在运行时提供，并影响搜索性能。 较大的值将给出更准确的结果，但将需要更长的时间返回。

如果不提供$$search_k$$，它将默认为$$n * n_{trees}$$，其中$$n$$是近似最近邻的数目。 否则，$$search_k$$和$$n_{trees}$$大致是独立的，即如果$$search_k$$保持不变，$$n_{trees}$$的值不会影响搜索时间，反之亦然。 基本上，建议在可用负载量的情况下尽可能大地设置$$n_{trees}$$，并且考虑到查询的时间限制，建议将$$search_k$$设置为尽可能大。

### python代码例子

```python
from annoy import AnnoyIndex
import random

f = 40
t = AnnoyIndex(f)  # Length of item vector that will be indexed
for i in xrange(1000):
    v = [random.gauss(0, 1) for z in xrange(f)]
    t.add_item(i, v)

t.build(10) # 10 trees
t.save('test.ann')

# ...

u = AnnoyIndex(f)
u.load('test.ann') # super fast, will just mmap the file
print(u.get_nns_by_item(0, 1000)) # will find the 1000 nearest neighbors
```
### python api

```python

AnnoyIndex(f, metric='angular') # returns a new index that's read-write and stores vector of f dimensions. Metric can be either "angular" or "euclidean".
a.add_item(i, v) #adds item i (any nonnegative integer) with vector v. Note that it will allocate memory for max(i)+1 items.
a.build(n_trees) #builds a forest of n_trees trees. More trees gives higher precision when querying. After calling build, no more items can be added.
a.save(fn) #saves the index to disk.
a.load(fn) #loads (mmaps) an index from disk.
a.unload() #unloads.
a.get_nns_by_item(i, n, search_k=-1, include_distances=False) #returns the n closest items. During the query it will inspect up to search_k nodes which defaults to n_trees * n if not provided. search_k gives you a run-time tradeoff between better accuracy and speed. If you set include_distances to True, it will return a 2 element tuple with two lists in it: the second one containing all corresponding distances.
a.get_nns_by_vector(v, n, search_k=-1, include_distances=False) #same but query by vector v.
a.get_item_vector(i) #returns the vector for item i that was previously added.
a.get_distance(i, j) #returns the distance between items i and j. NOTE: this used to returned the squared distance, but has been changed as of Aug 2016.
a.get_n_items() #returns the number of items in the index.

```
