---
title: "Deep Reinforcement Learning (7) --- Sparse Reward" 
date: 2020-04-21 08:56:14
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning and having it Deep and Structured
image: https://i.imgur.com/MdVC5JZ.png
mathjax: true
---

我們在進行 Reinforcement Learning 的過程，其實很常會遇到的狀況就是，大部分的時間都沒有 Reward，在這樣的狀態下，機器只能一直隨機的選擇 Action，這樣其實不會學習到任何東西。

以下介紹三種方式，可以讓機器從 0 開始學習，最後可以達到目標。

<!-- more -->

Reward Shaping
---

Reward Shaping 其實非常直接，在一開始到真正得到 Reward 之前，我們人工幫機器設計一些額外的 Reward，讓機器可以在學習的過程中一直都有得到 reward，然後逐漸地藉由這些設計好的、非環境給予的 reward 來引導機器達到最後的目標，進而獲得最終的 reward。

在 OpenAI 研究團隊所提出的論文 " *Training Agent for First-Person Shooter Game with Actor-Critic Curriculum Learning* " 中，利用 Reinforcement Learning 來玩 VizDoom 這個射擊遊戲。

![](https://i.imgur.com/MJDh15h.png)

在論文中，設計了一些 reward 在遊戲過程如下 : 

![](https://i.imgur.com/ufzJxBy.png)

當失血、彈藥耗損、躲起來不動、一直存活都會有負 reward 的發生，而當撿到補給品跟彈藥或是移動都會有正 reward 的獲得。依照這樣的 reward 設計，可以讓機器成為一個具有攻擊力、積極探索的射手。

然而要如何設計這些額外的 reward ，需要的其實就是該領域的 domain knowledge，唯有融入領域的專業知識才不會讓整個 reward 設計反而弄巧成拙。

### Intrinsic Curiosity Module ( ICM )

一種實現 Reward Shaping 的方式就是利用  Intrinsic Curiosity Module function，當我們給予 state $s_t, a_t, s_{t+1}$，ICM 便會輸出另外一個 reward $r_t^i$，而最後我們希望的就是整個 total reward 應該要越大越好。

$$
\max R(\tau)=\max\sum\limits_{t=1}^T\big(r_t+r_t^i\big)
$$

![](https://i.imgur.com/lFwD1hr.png)


ICM 到底是一個什麼東西呢 ? 它其實就是機器的好奇力的表徵，可以是一個 Neural Network，藉由 $s_t,a_t$ 的輸入，來做 $s_{t+1}$ 的預測，如果預測出來的 $\hat{s}_{t+1}$ 與 $s_{t+1}$ 差距越大，則給予越高的 reward。

但在環境中，其實往往會存在一些無法預測，但其實並不是太重要的因素。如果 ICM 也將這些因素考慮進去，那很可能會造成干擾。

舉例來說，在一個遊戲畫面，可能會有一些背景的變動，例如樹葉飄動、草枝擺(?)、...，雖然無法預測，但其與遊戲過程幾乎是無關，若 ICM 也會對這些因素進行考慮，很可能訓練出來的角色就會一直看著這些畫面而不進行其他動作，因為這樣可以上它得到很大的 reward。

所以我們會對 ICM 結構再做一點調整，除了針對 $s_t,a_t$ 的輸入，來做 $s_{t+1}$ 預測的 network 之外，我們還會增加另外一個 network 來對 state 做特徵萃取。

![](https://i.imgur.com/Sp3vve5.png)

這個特徵萃取的 network 最主要的工作就是將 $s_t,s_{t+1}$ 輸出 $\phi(s_t),\phi(s_{t+1})$ 再對這兩個空間轉換後的向量來預測 $a_t$，如果可以讓預測的 $\hat{a}_t$ 越接近 $a_t$ 越好。

當訓練出來這樣的 network 後，其實就表示可以把 state 中無關的訊息過濾掉，這樣讓 $\phi$ 輸入之前的 network 就不會有上面說的問題。


Curriculum Learning
---

這是一種循序漸進的學習方式，讓機器在達到最終目標以前，我們先給予一些簡單的任務，讓它可以逐漸完成，隨著難度的增加，然後達到最後我們希望它完成的工作。

這樣的學習方式也一樣要利用人工來為機器設計一系列的課程。


### Reverse Curriculum Generation

這是一個比較通用的 Curriculum Learning 方式 : 
1. 首先我們先確定我們最終希望達到的目標是什麼，稱為 golden state $s_g$，
2. 找出與 $s_g$ 接近的 $s_1$，利用機器從這些 $s_1$ 與環境做運動，得到 reward，
3. 找出可以到達 $s_g$ 的state，並且刪除極端值 (reward 太大或太小，表示任務太簡單或太困難)。
4. 根據上述挑選出來的 state，再去抽樣更多的 state 重複上述步驟。

這裡有一個比較困難的部分，就是怎麼定義其他 state 與 $s_g$ 的距離。這其實是 case by case 的，根據任務的不同，兩個向量「接近」的定義也會有所不同。


Hierarchical Reinforcement Learning
---

階層式的強化學習指的就是我們會訓練多個有階層關係的 Agent，根據階層的位階不同，其負責的任務也會有階層的關係。

而每一個 Agent 都是將較高階層的 Agent 輸出作為輸入。

較高階層的 Agent 提出願景，中階 Agent 將願景作為輸入，輸出一些實現的構想，而低階的 Agent 則是將這些構想輸入來輸出最終結果。

但是當較低階 Agent 無法完成任務，則較高階 Agent 則會得到 penalty。另外，如果低階 Agent 最後輸出並不是高階 Agent 期望的結果，那麼則改寫高階 Agent 的期望。(?)

這邊的確有點莫名其妙，有興趣的可以來看看原始論文 " *Hierarchical Reinforcement Learning with Hindsight* "。

![](https://i.imgur.com/uxxJMj7.png)

上圖桃紅色部分就是高階 Agent 給予低階 Agent 的目標，而低階 Agent 則是負責將藍色部分移動到桃紅色部分即算達成目標。