---
title: Chapter 4 -- Greedy algorithms ( 3 )
date: 2019-11-15 03:22:14
categories:
- 課程筆記 Course
- 江蕙如 Algorithm
image: https://blog.techbridge.cc/img/kdchang/cs101/algorithm/algorithm-cover.png
mathjax: true
---

Shortest Paths ( DijKstra's Algorithm )
---

### Scenario

$G=(V,E)\ is\ a\ directed\ graph\ and\ I(e)=lenth\ of\ edge\ e\in E,\\ where\ I(e)\geq0\ can\ be\ distance,time\ or\ cost.$
$Find\ the\ shortest\ path\ P(v)\ from\ s\ to\ each\ other\ v\in V-\{s\},\ where\ I(P)=lenth\ of\ P=\sum_{e\in P} I(e)$
給定一個有向圖以及每個 edge 的 "長度"( 可以是距離、花費時間、花費成本...)，試圖找出從 s 出發距離最短的路徑。

<!-- more -->

這邊比較特殊的是，這邊不預設終點。因這個演算法不單單可以找出給定起終點的最短路徑問題，也可以更廣泛的解決給定起點不限終點的最短路徑問題。


```
DijKstra(G,I)
// S = the set of exploded nodes
// d(u) = the shortest path distance from s to u
1. initial S = {s} and d(s)=0
2. while S ≠ V then do
       choose node v ∉ S 但存在至少一條　edge 與 S 相連
       且 d'(v) = min { d(u)+I(e) } , u ∈ S and e = (u,v) 
       S = S + {v} and  d(v)=d'(v)
```

### Correctness

在證明正確性之前，我們先了解一下在演算法證明中的一些小技巧。之前說過，Greedy Algorithm 證明的兩個方向 : The Greedy Algorithm stays ahead 與 The exchange argument，在這個部份我們使用的是前者。

此外，在有 Loop 迴圈的演算法中，有時候我們會先試圖證明 Loop invariant property，在嘗試推出結果。所謂的 Loop invariant property 指的是在每一個迴圈中保持不變的性質。

**Loop invariant : 在演算法執行過程中，對所有 $u\in\ S$，且 $d(u)$ 是 $s-u$ 的最短路徑 $P(u)$ 的長度。**

S 的迴圈過程是一次又一次的迭代添加元素，所以我們可以用歸納法來進行證明。
Step 1. : 當 $|S|=1$，顯然成立。
Step 2. : 假設 $|S|=k\geq1$ 亦成立。
當 $|S|=k+1$ : 
假設在這個迴圈中我們添加的是 $v$，且令 $(u,v)$ 為 $s-v$ 的路徑 $P(v)$ 中的最後的 edge。

![](https://i.imgur.com/ddDe69H.jpg)

對其他的 $s-v$ 路徑 $P$ 來說，在 $s,v$ 之間一定有一個路徑是從 $S$ 內的 node $x$ 指到 $S$ 外的 node $y$。
從演算法可以知道 $d(u) + I(u,v) \leq d(x) + I(x,y)+I(y,u)$
又，$d(u)$ 是 $s-u$ 的最短路徑長度。
所以，對所有的 $s-v$ 路徑來說，$s-u-v$ 這樣的路徑一定是最短路徑。
$\Longrightarrow d(v)=d(u) + I(u,v)\leq I(P)$

從上面的證明可以知道，DijKstra's Algorithm 必須要在所有 edge 權重非負的條件下才能成立。

Implementation
---

在 DijKstra's Algorithm 中，implementation 最困難的部分就是 while loop 迴圈中的第一二行，必須配合適當的資料結構才能夠真正有效率的運行 DijKstra's Algorithm。

### Min Prior Queue

Prior Queue ( 優先佇列 ) 在進行需要提取最大或最小元素的演算法中十分有用，除了 DijKstra's Algorithm 外，還有許多演算法會建議使用 Prior Queue 來配合。

以下使用 Cormen 的 "Introduction to Algorithm" 一書中的圖例來更了解整個 DijKstra's Algorithm 的 implementation  : 

![](https://i.imgur.com/HCOBZQp.png)


