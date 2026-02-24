---
title: Chapter 3 -- Graph ( 2 )
date: 2019-10-11 10:50:22
categories:
- 課程筆記 Course
- 江蕙如 Algorithm
image: https://blog.techbridge.cc/img/kdchang/cs101/algorithm/algorithm-cover.png
mathjax: true
---


Representing Graph
---

$G=(V,E)$ is a graph
* $\mid V\mid=$ the number of nodes $\overset{let}{=} n$
$\mid E\mid=$ the number of edges $\overset{let}{=} m$
$\Longrightarrow n-1\leq m\leq \displaystyle{\binom{n}{r}} \leq n^2$
$\Longrightarrow$ 我們會希望整個複雜度可以控制在 $O(m+n)$ 

<!-- more -->

### Adjacency Matrix

The $adjacency\ matrix$ of $G$ 是一個 $n\times n$ 矩陣 $A$
$$
A=\begin{bmatrix}a_{11}a_{12}\cdots a_{1n}\\a_{21}a_{22}\cdots a_{2n}\\\vdots\\a_{n1}a_{n2}\cdots a_{nn}\end{bmatrix}
$$
* $a_{ij}=1$ , if $(i,j)\in E$
* $a_{ij}=0$ , if $(i,j)\not\in E$
* If $G$ is undirected graph, then $A$ is symmetric.
* The time complexity for checking if $(i,j)\in E=O(1)$
( 我們只要找第 $i$ 列第 $j$ 行即可知道是否存在 $(i,j)$ )
* The time complexity for finding out all neighbors of $i\in V=O(n)$
( 需要遍歷第 $i$ 行(列)，才能找出所有 neighbors )
* The space complexity $=O(n^2)$

我們可以從這個矩陣中 0 的數量跟密集程度來看看這是 Sparse Graph 還是 Dense Graph，如果今天是一個 Sparse Graph，其矩陣也是 Sparse，那麼相當於我們必須利用許多的空間來儲存 0 這樣無意義的資料，由此我們可以知道， 使用 adjacency matrix 來表示 Sparse Graph 是不太適合的。


### Adjacency List

$Adjacency\ List$ 中我們會先以一個 array 儲存每一個 vertex，之後再用 Linked List 來表示 vertex 與 vertex 之間的相鄰關係。

$$
Adj[i]=\{j\mid (i,j)\in E\}\\
\mid Adj[i]\mid=degree(i)=deg(i)=the\ number\ of\ neighbors\ of\ i
$$

* The time complexity for checking if $(i,j)\in E=O(n)$
( the worse case 就是要遍歷完整串 Linked List 才會確認兩者是否存在 edge )
* The space complexity $=O(n+m)$
* in-degree : 指的就是指向自己的 edge 數
out-degree : 指的就是指向別人的 edge 數



![](https://i.imgur.com/qJR1Rm6.png)
( Adjacency Mtrix v.s. Adjacency List，圖片來源 : [Graph: Intro(簡介)](http://alrightchiu.github.io/SecondRound/graph-introjian-jie.html) )

Implementation
---

在介紹 BFS 與 DFS 的 implementation 之前，我們要先了解所使用的資料結構 : Queue and Stack

### Queue

Queue 我們可以看作是一個水管，資料進出的型態是 「先進先出」 (First in, First out. FIFO)，而這樣的資料結構伴隨兩個 Pointer : front 以及 rear，這兩個指標分別指出目前資料的頭尾，其實也是指出接下來要輸出、輸入的資料位置。


<img width=500 src="https://i.imgur.com/5UcHSnc.jpg" >



### Stack

而 Stack 比較像是一個水瓶，資料型態則是 「後進先出」 (Last in, First out. LIFO)，值得注意的是，在 Stack 這樣的資料型態中，我們只需要一個 Pointer : Top ，用來指出最後進來的資料是誰，也相當於指出接下來要輸出的資料是哪一筆。


<img width=250 src="https://i.imgur.com/bNZmeNI.jpg" >



不論是 Queue 還是 Stack，我們都可以利用 Linked List 來進行 implement。

### Implementing BFS

在要進行 BFS 的 implementation 以前，我們得先決定 Garaph representation，在 BFS 裡面我們使用的是 Adjacency List 來表現一個 Graph，其原因是 Adjacency List 的儲存方式跟 BFS 的搜索方式是吻合的，如此一來可以有效降低 Complexity。

```
// T is a BFS tree rooted s
// Layers count i
// L[i] is Layer_list
```

```
BFS(s)
1. init i=0, L[0]={s}, T={} and mark[s]=True, marked[others] are False
# 設定各項初始值
2. while L[i] is not empty then L[i+1]={}
       for each u in L[i] 
           for each (u,v) in E :
               if marked[v] = false then
                   marked[v] = true
                   T = T + {(u,v)}
                   L[i+1] = L[i+1] + {v}
       i+=1
```

* 對於 $L[i]$ ，我們應該要用什麼資料結構來處理 ? 
事實上，Queue / Stack 均可，這兩者差異在於資料處理的順序，但在 BFS 中，每個 Layer 內部的順序性並非那麼重要。( 但若我們今天只想要用一條 List 儲存所有的 Layer 那就必須使用 Queue，因為層與層之間的順序性必須被考慮進去。 )

* Time Complexity : $O(m+n)$
在 BFS 的過程中，每一個 Vertex 都必須 check 每一個 neighbor，如果我們選用的是 Adjacency List ，便可把複雜度控制在 $O(m+n)$ 之中。

但倘若我們選擇的是 Adjacency Matrix，則複雜度會趨近於 $n^2$ 。



### Implementing DFS

之前我們給的 DFS 是一個 recursive code，也可以將其譯為一個 iterative 的版本。

這裡我們使用的是 stack 的型態來處理 vertex set S。
```
// S is a stack of vertice its neighbors haven't been marked entirely.
```

```
DFS(s)
1. init S={s}
2. while S is not empty then
       remove a vertex u from S
       if marked[u] = false then
           marked[u] = true
           for each (u,v) in E 
               S+={v}
```

* iterative code 使用的是 stack 形式做處理，而 recursive code 其實也是 stack，因為 recursive 本身就是一個 stack 形式。
* Time Complexity : $O(m+n)$ .... Adjacency List


Summary
---

### Adjacency matrix VS. Adjacency list for graph

|Scenario|Choice|Complexity|
|-|-|-|
|Find an edge|Adjacency matrix|$O(1)$|
|Find degree|Adjacency list|$<O(n)$|
|Traverse the graph|Adjacency list|$O(m+n)$|
|Storage for sparse graph|Adjacency list|$<O(n^2)$|
|Storage for dense graph|Adjacency matrix|$O(n^2)$|
|Edge insertion/deletion|Adjacency matrix|$O(1)$|
|Most application|Adjacency list|-|


### Graph traversal

* Data structure of BFS : queue / stack
* Data structure of DFS : stack
* Implementation of BFS/DFS with Adjacency list : $O(n+m)$