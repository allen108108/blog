---
title: Chapter 3 -- Graph ( 1 )
date: 2019-10-05 22:12:00
categories:
- 課程筆記 Course
- 江蕙如 Algorithm
image: https://blog.techbridge.cc/img/kdchang/cs101/algorithm/algorithm-cover.png
mathjax: true
---

CAR Theorem
---
CAR Theorem 作為本章節的開頭，提供了我們解決問題的思維流程，而 CAR 包含三個主要的部份 : Criticality、Abstraction 以及 Restriction

<!-- more -->

### Criticality

當我們面對一個待解決的問題時，首先要做的就是純化它。
將ㄧ些旁枝末節的部分先捨棄，直達問題的核心。

### Abstraction

接著我們從更高層次的角度來試圖理解這個問題、解決這個問題 ( Graph Theory 便是一個很好的例子 )，再慢慢的往低層次的方向前進。

### Restriction

最後，我們可以把剛剛被簡化的部分重新考慮進來，試著延伸、調整設計好的演算法來解決最原始的問題。但，千萬不要一開始就處理這樣的部分。


Seven Bridge Problem
---

十八世紀 Königsberg 市區中河道中心有兩座小島，分別有七座橋連接彼此與河的兩岸，Euler 試圖解決一個問題是 : 是否有可能每一座橋只經過一次的情況下，把每一座橋都走過？

Euler 將問題簡化成點和線的情況，並證明這個問題並不存在解。[^註1]

