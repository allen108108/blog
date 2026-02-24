---
title: Chapter 3 -- Graph ( 4 )
date: 2019-11-02 16:31:39
categories:
- 課程筆記 Course
- 江蕙如 Algorithm
image: https://blog.techbridge.cc/img/kdchang/cs101/algorithm/algorithm-cover.png
mathjax: true
---

Directed Acyclic Graphs (DAGs) 是被廣泛討論的一種有向圖型態。而接下來要講的 Topological Ordering 也主要是針對 DAGs 做討論。

<!-- more -->

Directed Acyclic Graphs (DAGs)
---

在正式討論 DAGs 之前，我們先來試想一下，在無向圖 ( Undirected graph )中，沒有 cycle 的狀況是什麼樣子 ?

很直覺的可以發現，沒有 cycle 的無向圖會是以一個 tree 或是 forest (multiple tree) 的形態出現，如果是單一個 tree 則有 n-1 條 edges，若為 forest 則 edge 數必然小於 n-1  ( n 為 node 數量 )，也就是說，一個沒有 cycle 的無向圖，其 edge 的狀態是很稀疏 (sparse) 的。

但是一個 directed acyclic graph ，其 edge 的呈現是可以很密集 ( dense ) 的狀態。換句話說， DAGs 可以擁有很豐富的結構型態。


<img width=500 src="https://i.imgur.com/G6SU4qT.png" >


DAGs 適合用來表達具有依存關係、先後順序的物件，如果這些關係用 DAGs 來表達，且其中出現 cycle 則表示這樣的依存關係會呈現一種僵局 ( deadlock )。

DAGs 的討論之所以重要，是因為生活中其實會有不少這種依存關係出現，舉例來說 : 大學四年的課程規劃，科目之間會有一些順序上的限制，不然能會出現擋修的狀態。再者，在電腦中，CPU處理非常多的指令，也必須釐清處指令間的順序、依存關係才能做適當的執行。

Topological Ordering
---

### Definition

* Given a directed graph G, a Topological Ordering is an ordering of its nodes as $v_1,v_2,\cdots,v_n$ s.t. $\forall$  edge $(v_i,v_j),i<j$
對於一個有向圖來說，我們可以將所有 nodes 做出一個排序，使得任一 directed edge 的兩個端點必符合這樣的優先順序。

### Theorem

$G\ has\ a\ topological\ ordering\Longleftrightarrow G\ is\ a\ DAG.$

**Proof :**

($\Longrightarrow$)

Suppose to the contrary that $G$ is not a DAG.
$\Longrightarrow\exists$ a cycle $(v_i,\cdots,v_j,v_i)$ in $G$, where $v_i$ is the node with smallest index $i$.
$\Longrightarrow j>i$
$\because$ edge $(v_j,v_i)$ exists. ( $\because$ cycle $(v_i,\cdots,v_j,v_i)$ )
$\therefore j<i$ ($\rightarrow\leftarrow$)

($\Longleftarrow$) 要利用下面 lemma 才能證明[^註1]



Claim : $G\ is\ a\ DAG\Longrightarrow\exists\ node\ v_k\ doesn't\ has\ incoming\ edges.$

**Proof :**

Suppose to the contrary that $\exists$ an incoming edge in $v$, $\forall v\in V$
$\Longrightarrow v$ has an incoming edge $(v_1,v)$
$\Longrightarrow v_1$ has an incoming edge $(v_2,v_1)$
Continue this process
$\Longrightarrow v_n$ has an incoming edge $(v_{n+1},v_n)$

$\because$ $\mid V\mid=n$
$\therefore\exists$ $v_i,v_j\in \{v_1,v_2,\cdots,v_{n+1}\}$ s.t. $v_i=v_j$
$\Longrightarrow v_i,v_{i+1},\cdots,v_{j-1},v_j=v_i$ form a cycle in G. ($\rightarrow\leftarrow$)

上述這個 Lemma 非常重要，因為有了這個定理，我們可以確定一個 DAG 裡面一定會有一個點是有 incoming edge。而這一個點是不會受到其他點所控制，那我們便可以用其作為 topological order 的第一個點。


