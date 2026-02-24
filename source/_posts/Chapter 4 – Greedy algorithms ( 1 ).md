---
title: Chapter 4 -- Greedy algorithms ( 1 )
date: 2019-11-05 00:31:21
categories:
- 課程筆記 Course
- 江蕙如 Algorithm
image: https://blog.techbridge.cc/img/kdchang/cs101/algorithm/algorithm-cover.png
mathjax: true
---


Greedy algorithm
---

當我們在設計演算法時，往往都是一個一個步驟累積而成，而每一個步驟往往都必須要對某些狀況做決定。Greedy Algorithm 在每一個步驟都選擇「當下」「最好」的決定。也就是說，Greedy Algorithm 在每一步驟下都必須要去試著最優化我們所訂定的規範 (criterion)。而且，這樣的演算法不會回頭、不會反悔、不會猶豫。

<!-- more -->

這樣短視近利的方式，直覺且快速，因此適用於各種問題上。雖然說在某些問題上，Greedt Algorithm 可能可以得到最佳解，但大部分狀況，solution 都不會是最好的。

因此如何證明 Greedy Algorithm 在這一小部分問題上的確可以得到最佳解，將是最大的挑戰之一。一般來說，我們可以利用下列兩種方式來進行證明 : 

1. The Greedy Algorithm stays ahead.
利用 Greedy Algorithm 可以一路最佳化。
2. The exchange argument.
預設有一個最佳解，將其中與 Greedy Algorithm 不符的 criterions 交換，試圖證明這樣交換後仍然可得最優解。

Interval Scheduling
---
### Scenario
$Let\ \{I(i)=\big[s(i),f(i)\big)\mid i=1,2,3,\cdots,n\}\ is\ a\ set\ of\ requests\ interval.$ 
$Find\ a\ compatible(no\ overlap)\ requests\ with\ maximim\ size.$ 
給定一個區間，以及 $n$ 個子區間，試圖在這個區間中找出最多不重複的子區間。

![](https://i.imgur.com/dGLn0Si.png)

### Criterion ( Greedy Rule )

前面有提到，Greedy Algorithm 在每一步驟下都必須要去試著最優化我們所訂定的規範 (criterion)，在 Interval Scheduling 問題來說，就是先選出一個在當下比較「好的 interval」，刪除掉 overlap 的選擇，之後再選擇一個「好的 interval」.....，一值重複下去。

或許我們可以思考一下，最好的 interval 的可能有哪些 : 

1. 最早開始 ?
2. 最短區間 ?
3. 最少衝突 ?
4. 最晚結束 ?



<img width=500 src="https://i.imgur.com/fUZ4NHd.png" >


上面四種方式，最直覺的大概數「最少衝突」這一種狀況，在這並非最優解。Greedy Algorithm 在選取條件上，往往不是那麼直覺，這也是比較需要花時間思考的部分。

最早結束的 interval 才是每一輪選擇的最好條件，因為大區間的起始固定，因此越早結束的子區間也強制其長度，另一方面也可以越早將資源放出來給其他子區間來使用。

```
Interval-Scheduling(R)
# R是當下可選擇的requests set,而 A 是我們選擇的 requsts set
1. A=∅
2. while R is not empty then do
       choose I(i) with minimum f(i)
       A=A+{I(i)}
       X={ I(j)| I(j) is not compatible with I(i) }
       R=R-{I(i)}-X
3. return A
```

Greedy Algorithm 在演算法中很容易實現，主要也是因為其架構幾乎都是一個 loop。

### Correctness of Algorithm

**Compatible**

因為我們每一次個 loop 都會刪除 overlap 的選擇，再去下一個 loop 中挑選一個 request 放進 $A$ 中，因此我們可以確定 $A$ 中的 requests 必然都是 compatible。

**Maximum set**

Cliam : $O$ is an optimal solution, then $|A|=|O|$

(1) 這裡我們先提出一個引理 : 
$A=\{I(i_1),I(i_2),\cdots,I(i_n)\}$ and $O=\{I(j_1),I(j_2),\cdots,I(j_m)\},where\ f(j_u)\leq f(j_v)\ when\ u\leq v\Longrightarrow \forall\ k\leq n,\ f(i_k)\leq f(j_k)$

Step1. $k=1\Longrightarrow f(i_1)\leq f(j_1)$ 成立
($\because i_1$ 是所有 requests 中結束時間最早的)

Step2. 假設 $k=r$ 時亦成立 $\Longrightarrow f(i_r)\leq f(j_r)$
$\because I(j_r)$ 與 $I(j_{r+1})$ 為 compatible ，且 $I(i_r)$ 與 $I(i_{r+1})$ 為 compatible
$\therefore I(i_r)$ 與 $I(i_{r+1})、I(j_{r+1})$ 均為 compatible
$\Longrightarrow$ 在 $r+1$ 這個 loop 中，$R$ 裡面同時存在 $I(i_{r+1})、I(j_{r+1})$
$\Longrightarrow f(i_{r+1})\leq f(j_{r+1})$

(2)
Suppose to the contrary that $A$ is not optimal solution.
$\Longrightarrow |A|=n<m= |O|$
$\Longrightarrow \exists\ I(j_{n+1})\in O\subseteq R$ s.t. $I(j_{n+1})$與$I(j_{n})$ compatible， 且$f(i_n)<s(j_{n+1})$
$\Longrightarrow$ 當 $I(i_n)$ 被挑出來放進 $A$ 後，$R$ 裡面至少還有一個 $I(j_{n+1})$，整個 loop 不會結束。
$\Longrightarrow |A|\geq n+1$ ($\rightarrow\leftarrow$)


### Complexity

我們在進行上面的演算法時，由於我們必須要比較 finish time，所以我們會先對 $R$ 做一次排序，複雜度為 $O(n\log n)$ (課程還沒講過這部分，暫時先了解即可)。

之後我們每一次 loop 挑出排序後的 $R$ 最前面的 interval 後，都必須要重複查找 $R$ 中是否有 overlap 的 intervals 進行刪除，總共的複雜度是 $O(n^2)$。(裡面必然有一些 interval 會在不同 loop 中被重複查找)

由於排序的複雜度不可避免，我們便想這邊利用一些方式來減少整個 loop 的複雜度，讓整個演算法複雜度壓到只有排序這邊產生的 $O(n\log n)$ 。

概念就是 **每一次對 $R$ 進行 overlap interval 刪除時，不要全部刪除**

* 先建立一個 $S$ 用來儲存每一個 interval 的 start time。
* 每一輪從 $R$ 挑出最前面我們要的 interval $I(i_k)$ 後，按照 $R$ 排序，僅刪除 $s(i_l)<f(i_k)$ 的 interval $I(i_l)$，遇到 $s(i_l)>f(i_k)$ 即停止。這樣可以確保所有的 intervals 在全部 loop 結束時只會被查找過一次。也就是說，整個 loop 查找完所花的時間複雜度只有 $O(n)$。

下列動畫可以試著解釋這整個演算法流程 : 

![](https://i.imgur.com/mOdtdlC.gif)

這樣不但可以減少每一個 loop 的檢索時間，還可以確保前面該刪除沒刪除的 intervals 一定會在後面的 loop 被刪掉。

藉由這樣的方式，我們便可以將整體的時間複雜度壓到只剩下 $O(n\log n)$。





