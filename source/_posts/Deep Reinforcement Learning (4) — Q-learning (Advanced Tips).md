---
title: "Deep Reinforcement Learning (4) --- Q-learning (Advanced Tips)" 
date: 2019-12-05 12:13:28
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning and having it Deep and Structured
image: https://i.imgur.com/MdVC5JZ.png
mathjax: true
---


這一個部分延續上一篇文章，對 Q-Learning 的一些技巧做了更深入的延伸。

<!-- more -->

Tips of Q-Learning (Advanced)
---

### Double DQN - 解決偏高的 Q-value

一般的 DQN ( Deep Q Network )，更新 $\pi$ 的方式是找出可以得到最大 Q 值的 action $a$ 來做替換，這樣的概念直觀但卻可能使 Q 值高估了 expect cumulated reward。

為什麼會高估 ? 主要是因為我們每一次選取的 action 都是可以得到最大 Q 值的 action，但 DQN 是一個網路結構，每一個 action 的 Q 值跟真正的 expect cumulated reward 本來就可能會有誤差，當我們要選擇會形成最大 Q 值的 action 其實也就是傾向於選擇高估 expect cumulated reward 的那個 action，一次又一次的都是選擇這樣高估 expect cumulated reward 的 aciton，所以我們繪製 DQN 的 Q-value 與 training steps 圖表時，DQN 的曲線通常會呈現偏高且遞增的圖形。

以下就是利用 DQN 與 Double DQN 玩六種遊戲所繪製的圖 : 