![](https://i.imgur.com/OeEeQBq.png)

( a ) 從兩個無 incoming edge 之點擇一
( b ) 確定了 topological order 的第一點後，我們便不在意此點會支配那些 nodes (暫時將 outcoming edge以虛線代替)，原本是 DAG，少掉其中一點的 graph 仍然是 DAG ( 原本沒有 cycle，缺了一點仍然不會有 cycle )
( c ) 既然少掉一點的 graph 仍然是 DAG，那麼它也仍然存在一個沒有 incoming edge 的點。
( d )~( g ) 重複這樣的方式便可以找出整個 topological order。

而這個 lemma 也是證明上述 Theorem 的重要關鍵

($\Longleftarrow$)

Prove by induction.

Suppose that $\mid V_G\mid=k$

Step 1. : $k=1$

G has only one node, and has no edge. $\Longrightarrow$ This is trivial case.
The Theorem is true when $k=1$.

Step 2. : If the theorem is true when $k\leq n$.

Suppose that $G$ is a DAG and $\mid V_G\mid=n+1$
$\Longrightarrow\exists$ node $v$ without incoming edge

Let $G'=G-\{v\}$ and $\mid V_{G'}\mid=n$

$\because G'$ is still a DAG. $\Longrightarrow$ $G'$ has a topological ordering $R=\{r_1,r_2,\cdots,r_n\}$.

Let ordering $R'=\{v,r_1,r_2,\cdots,r_n\}$

$\because v$ has no incoming edge and $\{r_1,r_2,\cdots,r_n\}$ is a topologival ordering
$\therefore R'=\{v,r_1,r_2,\cdots,r_n\}$ is also a topological ordering.($\because \not\exists$ edge $(v,r_m),\forall m\in\{1,2,3,...,n\}$) 

上面的數學歸納法步驟其實也告訴我們一個 topological sorting 的演算法

```
TopologicalOrder(G) :
1. 找到一個沒有 incoming edge 的點 v
2. 對其進行排序
3. G=G-{v} ＃從 G 中移除　ｖ
4. 若 G 不為空，則 TopologicalOrder(G)
```
這樣的演算法，時間複雜度為 $O(n^2)$
( 因為一共要做 n 次 iterations，且每一次都要找一次沒有 incoming edge的點 )

但是我們其實可以配合資料結構來降低時間複雜度，就是一種以「空間換取時間」的概念。

* 用 Adjacency List 來做為基礎表示法
* 每一個 node 必須多給一點空間來儲存 $indgre(\mathbb{w})$ ，這是每一個 iteration 時，還沒有被刪除的 node 的 incoming edge 數量。
* 我們還需要多給一個空間來儲存一個新的集合 $S$，主要是用來放每一個 iteration 中沒有 incoming edge 的 node。

![](https://i.imgur.com/SbGXeSH.png)

我們利用上圖來大致解釋一下過程 : 
( a ) 初始化 $indgre(\mathbb{w})$ 以及 $S$，這裡要掃過所有的 nodes 跟 edges，所以需要時間複雜度為 $O(m+n)$ 

( b ) 當我對某一個 $indgre(\mathbb{w})=0$ 的 node 進行排序後，它所連結到的 nodes $indgre(\mathbb{w})$ 都會 -1，且我們會直接將 $indgre(\mathbb{w})=0$ 的點放進 $S$ 中，而以排序的點也會同時移出 $S$。在圖中，粉紅色標記其實就相當於指出了 $S$ 現在有哪些點。因為這些動作都不需要掃過所有 nodes 跟 edges，時間複雜度相當於 $O(1)$。

( c )~ 後續就依照這樣的方式重複進行，因此總時間複雜度也就是初始化時候的時間複雜度為 $O(m+n)$

這樣的方式相當於使用 BFS 來進行 Topological Sorting，其實在一般的課本或是網路上大多使用的是 DFS 來做 Topological Sorting。

以下是演算法聖經 Cormen 等人所著的 "*Introduction To Algorithm*" 中利用 DFS 來尋找 Topological Ordering 的演算法

```
TopologicalOrder(G) :
1. Call DFS(G) to compute finishing times v.f for each vertex v 
2. As each vertex is finished, insert it onto the front of a linked list
3. Return the linked list of vertices
```

在前面我們有提過，DFS 演算法中，每一個節點都會被進出各一次，我們可以藉此找出每一個節點的進入時間 (discover time, $v.d$ ) 以及離開時間 ( finish time, $v.f$ )。

尋找 Topological Ordering 的過程，我們先對 $G$ 做一次 DFS，找出所有 nodes 的 finish time $v.f$，之後再根據這個 $v.f$ 進行排序即可。

![](https://i.imgur.com/oIcw0Fq.png)


 


註釋
---

[^註1]: 
原本我想直接利用反證法，不經過 Lemma 來直接證明
Suppose to the contrary that G has no topological ordering 但這樣的證明會遇到一個問題 : 非 topological ordering 的排序並不一定會保證會出現 cycle ，這樣我就無法利用這樣的特性製造矛盾。