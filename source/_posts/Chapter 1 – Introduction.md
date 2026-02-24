---
title: Chapter 1 -- Introduction
date: 2019-09-30 22:10:50
categories:
- 課程筆記 Course
- 江蕙如 Algorithm
image: https://blog.techbridge.cc/img/kdchang/cs101/algorithm/algorithm-cover.png
mathjax: true
---


演算法 Algorithm 的定義 : 
*A finite, definite, effective procedure, with some output. -- Donald Knuth,1968.*

*A well-defined procedure for transforming some input to a desire output. -- Cormen et al. Introduction to Algorithm*

整個演算法的分析步驟如下 : 
1. 遇到自然且實際的問題待解決
2. 使用簡潔的方式描述問題
3. 找出一套演算法可以清楚、簡單的解決問題
4. 證明這套演算法是正確的，並且解決問題所花的時間必須在可接受的範圍內。

<!-- more -->

Stable Matching Problem
---

問題描述 : 

現在有 n 位男性與 n 位女性，雙方都清楚了解對方的情況下，要求每一位男( 女 )性都要依照自己的喜好對 n 位異性做出一個 ranking list 排序 ( 無平手狀況 )。而我們現在的工作就是要將雙方進行配對，而配對的條件是 : 
(1) Perfect matching : 一對一配對，不會有落單的人沒有被配對到
(2) Stable Matching : 被配對的雙方都是 Happy and Stable，意即，不存在一對男女互相喜歡對方更勝過於我們幫他們配對的對象。


Random Matching and Fixing up
---

Step 1 : 先進行隨機配對
Step 2 : 當存在 unstable pair ，藉由交換配對來進行修正

這樣的方式會有幾個問題，我們利用交換配對來進行修正，很有可能解決了某一對 unstable 狀況，但卻另外造成其他的 unstable。這樣的方式很容易造成無限迴圈。


Gale & Shapley Algorithm  -- Propose and Reject
---

Step 1 : 男性向女性 Propose
Step 2 : 女性決定要 Accept 還是要 Reject

```=
function stableMatching {
    Initialize all m ∈ M and w ∈ W to free 
    # 初始化所有人均為單身
    while ∃ free man m who still has a woman w to propose to {
       w = first woman on m's list to whom m has not yet proposed
       # 男生要進行 Propose 必然會先從名單上最高分的女性進行 Propose
       if w is free
         (m, w) become engaged
       # 如果女性單身，則配對
       else some pair (m', w) already exists
         if w prefers m to m'
            m' becomes free
           (m, w) become engaged 
         else
           (m', w) remain engaged
       # 若女性已被有配對，則比較兩位男性對於這女性的名單 ranking　高低來決定與誰配對
    }
}
```

這個演算法有一個特別的地方在於中間會有 Deffered Decision Making 的過程，簡單來說就是男女雙方都可以有一個猶豫期來決定這個配對要繼續還是解除。待所有配對都進入這個猶豫期且不變後，才會正式確定全體配對。

在這樣的演算法下，男性主動 Propose ,而女性被動的選擇是否要 Reject。對男性來說，最後的結果必然是最優化的結果，然而對於只能被動接受或拒絕的女性就未必會是最優化的結果。


### 1. Convergence of Gale & Shapley Algorithm
**Claim : Gale & Shapley Algorithm terminates after at most $n^2$ iterations.**

**<proof>**

我們要看 while loop 最多執行多少次，最直覺的方式就是看 free men 個數來決定，但這樣會非常難算，因為每一次的 iteration 都可能更動前幾次的狀態，使得男生可能在 free 或是 engaged 狀態跳來跳去。

換一個角度來看，每一個 while loop 都一定會有男性向女性提出一個 propose，所以我們直接算最多可能有幾種 propose 可能。

每一位男性最多可以提出 $n$ 個 propose，總共又有 $n$ 個男性，因此整個 while loop 最多可以執行 $n^2$ 次。



### 2. Correctness of Gale & Shapley Algorithm

這裡先提出幾個對於 Gale & Shapley Algorithm 的觀察 : 
* Observation 1 : 男生必然會從其 ranking list 最高分的女性開始 propose
* Observation 2 : Free woman 必然會接受第一個 propose
* Observation 3 : 男性交換配對的方向是一路從 high ranking 換到 low ranking，但女性則是從 low ranking 一路換到 high ranking


**Claim : Gale & Shapley Algorithm return a perfect matching.**

**<proof>**

假設 Gale & Shapley Algorithm 最後會 return 一個 free man M。
在此假設下也必然至少存在一位 Free woman W，仍然沒有被 Proposed。從 observation 2 可以確定此 W 從來沒有被 proposed，當然包括 M 。
從上述兩個狀態來看，Gale & Shapley Algorithm 會繼續執行 while loop 不會 return 。( $\rightarrow\leftarrow$ )


**Claim : Gale & Shapley Algorithm return a stable matching.**

**<proof>**

假設 Gale & Shapley Algorithm 最後會 return 一個 unstable matching，意即，存在至少一對男女喜歡對方更勝於自己的配對。