![](https://i.imgur.com/i2PaLDX.png)

Double DQN 其實就是一種 Double check 的機制

$DQN$

$$
\mathcal{L}(s_t,a_t)=Q(s_t,a_t)-\big[r_t+\max\limits_{a}Q(s_{t+1},a)\big]
$$

$Double-DQN$

$$
\mathcal{L}(s_t,a_t)=Q(s_t,a_t)-\big[r_t+Q'\big(s_{t+1},\arg\max\limits_{a}Q(s_{t+1},a)\big)\big]
$$

Double-DQN 利用了另外一個 $Q'$ (實務上其實就是使用固定住的那個 target function) 來重新計算 Q 值。如果我們真的 Q 高估了 expect cumulated reward，但只要 Q' 計算時沒有高估，便可以平衡調前面高估 expect cumulated reward 的狀況，反之亦然，假設 Q' 有高估 expect cumulated reward ，但只要 Q 沒有選到會高估 expect cumulated reward 的 action 就也是可以平衡掉影響。




### Dueling DQN - 更有效率的 action 更新

我們在進行一般 DQN 訓練時，往往都是對所有的 action 做取樣，然後針對被取樣的這些 action 來更新網路參數來讓 $Q(s_t,a_t)$ 盡可能接近 $r_t+\max\limits_{a}Q(s_{t+1},a)$。

但這樣的更新事實上很沒有效率，如果沒有被取樣到的 action 就不會被考量到一起做參數更新。而 Dueling DQN 便是變更 DQN 的結構，來讓每一次的抽樣後，會連未被抽樣到的 action  一起考慮進去來做參數更新。

![](https://i.imgur.com/zMaKB2Y.png)

上圖上就是一般的 DQN 結構，輸出 $Q(s,a)$，$Q(s,a)$ 的每一個維度就是每一個 action 的 expect cumulated reward 。
上圖下則是 Dueling DQN，在最後幾層分成兩個通道，一個通道輸出一個 scalar $V(s)$，而另外一個通道則是輸出一個 $A(s,a)$，最後再讓這兩個通道相加 ( $A(s,a)$ 的每一個維度都加上 $V(s)$ ) 成為 $Q(s,a)$。

Dueling DQN 這樣的方法，如果我們只有抽樣到某幾個 action，並且期待這幾個被抽樣出來的 action expect cumulated reward 可以逼近 Target value，但由於 $Q(s,a)=A(s,a)+V(s)$，當我們要調整 $Q(s,a)$ 的某幾個維度，我們希望可以藉由 $V(s)$ 對所有維度都統一進行調整。

這裡要注意的是 $A(s,a)$，為了要利用 $V$ 來調整 $Q$，我們必須對於 $A$ 必須要有所限制，不然的話很可能訓練到最後 $V=0$ 且 $A=Q$，或者，$A$ 的變化太靈活，最後 $V$ 不動，反而是 $A$ 中相對應被抽樣 action 的維度在改變而已，失去了 Dueling DQN 的意義。在論文中的限制便是 $A$ 的每一個維度總和必須為 $0$。

在實作上，這兩個通道其實只需要在一般的 DQN 網路中稍微改一下即可，而對於 $A$ 的限制也只是在這個通道的最後加上一個 normalization 。


### Prioritized Reply - 資料有其優先性的處理

當我們從 buffer 裡面取樣資料時，會以一個 uniform distribution 的想法來抽樣，每一筆資料的機率都是相等的。但在現實中，很有可能會對於某些資料的需求更高，我們會希望有更高的機率可以抽樣到這些資料，因此會在這些資料上給予一定的 priority。


### Multi step - TD 與 MC 的權衡

這個方法其實是在 TD method ( 一筆資料是 $(s_t,a_t,r_t,s_{t+1})$ ) 與 MC method ( 一筆資料是 $\{(s_i,a_i,r_i,s_{i+1})\}_{i=1}^T$ ) 中取一個平衡點。我們不再只讓 policy 跟 environment 進行一次互動而已，我們一筆資料是 $N$ 次互動的資料 : $\{(s_i,a_i,r_i,s_{i+1})\}_{i=t}^{N+t}$

在這樣的情況下， Loss Function $\mathcal{L}$ 也必須有所調整 : 

$$
\mathcal{L}(s_t,a_t)=Q(s_t,a_t)-\big[\sum\limits_{i=t}^{N+t}r_i+\max\limits_{a}Q^{target}(s_{N+t+1},a)\big]
$$




### Noisy Net - Epsilon Greedy 的延伸

上一篇文章我們在講 Epsilon Greedy 的說法是，為了讓其他的 action 也必須要有一些機率被取樣到，給予 DQN 能有更強的探索力。換一個角度來說，我們也是對於 action 加了一定程度的 noise。

但在 Noise Net 中，我們仍然是給予一些 noise，但是是加在 DQN 的參數中，當 policy 要跟 environment 進行互動前，就先 sample noise 加進DQN 參數中再來進行 $Q^{noise}$ 的估算。 

但其實從現實面來說，Noise Net 的概念會比 Epsilon Greedy 更貼近現實。當我們使用 Epsilon Greedy，在每一個 state 中所採取的 action 會由 $\epsilon$ 來決定抽樣的機率，這顯然不是太合理。正常我們在與環境作互動時，應該要看見同一個 (或類似) 的 state 會有同樣的 action 才對。因此，為了讓 noise 能存在，又可以在每一個 state 上可以有相同的 action 選擇，便是將 noise 加入 DQN 結構中。

此外也要注意，必須在同一個 episode 中使用同一個 $\epsilon$，才能確保在這整個 episode 中， policy 會有統一的 action 選擇策略。因此，要重新 sample $\epsilon$ 就必須在結束一個 episode 後才做 sample。


### Distributional Q-function - 探索 Q 的特性

最後這兩段，在課程中都僅有非常簡單的介紹，因此我便也簡單的描述一下即可。

原本的 Q-function 輸出的 Q 值，其實就是 Q 值分佈的平均 ( 期望值 )，但對於同一個 Q 值來說，可以有許多種 Q 分佈的可能性。

假如我們將 Q 值的輸出限制在一個範圍之間，那我們可以針對每一個 action 找出對應的 Q值分佈，最後我們選擇 action 就是看哪一個分佈的平均值最高就選哪一個 action 。

感覺有點繞了一大圈，再回到原點。但其實這一大圈給了我們一些資訊，譬如說，我們可以知道每一個 action 的 Q 值分佈後，我們便可以計算出其 variance，當兩個 action 的 Q 值均值差不多，或許我們可以選擇 variance 較小的來規避掉一些風險可能性。



### Rainbow

簡單來說就是結合上述方法的一個組合式方式。

![](https://i.imgur.com/FjrTXjX.png)

上圖左可以看出 Rainbow 的整體優越性，但上圖右藉由一個一個演算法從 Rainbow 中抽出來看看在 Rainbow 中真正有比較決定性的演算方法是哪一個。