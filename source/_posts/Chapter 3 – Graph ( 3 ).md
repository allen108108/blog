---
title: Chapter 3 -- Graph ( 3 )
date: 2019-10-24 16:03:34
categories:
- 課程筆記 Course
- 江蕙如 Algorithm
image: https://blog.techbridge.cc/img/kdchang/cs101/algorithm/algorithm-cover.png
mathjax: true
---


Application of BFS -- Testing Bipartiteness
---

### Definition 

The nodes of graph can be partitioned into two sets $X$ and $Y$ and one every edge has one end in $X$ and the other end in $Y$.We call it bipartite graph ( bigraph ) .
一個 graph 的 nodes 可以分成兩個集合，且每一條 edge 兩端連接的端點均屬不同集合者稱之為 bipartite graph ( bigraph ) 。

<!-- more -->

從此定義我們可以推得兩個 bipartite graph 性質

* $X\cap Y=\phi$
* $\forall x_1,x_2\in X\Longrightarrow x_1\ isn't\ the\ neighbor\ of\ x_2\Longrightarrow\not\exists\ edge\ between\ x_1\ and\ x_2$

了解其定義後，我們要問的問題是，給定一個 graph，我們如何判定是否為 bipartite graph ? 直覺一點的方式就是直接將每一個 edge 的兩個端點塗上兩個不同的顏色，然後看看是否會有衝突的地方。

![](https://i.imgur.com/HIRe8wh.png)

如果寫成 procedure 的話 (假設 $G$ 是 connected graph以簡化運算)

```
1. 先任選一點 s∈V,並且將其著色成紅色
2. 對它的所有 neighbors 著色成藍色
3. 重複對 neighbors 塗 紅/藍 色直至所有點都被著色完成
4. 若為 bipartite graph 則所有 edges 的兩端都為不同顏色
```

這個 procedure 要 implement 其實就是用 BFS，只是在 BFS procedure 中每一個 layer 再多加一個著色的動作即可。

[ 補充 ]

實際上，有一個定理可以幫助我們更方便來做判定 : 

$Graph\ G\ is\ bipartite,\ if\ and\ only\ if\  it\ doesn't\ have\ any\ odd\ cycle.$
如果 $G$ 是一個 bipartite graph，則不存在奇數 circle。也就是說，$G$ 裡面的每一個 circle，都是由偶數條 edges 所構成。

**Claim : $G\ is\ a\ connected\ graph\ and\ L_0,L_1,\cdots are\ the\ layers\ produced\ by\ BFS\ starting\ at\ node\ s$**
1. $There\ is\ no\ edge\ of\ G\ joins\ two\ nodes\ of\ the\ same\ layer\ \Longleftrightarrow\ if\ G\ is\ bipartite.$
2. $There\ is\ an\ edge\ of\ G\ joins\ two\ nodes\ of\ the\ same\ layer\ \Longleftrightarrow\ G\ contains\ an\ odd-length\ cycle.$



![](https://i.imgur.com/wx1NDSE.png)



**Proof :**

(2.$\Longrightarrow$)
假設在第 $L_j$ 層有一條 edge 相連兩個 nodes $x,y$，由於 BFS 均可以 tree 的型態表示，此兩點 $x,y$ 與原點 $s$ 之間必然各自存在一條 path，且這兩條 paths 會在某一層 $L_i$ 中某一點 $z$ 交會。


<img width=500 src="https://i.imgur.com/79x2j7a.png" >



從上圖可以知道 $x,y,z$ 三個 nodes 會形成一個 cycle，其 length$=1+2\times (j-i)$
$\therefore\ x,y,z$ 三個 nodes 會形成一個 odd cycle
$\Longrightarrow$ G is not a bipartite graph

(2.$\Longleftarrow$) 
Clearly by definition.

(1.) 
Clearly by 2.

Connectivity in directed graphs
---

### Definition

* Nodes $u,v$ are mutually reachable if $\exists$ a path from $u$ to $v$ and also $\exists$ a path from $v$ to $u$
若兩點間互相存在一條 path 可以從一點到另一點，則稱之為 mutmally reachable。
* A directed graph is strongly connected if $\forall$ pairs of nodes are mutually reachable.
一個有向圖若任兩點均為 mutually reachable，則稱其為 stronglly connected。
* The strongly (connected) component $S$ containing $s$ in a directed graph is the maximal set if $\forall v\in S$ s.t. $v$ and $s$ are mutually reachable.
所有和 $s$ 為 mutually reachable 的 nodes 形成一個 strongly component。

### Lemma

$If\ u\ and\ v\ are\ mutually\ reachable,\ and\ v\ and\ w\ are\ mutually\ reachable,\ then\ u\ and\ w\ are\ mutually\ rea  chable.$

### Testing strongly connectivity

如果給定一個 directed graph ，我們可以如何檢查它是否為 strongly connected ? 最直覺也是最笨的方式就是做兩次 BFS，第一次先檢查 $s$ 可以到達的點有哪些，第二次再檢查這些點能不能回到 $s$。

然而這樣的複雜度太高，因此稍微調整一下。

```
TestStronglyConnectivity(G):

1. 選擇任一點作為起始點 s
2. R=BFS(s,G)  
#針對此起始點對 G 做BFS
3. R'=BFS(s,G') where G' = the graph that it's edges are reverse direction of all edges of G
#製造一個方向與 G 完全相反的有向圖 G'，並做一次 BFS
4. 若 R=V=R'，則 return true else false
```

這裡每一個步驟都是 linear time complexity，最後的總時間複雜度為 $O(m+n)$

關於 strongly (connected) component 有一些蠻有趣的性質

* 任一個 directed graph 必可以拆出數個 strongly (connected) component。
* 將 directed graph 的所有 strongly (connected) component 都各自視為一個點，那麼必會形成一個 directed acyclic graph。


<img width=500 src="https://i.imgur.com/YZJEmIi.png" >




### Theorem

$For\ any\ two\ nodes\ s\ and\ t\ in\ a\ directed\ graph,\ their\ strongly\ component\ are\ identical\ or\ disjoint.$
在一個有向圖中任取兩個 nodes，這兩個 nodes 必在相同的 Strongly Component 不然就各在互斥的兩個 strongly component 中。

**Proof :**

( Case 1. ) $s$ and $t$ are mutually reachable.

Clearly be definition.

( Case 2. ) $s$ and $t$ are not mutually reachable.

Suppose to the contrary that $\exists v$ s.t. $s$ and $v$ are mutually reachable and $t$ and $v$ are mutually reachable

$\Longrightarrow$ $s$ and $t$ are mutually reachable. ($\rightarrow\leftarrow$)


