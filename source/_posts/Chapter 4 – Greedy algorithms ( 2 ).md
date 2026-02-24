---
title: Chapter 4 -- Greedy algorithms ( 2 )
date: 2019-11-11 01:05:32
categories:
- 課程筆記 Course
- 江蕙如 Algorithm
image: https://blog.techbridge.cc/img/kdchang/cs101/algorithm/algorithm-cover.png
mathjax: true
---

Interval Partition
---

### Scenario
$Let\ \{I(i)=\big[s(i),f(i)\big)\mid i=1,2,3,\cdots,n\}\ is\ a\ set\ of\ requests\ interval.$ 
$Partition\ these\ requests\ into\ a\ minimum\ number\ of\ compatible(no\ overlap)\ subsets.$
給定一個區間，以及 $n$ 個子區間，試圖將這個子區間分成最少的子集合，子集合內的子區間互不重疊。

<!-- more -->

![](https://i.imgur.com/aXOAugj.png)




要處理這個問題前，我們先定義 $depth$ 這個名詞。
$depth$ 指的就是在這些 requests 中，在同一個時間點上最多有幾個 requests 重疊。

（這邊我認為課程中江蕙如教授並沒有講得很清楚，究竟 $depth$ 指的是同一時間最多重複的 requests 數量? 還是就整個時間軸來看重複率最多的那個重複數量? 舉例來說，如果整個時間軸掃過去，有一個時間點出現了 4 個requests 重複，但有四個時間點出現了 3 個 requests 重複，那 $depth$ 應該是 3還是4 ?就後面的討論來看，$depth$ 應該是 4）

**Claim : $depth$ is the least number of partition subset**

**Proof :**

Suppose that $depth=d$
$\therefore \exists I_1,I_2,\cdots,I_d$ ，這些子區間會在某一個時間點互相重疊。
$\Longrightarrow$ 這些子區間就必須要被分配到不同的 subsets 中。
$\Longrightarrow$ 我們至少就需要 $d$ 個 subsets。

現在我們有幾個問題 : 

* 我們能不能用 $O(n\log n)$ 的複雜度找出 $d$ ?
* $d$ 是我們想要找的答案的下界，那有沒有可能 $d$ 就是答案 ? 怎麼找 ?

如果這兩個問題都能解決，那我們勢必可以確定這樣找出來的答案 $d$ 一定是最優解。


**如何用 $O(n\log n)$ 的時間複雜度找出 $depth$ ?**

```
1. 首先將所有子區間按照 start time 進行排序
2. 製造一個序列 S，是將 start time 與 finish time 混合排序
3. 初始化 d=0 , d-max=0
4. 依照 S 中的順序，遇到 Start time 則 d=d+1，遇到 finish time 則 d=d-1
   並且將至今遇到的最大 d 保存在 d-max 中
```

整個過程大概可以用下圖來表示

![](https://i.imgur.com/ve1UgXu.png)

這樣的過程我們可以計算總時間複雜度 : 

Step 1. $O(n\log n)$
Step 2. $O((2n)\log (2n))=O(n\log n)$
Step 3. $O(1)$
Step 4. $O(n)$

總時間複雜度則為 $O(n\log n)$，而且整個複雜度不與所佔的時間總長度相關，即使今天有 request 站了一百萬個時間單位都不影響整個複雜度。


**$d$ 就是最小的 partition number of subsets**

```
Interval-Partitioning(R):
1.將所有子區間針對 start time進行排序 = {I(1),I(2),...,I(n)}
2.找出 depth = d，並且創造一個 Label set L = {L(1),L(2),...,L(d)}
3.for j from 1 to n :
    排除所有與 I(j) 衝突的 Label
    if 存在不與 I(j) 衝突的 L(i) : 
        將 I(j) 配給 L(i)
    else 先擱置 I(j)
```
我們一樣用動畫來呈現整個流程較為清楚 : 

![](https://i.imgur.com/clmNUV5.gif)

### Correctness of Algorithm

#### 每一個子區間都一定會有 Label

仔細看看上面的 procedure 其實最後一步的 else 似乎不會碰到 ? 的確，這是整個演算法的一個防呆措施，但其實根本不會有「配不到 label」 的情況發生。

假設我們在整個演算法過程中檢查到 $I(i)$ 這個子區間，而它與前面的 $t$ 個子區間都有重疊到。根據我們一開始找出來的 $depth\ d$ 我們可以知道 $t+1\leq d$。 
( $I(i)$ 跟前面 $t$ 個子區間重疊的深度一定不會超過 $d$ )

$\Longrightarrow t\leq d-1$
$\Longrightarrow$ 檢查到 $I(i)$ 時一定至少有一個 label 可以跟它搭配
$\Longrightarrow$ $I(i)$ 一定可以配到 label，因此 else 的步驟一定不會發生。

#### 每一個 subset 內的子區間必然不會 overlap

$\forall$ overlapping $I(i)$ and $I(j)$ 
W.L.O.G.  ,$i<j$

從我們提出的演算法來看，當我們在 $I(j)$ 這個 iteration 時，$I(i)$ 已經被處理過了，而且也已經有了一個 Label。此時，在這個 iteration 中的第一步，我們會先排除掉所有衝突的 Label，而 $I(i)$ 的 Label 勢必會被排除，因此 $I(j)$ 也絕對不會跟 $I(i)$ 配在同一個 Label 裡面。

從以上的說明發現，這樣的演算法一定可以做出一個 $d$ 個 partition subsets ，而且這樣的結果一定是最優解。


Scheduling to Minimize Lateness
---

### Scenario
$Let\ \{I(i)\ with\ lenth\ t(i)\ and\ deadline\ d(i)\mid i=1,2,3,\cdots,n\}\ is\ a\ set\ of\ requests\ interval.$ 
$Scheduling\ these\ requests\ without\ overlapping\ and\ minimize\ maximum\ lateness.$
$Lateness : I(j)=max\{0,f(i)-d(i)\}$

給定一個區間，以及 $n$ 個子區間，這些子區間沒有限定 strat time 及 finish time，但有限制其長度及 deadline，試圖將這個子區間安排順序使這些子區間完全不重複，並且盡可能使延遲 ( finish time 超過 deadline) 最小化。


![](https://i.imgur.com/aBttlPR.png)

### Criterion ( Greedy Rule )

我們這個問題也一樣先找出 Greedy Rule : 

1. 最短區間優先 ?
若有兩個區間 $I_1$ : deadline=100, lenth=1 ; $I_2$ : deadline=10, lenth=10，若以最短區間優先，則我們會先排序 $I_1$ 再來排序 $I_2$，但這樣很明顯地會產生 Lateness=9。但我們排序的方式為 $I_2$, $I_1$ ，則 Lateness=0。
2. 最短餘裕優先 ?
若有兩個區間 $I_1$ : deadline=2, lenth=1 ; $I_2$ : deadline=10, lenth=10，我們以最短餘裕優先的方式進行，排序的方式會是 $I_2,I_1$，產生的 Lateness=9。但若我們排序方式改為 $I_1,I_2$ ，則 Lateness可以壓到 1。
3. Deadline 最早優先 ?

相較於 Interval Scheduling 的 Greedy Rule，這個 Greedy Rule 相對來說比較貼近直覺。

```
Min-Lateness(R,s):
// f=solution schedule finish time ; s=start time of the schedule
1. 先將所有子區間依照 deadline 進行 ascending 排序
2. 初始化 f = s
3. for i from 1 to n do
      將 I(i) 分配到 schedule 中，s(i)=f, f(i)=f+t(i)
      f=f(i)
4. return { (s(i),f(i)] | i=1,2,...,n }
``` 

這樣的演算法中，我們並不限定必須要從 time=0 開始進行 scheduling，所以我們可以任意給定一個 start time s。

再來比較重要的一個概念是，這個演算法可以確保最後輸出的 solution 中間不會有閒置時間 ( idle time )，但是事實上，一個合理的 optimal solution 是可以容許有 idle time 的。


### Correctness of Algorithm

**Claim : 一定存在一個最優解中間沒有 idle time**

假設我們手中有一個含有 idle time 的 optimal solution，那麼顯而易見的，我們可以將這接閒置時間進行壓縮，讓每一個子區間的頭尾相連，這樣的解也一定會是 optimal solution。

**Claim : 任何沒有 inversion 及 idle time 的 schedule，必然擁有相同的 Maximum Lateness**

首先我們先定義 inversion : 如果在 schedule 中存在兩個子區間 $I(i),I(j)$ 滿足 $s(i)<s(j)$ 但 $d(j)<d(i)$ 者稱之。白話一點說，一般我們會按照 deadline 來安排順序，deadline 比較早的，我們就會比較早排序，它的 start time 也會比較早，但 inverstion 就是排列順序與 deadline 順序相反的子區間。

兩個沒有 inversion 及 idle time 但卻不同的 schedule 中，不同的部分只會出現在 「相同 deadline,卻不同 start time」的子區間中。如果真的有可能會出現不同的 Lateness 也會出現在這部分。

因此，我們現在只考慮這樣子的 deadline $d$，以及有相同 deadline $d$ 的這些區間。

因為這些區間最後排序的 start time 與總長度勢必都一樣，但又具有相同的 deadline $d$，所以不管最後一個被排序的子區間是哪一個，它的 finish time 與 deadline $d$ 的時間差一定會一樣。

![](https://i.imgur.com/TvOlXhh.png)


**Claim : 一定有一個 optimal solution 沒有 idle time 及 inversion**

(1) 一定存在 optimal solution 沒有 idle time : Clearly.
(2) 一定存在 optimal solution 沒有 inversion

假設現在有一個帶有 inversion 的 optimal solution，我們只需考慮出現 inversion 的區間 finish time 都在它們的 deadline 之後。

( 如果 finish time 都在它們的 deadline 之前，就沒有 Lateness 的問題，我們只要直接將兩個區間調換位置即可，不會影響 optimal solution )

考慮到上述的狀況，當我們針對 inversion 的子區間進行調換時，Lateness 勢必會改變，但絕對不會大於原本 inversion 的 Lateness。

![](https://i.imgur.com/r8UcPtV.png)

所以如果原本已經是 optimal solution，經過這樣的子區間調換後，他也依舊會是 optimal solution。

**Theorem : The Min-Lateness(R,s) Algorithm schedule S is optimal.**

假設，我們可以找到一個 optimal schedule $O$
$\Longrightarrow$ $Max-Lateness(O)$ is minimal.
W.L.O.G. 假設 $O$ 無 idle time

(Case 1.) 如果 $O$ 無 inversions : 
那麼 $O$ 與 $S$ 同樣都是沒有 idle time 且不具 inversions 的 schedules，則兩者具有相同的 max lateness，$\Longrightarrow Max-Lateness(S)=Max-Lateness(O)$ is minimal.

(Case 2.) 如果 $O$ 有 inversions : 
我們可以經由不斷的 inversion 區間的對調產生一個 Schedule $O'$，但從之前的證明過程我們可以知道 $Max-Lateness(O')\leq Max-Lateness(O)$
$\Longrightarrow O$ is not optimal. $(\rightarrow\leftarrow)$

補充
---

### Interval-Graph

前面介紹的三種問題型態，都跟 interval 有關，而其中 interval-partition 又跟 graph 有某種程度上的關聯性。那我們能不能將 interval 問題 轉換為 graph 問題 ?

事實上是可以的。我們將每一個 interval 都視為一個 node，如果有 overlap 的兩個 node 我們就給予一條 edge，這樣便可以將interval 問題 轉換為 graph 問題來去處理。

然而，並非所有 interval 問題都可以藉由 graph 來簡化問題，一般我們在生活中遇到的問題都是複雜且困難的，要想利用 graph 配合 greedy algorithm 來處理基本上都是不可行。

但是，簡單的問題上，或許我們可以從這樣的角度來嘗試看看，畢竟 greedy algorithm 的想法很直覺而且比較簡單。


### 從時間管理 ( Time-management ) 角度來看 interval 問題

![](https://i.imgur.com/KQTYAjJ.jpg)

在上圖中，你會怎麼安排你的工作優先順序呢 ?

一般人的做法 (包含我) 或許第一時間是這樣安排的 : 

$$
I > III > II > IV 
$$

但在一般時間管理的角度來看，這樣的安排比較適當 : 

$$
I > III > IV > II 
$$

差別在於 $II$ 跟 $IV$ 的順序不同。
時間管理專家 (我沒聽清楚名稱) 認為，如果先從事輕且急的工作，把重且緩的工作排程放在後面，最後這些重且緩的工作會演變成為又重又急的工作，這將在後面造成非常大的時間壓力。