![](https://i.imgur.com/SWbTa5j.png)

這篇論文不但被視為是圖論的始源，也是奠定了數學中拓樸學的基礎。


Definition & Application of Graph
---

### Definition

* A Graph $G(V,E)$ where $V$ is a vertices ( nodes ) set and $E$ is a edge set.
 A Graph $G$ 是由點集合$V$與線集合 $E$ 構成
* $E=\{e\mid e=(u,v),\ u,v\in V\}$
每一個 edge 均是由兩個頂點所構成
* $G$ is a undirected graph, if edges are undirected, i.e. , $e=(u,v)=(v,u)=\{v,u\}$
Undirected graph 具對稱性，連接兩頂點的 edge 可用大括號表示

* $G$ is a directed graph, if edges are directed, i.e. , $e=(u,v)\neq(v,u)$
Undirected graph 不具對稱性。

* $u,v\in V$, $v$ is a neighbor of $u$, if $\exists$ $e=(u,v)$
$v$ 與 $u$ 有 edge 相連，則可稱 $v$ 為 $u$ 的 neighbor。(undirected graph)

* $P=(v_1,v_2,\cdots,v_i,v_{i+1},\cdots,v_k)$ is a path , if $(v_i,v_{i+1})\in E$ and $v_i\in V,\ \forall\ i=1,2,\cdots,k$
path $P$ 是由一個頂點序列所組成。

* $P$ is a simple path, if all vertices are distinct. 
Simple Path 指的是所有頂點均不重複。

* $P$ is a cycle path, if $v_1=v_k,\ k>2$ and $v_2,\cdots,v_{k-1}$ are distinct.
Cycle Path　則是起點與終點相同，其餘頂點均不重複。

* $G$ is connected, if $\forall\ u,v\in V,\ \exists$ a path $P$ from $u$ to $v$
若 Ｇ 中任兩個頂點均存在一個 path，則稱 G 為 connected。

* The distance of $u,v=$ the minimum number of nodes of $u-v$ path $P$
兩頂點的距離由最短 path 決定。

* $G$ is a tree ,if it is connected and it doesn't contain cycle path.
一個 graph 是 connected 但不含任何的 cycle path 我們稱之為 tree。

以上定義在 directed graph 中均適用，但需考慮其方向性。


### Tree

Tree 可以說是最簡單的 graph 型態，具有一些比較特別的性質 : 

1. 下列三個條件中任兩個條件成立必能推得第三個成立
(1) $G$ is connected
(2) $G$ doesn't contain any cycle
(3) $G$ has $n-1$ edges

2. Tree 中任兩點存在唯一 path。

Rooted Tree 是 tree 的一種特殊形態，其 root 會位在某一個頂點 $r$ 上。在ㄧ些具有層次結構性 (hierarchy) 的狀況下，我們多用 rooted tree 來表現。 

Graph Connectivity and Graph Traversal
---
經過上面的定義之後，我們便開始思考頂點間的 Connectivity，若 $s,t$兩點具 connctivity，便稱為 $s-t\ connectivity$，要探討這種 s-t connectivity problem，這邊引進兩種方法 : 

(1) Breadth-First Search
(2)

### Breadth-First Search ( BFS , 廣度優先搜尋 )

BFS 的主要概念是水波的傳導，當我們想要知道與某一點具有 connectivity 的其他頂點時，藉由水波傳導過程將所有頂點做分層。

![](https://i.imgur.com/DJ94uo0.png)


起點做為第 0 層，將其 neighbors 作為第一層，再各自找尋 neighbors 做為第二層，依此類推，我們便可以找到與起點有 connectivity 的所有點。( 如果有重複的點會被 skip )

除此之外，BFS 這樣的分層方式也間接地決定了各點與起點的 distance，$L_i$ 的下標 $i$ 便是在此層中的所有點與起點的 distance.

也因為重複的 vertex 會被後面的 layer 所忽略，因此在 BFS 下會有一些 edge 也被忽略 ( 稱為 non-tree edge )。因此 BFS 下必然會產生一個 tree 的型態，稱為 BFS Tree。

![](https://i.imgur.com/EiqkSEO.png)

BFS 有一個重要的性質如下 : 

**Claim : Assume $T$ is a BFS Tree , $x\in L_i\ ,\ y\in L_j$ are vertices of $T$. If $e=(x,y)$ is an edge of $G$, then $\mid i-j\mid\leq1$.**
$x,y$ 為一個 BFS Tree 的兩個相異頂點，若存在一個 edge $e=(x,y)$ ，那麼 $x,y$ 在 BFS 的層數相差不超過 1。

**Proof :**
Suppose to  the contrary that $\mid i-j\mid > 1$ , $W.L.O.G$ assume $j-1>1$
$\because x\in L_i\Longrightarrow$ the neighbors of $x$ in $L_{k}$ , where $k\leq i+1$
$\because e=(x,y)$ is an edge of $G$ and $y\in L_j$
$\Longrightarrow y$ is a neighbor of $x$ and $y\in L_j$
$\Longrightarrow j\leq i+1$
$\Longrightarrow j-i\leq 1$ ( $\rightarrow\leftarrow$ )

從一開始的 BFS 圖片我們也可以定義 $Connected\ Component$ : 

* Connected component contain vertex s is the largest set of vertices that are reachable from s. 
包含 s 的 connected component 指的就是 s 可以到達的最大點集合

```
Connected-component(s):
1. initial R={s}
2. while ∃ edge(u,v) where u∈R and v∉R, then R+={v}
3. return R
```
**Claim : $R = connected-componont(s)$**

**Proof :**

(Case 1) $R\subseteq connected-componont(s)$

By Definition, clearly.

(Case 2) $R\supseteq connected-componont(s)$

Suppose to the contrary that $R\not\supseteq connected-componont(s)$
$\therefore \exists x\in connected−componont(s)$ and $x\not\in R$
$\because x\in connected−componont(s)\Longrightarrow \exists\ path\ P=(s,x_1,x_2,\cdots,x_n,x)$

initial $R=\{s\}$
$\because (s,x_1)\in E\Longrightarrow x_1\in R$
$\because (x_1,x_2)\in E\Longrightarrow x_2\in R$

Continue this process .....

$\because (x_n,x)\in E\Longrightarrow x\in R$ ($\rightarrow\leftarrow$)



我們常常在繪圖軟體要進行大面積著色時，很常會用到這樣的工具


<img width=300 src="https://i.imgur.com/TWAlTbV.png" >


這便是將每一個 pixel 當作 vertex，利用 Connected component 來快速著色。


### Depth-First Search ( DFS , 深度優先搜尋 )

DFS 的概念比較像是走迷宮，先進行無腦的直衝，等到走進死胡同再退回前一個節點選擇另外一條路前進。

拿前面的圖片為例，DFS 的過程如下 : 

![](https://i.imgur.com/e1bl4Cj.jpg)

這裡面最後一步雖然說結束，但事實上它還是會一路退回到上一個節點確定都沒有其他路徑可以走，就這樣一直退到 vertex 1。

可以寫成下列的 procedure

```
DFS(s) :
1. mark s and R+{s}
2. if ∃ non-marked smallest v s.t. (s,v)∈E ,then DFS(v)
```

從 DFS 的過程來看，有一個特性是每一條路徑都一定會被 check 兩次 (來回)，不管有沒有被 skip 掉都一樣。這也可以推得 DFS 整個過程中，在每一個節點都會有一個進出的過程


<img width=400 src="https://i.imgur.com/eXO3cRZ.png" >


又如同 BFS 一樣，經過 DFS 後都會形成一棵 DFS Tree，走過的路徑便是 Tree Edge，而被忽略的則是 Non-Tree Edge。

![](https://i.imgur.com/BMXqxRA.png)


**Claim : Assume $T$ is a DFS Tree , $(x,y)$ is a non-tree edge of $T$, then $x,y$ is ancester (descendant) of the other. $(x,y)$ 為一個 DFS Tree 的 non-tree edge，則 $x,y$ 必為祖孫層級(上下至少跨兩個層)的關係。**

**Proof**
$W.L.O.G.$ suppose $x$ is reached earlier by DFS.
$(x,y)$ 是 Non-Tree edge (被忽略的 edge) 
$\Longrightarrow (x,y)$ 這條 edge 第一次check沒有被選中，而第二次被判斷要被忽略
$\Longrightarrow y$ 在第二次 check $(x,y)$ 這條路徑時，$y$ 已經被 marked 。
$\Longrightarrow y$ 一定是在 DFS 進入 $x$ 到退出 $x$ 這中間被 marked。
$\Longrightarrow x$ must be the ancester of $y$

### Summary of BFS & DFS

* BFS 或 DFS 均可以找出 Connected-Component
* BFS Tree　寬且淺，DFS Tree 窄且深。
* DFS Tree 的 Non-Tree Edge 連結上下層級且至少相差兩層以上的 vertex ;
  BFS Tree 的 Non-Tree Edge 連結上下一層或是同層的 vertex。

註釋
---


[^註1]:
證明可參閱 [*數學傳播 36卷4期 pp. 42-47*](https://web.math.sinica.edu.tw/math_media/d364/36404.pdf) 