存在 instable pairs $(m,w)$ 與 $(m',w')$，且 $m$ prefers $w'$ to $w$ and $w'$ prefers $m$ to $m'$


![](https://i.imgur.com/D4qCw7T.jpg)


我們可以知道，在 Gale & Shapley Algorithm 執行過後的配對會是最後一次 propose 的結果 ( 不管前面經過多少次配對或解開配對 )。

Case 1 : $m$ 不曾 propose $w'$
$m$ 最後停在 $w$ 的配對狀態，且未曾與 $w'$ 配對過，則可確定在 $m$ 的 ranking list 上 $w$ 的名次必然在 $w'$ 之前，也就是 $m$ prefers $w$ to $w'$ ( $\rightarrow\leftarrow$ )

Case 2 : $m$ 曾經 propose $w'$
既然 $m$ 曾經 propose $w'$，但最後卻與 $w$ 配對，則可確定當時 propose $w'$ 時被拒絕，這樣也表示 $w'$ 喜歡某人( 可能是 $m'$ 也可能是另外一個 $m''$ )勝過 $m$。
若 $m'=m''$ ，則 $w'$ prefers $m'$ to $m$ ( $\rightarrow\leftarrow$ )
若 $m'\neq m''$ ，則 $w'$ 喜歡 $m'$ 勝過 $m''$，根據Case 2的推導， $w'$ 喜歡 $m''$ 又勝過 $m$，那麼 $w'$ 喜歡 $m'$ 必然勝過 $m$ ( $\rightarrow\leftarrow$ )

### 3. Male-Optimality of Gale & Shapley Algorithm

**Claim : Assume $$w^*=best(m)$$ is the best valid partner of m $\Longrightarrow$ Gale & Shapley Algorithm return solution $$S^* =\left\{(m, w^*)\ |\ \forall m\right\}$$ .** 

對每一個男性 m 來說，一定會存在一些女性是可能的配對對象，之所以稱之為「可能」，意即在這樣的配對下是有可能達到全體 stable matching 的狀態。而在這個 property 下，可以確定每一個男性最後配對的對象必然是可能對象中最好的那一個。而這個 propertｙ　也間接證明了，Gale & Shapley Algorithm　return 的必是唯一解[^註1]。


**<proof>**

假設某次執行 Gale & Shapley Algorithm $E$ 時 solution $S_E$ 中存在一組配對女方並非男方的 Best valid partner。

$\Longrightarrow$ 既然女方並非男方的 Best valid partner,則可確定這配對中的男生，一定有被自己的 valid partner 拒絕的經驗

$\Longrightarrow$ $W.L.O.G$ 假設 $m$ 是所有男性中第一個被 valid partner 拒絕的男性，則在 $S_E$ 中  $(m,w)$， $w\neq w^*$

$\Longrightarrow$ $\exists$ a valid partner $w'$ s.t. $m$ 被 $w'$ 拒絕過 ，從 observation 3 可以確定第一個拒絕 $m$ 的 $w'=w^*$

$\Longrightarrow$ $\exists$ $m'$ s.t. $w^*$ 喜歡他更勝於 $m$ $\cdots\cdots( 1 )$ 

$\Longrightarrow$ $$w^*$$ 是 $m$ 的 valid partner，那麼必然在某一次執行 $E'$ 中的結果 $S_{E'}$ 是 stable matching ，且 $(m,w^*)\in S_{E'}$，而且在 $S_{E'}$ 之中一定有一個 $w''$ 與 $m'$ 配對 $(m',w'')$

$\Longrightarrow$  $m$ 之所以被 $$w^*$$ 拒絕，是因為 $m'$ 的緣故。也就是說在 $m$ 對 $$w^*$$ propose 以前，$m'$ 已先對 $$w^*$$ propose。前面我們假設 $m$ 是第一個被 valid partner 拒絕的 Case，因此我們可以總結一下 : 
1. 在 $m$ 對 $$w^*$$ propose 的當下 ，$$w^*$$ 已經先和 $m'$ 配對，所以才會拒絕他
2. 在 $m$ 對 $$w^*$$ propose 之前，$m'$ 並未被任何的 valid partner 拒絕過 。( 因為 $m$ 被 $$w^*$$ 拒絕是第一個 case )

$\Longrightarrow$ 從 $m'$ 的 ranking ist 來看，排在 $$w^*$$ 之前的都必然是 invalid partner，又 $w''$ 是他的 valid partner，所以 $w''$ 的 ranking 一定比 $$w^*$$ 低

$\Longrightarrow$  $m'$ 喜歡 $$w^*$$ 更勝於 $w''$ $\cdots\cdots( 2 )$

從  ( 1 ) ( 2 ) 來看，在 Stable Matching solution $S_{E'}$ 中會存在一個 instable 的狀況 ( $\rightarrow\leftarrow$ )，得證。


註釋
---

[^註1]: 這個部分有個邏輯性的問題待釐清，Stable Matching 的解並不會是唯一解，但 Gale & Shapley Algorithm 會吐出的是對男性來說最優的解，也就是 Gale & Shapley Algorithm 的解是唯一的。
