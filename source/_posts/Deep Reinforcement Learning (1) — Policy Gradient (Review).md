---
title: "Deep Reinforcement Learning (1) --- Policy Gradient (Review)" 
date: 2019-12-04 12:10:09
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning and having it Deep and Structured
image: https://i.imgur.com/MdVC5JZ.png
mathjax: true
---


作為 Deep Reinforcement Learning 的第一堂課，主要是針對 Machine Learning 中 RL 的部分進行簡單的複習及補充。有些基礎的概念，在 MLDS GAN 系列課程中也已有提及，閱讀此篇筆記前可先看看 " [Reinforcement Learning](https://) " 及 " [Generative Adversarial Network (9) — Sequence Generation](http://bit.ly/2D6MEMp) " 這兩篇筆記。

<!-- more -->

Reinforcement Learning
---

Reinforcement Learning 說穿了就是利用機器進行實作，並且從錯誤中自行學習應該採取什麼樣子的「策略」，可以讓自己在任務中得到最大的獎勵、分數。如果拿玩遊戲跟 AlphaGO 作為例子，我們可以將整個 Reinforcement Learning 分成三個主要的部份 : **Actor, Environment 以及 Reward Function**，其中 Environment 以及 Reward Function 是我們無法調整的部分。

![](https://i.imgur.com/xNvQEIO.png)

機器進行 Actor 跟 Environment 的每一個 State 作互動會經過 Reward Function 得到 Reward。我們藉由這個 Reward Function 來調整 Actor 使 Reward 最大化。

機器決定怎麼進行 Actor 便是由 Policy $\pi$ 這個 Neural Network 來決定，當我們輸入一個 State 這個網路 $\pi$ 會輸出一個 Actor 分佈。利用這個分佈來決定我們要採用哪一個 Actor。

整個互動的過程就如同下圖所展示 : 

![](https://i.imgur.com/fdl672p.png)

我們通常將一整個得到最終 Reward 的過程稱為一個 Episode ( 舉例來說，遊戲從開始到 Game Over、棋盤從開始到分出勝負、... )，那麼一個 Episode 可以用以下的歷程 $trajectory$ 來表示

$$
trajectory\ \tau=\{s_1,a_1,r_1,s_2,a_2,r_2,\cdots\cdots s_T,a_T,r_T\}
$$

所以這樣的 $trajectory$ 出現的機率是我們可以進行估算的 

$$
P_{\theta}(\tau)=P(s_1)P_{\theta}(a_1\mid s_1)P(s_2\mid s_1,a_1)\cdot P_{\theta}(a_2\mid s_2)P(s_3\mid s_2,a_2)\cdots\\
=P(s_1)\prod\limits_{t=1}^{T}P_{\theta}(a_t\mid s_t)P(s_{t+1}\mid s_t,a_t)
$$

每一個 Actor 過後，我們會得到一個 reward $r_t$，當一整個歷程結束後，reward 的總和是 $R(\tau)=\sum\limits_{t=1}^{T}r_t$，我們希望經過 Actor 這個網路的優化可以讓最後的 reward 總和盡量的高。也就是說，我們會希望藉由 $\theta$ 的調整下，使期望 reward $\bar{R}_{\theta}$  可以盡可能的高。

$$
\bar{R}_{\theta}=\sum\limits_{\tau}R(\tau)P_{\theta}(\tau)=\mathbb{E}_{\tau\sim P_{\theta}(\tau)}\big[R(\tau)\big]
$$

Policy Gradient
---

我們將上面的 reward 期望值做一些調整

$$
\nabla\bar{R}_{\theta}=\sum\limits_{\tau}R(\tau)\nabla P_{\theta}(\tau)\\
=\sum\limits_{\tau}R(\tau)P_{\theta}(\tau)\frac{\nabla P_{\theta}(\tau)}{P_{\theta}(\tau)}\\
=\sum\limits_{\tau}R(\tau)P_{\theta}(\tau)\nabla\log P_{\theta}(\tau)\\
=\mathbb{E}_{\tau\sim P_{\theta}(\tau)}\big[R(\tau)\nabla\log P_{\theta}(\tau)\big]\\
\approx\frac{1}{N}\sum\limits_{i=1}^{N}R(\tau^n)\nabla\log P_{\theta}(\tau^n)\\
=\frac{1}{N}\sum\limits_{i=1}^{N}\sum\limits_{t=1}^{T_n}R(\tau^n)\nabla\log P_{\theta}(a_t^n|s_t^n)
$$

在這邊想提一下 "Policy" 這樣的名詞，直接翻譯叫做「策略」。我想說的是，為什麼要特別提到 Policy 這樣的詞彙 ? 跟我們以前所了解的梯度下降到底有什麼不同 ? 

上面的討論簡化了一些部分，如果依照以往的整個神經網路優化過程來看，我們應該要優化網路來使 Loss 盡可能的小。但在 Reinforcement Learning 中，我們卻沒辦法這樣做。

在 Reinforcement Learning 中，我們用的是 Reward Function 取代 Loss function，目標也從最小化 Loss 改成最大化 Reward。現在的問題就是，我們很無法對 Reward 進行梯度下降法來優化 Actor 這個神經網路。主要的原因是因為 Actor 輸出的是一個 Action ，並不是 Reward，而且最後得到的 Reward $R(\tau)$ 事實上我們會看不到與 Actor 的參數 $\theta$ 之關係。

因此我們會用上面的方式調整來進行梯度下降，雖然我們要求的是 $\nabla\bar{R}_{\theta}$ 但其實是在求 $\nabla P_{\theta}(\tau)$ 其實也就是對 「策略」本身求梯度。

在進行 Policy Gradient 必須要注意的是，每一次 Sample 的 Data 只能進行一次參數更新。當參數更新過後就必須要重新 Sample 一次 Data。

![](https://i.imgur.com/Hoc50Qa.png)


Tips of Reinforcement Learning
---

### Tip 1. Add baseline/threshold

$$
\nabla\bar{R}_{\theta}\approx\frac{1}{N}\sum\limits_{i=1}^{N}\sum\limits_{t=1}^{T_n}R(\tau^n)\nabla\log P_{\theta}(a_t^n|s_t^n)
$$

從上面的式子來看，當我們進行參數優化時，在意義上就是要將正的 $R(\tau^n)$ 之機率提升，然後將負的 $R(\tau^n)$ 之機率降低。

雖然這樣說，但大部分的 Reinforcement Learning 場景都不會有「負分」的狀況出現，在這樣恆正的 Reward 下，會出現一些問題。

在理想的狀態下，假設我們的 Actor 有三個選擇，各自能獲得不同的 $R(\tau)$ (綠色部分)，在三個選擇都會被 sample 到的前提下，三個狀況都會被增加機率，擁有較大 $R(\tau)$ 的機率會被增加的較多，而較小 $R(\tau)$ 的機率會被增加的較少。經過 Normalization 後，是可以達到我們想要的效果( 增加較少 reward 被降低機率、增加較多 reward 的被增加機率 )。

![](https://i.imgur.com/sRBCQ3V.png)

但是問題出在，我們在 sample 的過程中，不可能 sample 到所有的可能性，如果我們沒有 sample 到擁有較大 $R(\tau)$ 的 actor，那麼這 actor 經過 Normalization 後反而會降機機率。

![](https://i.imgur.com/e2n4gsh.png)

為了處理這樣的問題，我們在 $R(\tau)$ 的部份加上一個 threshold $b$，使整個 reward 可以變成有正有負的狀況以避開上述的問題。而這樣的 $b$ 通常是取 $R(\tau)$ 的平均值 (期望值)。

$$
\nabla\bar{R}_{\theta}\approx\frac{1}{N}\sum\limits_{i=1}^{N}\sum\limits_{t=1}^{T_n}\big(R(\tau^n)-b\big)\nabla\log P_{\theta}(a_t^n|s_t^n)
$$


### Tip 2. Assign Suitable Credit

仔細看一下 Reward Function 仍是有不合理之處

$$
\nabla\bar{R}_{\theta}\approx\frac{1}{N}\sum\limits_{i=1}^{N}\sum\limits_{t=1}^{T_n}\big(R(\tau^n)-b\big)\nabla\log P_{\theta}(a_t^n|s_t^n)
$$

我們對每一個 state $s_t^n$ 跟 actor $a_t^n$ 都乘上同一個 total reward $R(\tau^n)$ 這顯然不合理。最後的 Reward 大，不見得在任一個 state 的 actor 都是好的，反之亦然。因此我們希望給每一個 actor 不同的 reward 權重。

這裡的想法是，每一個時間點的 state $s_t^n$ 跟 actor $a_t^n$ 的重要性，跟前面時間點的 state 與 actor 無關。這在直覺上還算好理解，每一個狀況下我們要採取動作會對最後的 Reward 產生的影響跟之前是無關的，因為之前的 actor 已經結束了。

因此我們不再給每一個 actor 同樣的 total reward 權重，而是將其與之後的 reward 作為權重( 之前的 reward 就不管了 )。

![](https://i.imgur.com/9TbT86D.png)

所以我們可以再把 reward function 進行更改

$$
\nabla\bar{R}_{\theta}\approx\frac{1}{N}\sum\limits_{i=1}^{N}\sum\limits_{t=1}^{T}\big(\sum\limits_{t'=t}^{T_n}r_{t'}^n-b\big)\nabla\log P_{\theta}(a_t^n|s_t^n)
$$

但這樣還不夠貼近現實，雖然我們說這樣的計算可以表徵在任一個時間點下做的決定，會影響到後面所有的決定，導致 reward 產生變化。但我們可以合理的假設，這樣的影響力會隨著時間拉長而造成衰減，因此我們可以在 $r_{t'}^n$ 上再加上一個小於 $1$ 的 decay $\alpha$。

$$
\nabla\bar{R}_{\theta}\approx\frac{1}{N}\sum\limits_{i=1}^{N}\sum\limits_{t=1}^{T}\big(\sum\limits_{t'=t}^{T_n}\alpha^{t'-t} r_{t'}^n-b\big)\nabla\log P_{\theta}(a_t^n|s_t^n)
$$
