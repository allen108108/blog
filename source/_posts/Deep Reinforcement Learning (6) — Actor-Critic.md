---
title: "Deep Reinforcement Learning (6) --- Actor-Critic" 
date: 2020-04-21 08:56:14
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning and having it Deep and Structured
image: https://i.imgur.com/MdVC5JZ.png
mathjax: true
---

從 Policy Gradieint 開始說起
---

在前面的課程筆記 "[Deep Reinforcement Learning (1) --- Policy Gradient (Review)](https://)" 中，最後我們推導出了 policy gradient 的通式如下 :

$$
\nabla\bar{R}_{\theta}\approx\frac{1}{N}\sum\limits_{i=1}^{N}\sum\limits_{t=1}^{T}\big(\sum\limits_{t'=t}^{T_n}\alpha^{t'-t} r_{t'}^n-b\big)\nabla\log P_{\theta}(a_t^n|s_t^n)
$$

<!-- more -->

然而在這樣的通式中，$G_t^n=\sum\limits_{t'=t}^{T_n}\alpha^{t'-t} r_{t'}^n$ 其實是非常不穩定的，因為互動的過程中其實是充滿隨機性的，每一個 $s_t$，會有對應的隨機 $a_t$，隨後又會有一個隨機的 $s_{t+1}$ 出現，這過程中，$r_t$ 便也充滿了隨機性於其中，而 $G_t^n$ 就是這些 $r_t$ 的總和，當然會非常不穩定。

換一個角度來說，我們就是藉由抽樣 $G_t^n$ 來做梯度的估測，但如果我們的抽樣數不夠多 ( 通常都會不夠多 ) 那麼如果我們抽到表現較差的 $G_t^n$，便會很有可能影響到後面的結果。因此要解這一個問題，就必須要想辦法處理 $G_t^n$ 這種不穩定的狀況，再直接一點說，我們能不能直接估出 $G_t^n$ 的期望值來取代 $G_t^n$ ? 既可以有表徵的能力，也可以避免 $G_t^n$ 的震盪來影響結果。

那，$\mathbb{E}\big[G_t^n\big]$ 到底指的是什麼呢 ?

與 Q-Learning 結合
---

從 $\mathbb{E}\big[G_t^n\big]$ 的定義來看，其實它是等同於 Q-function 的，兩者都是在計算在時間 $t$ 之後獲得 cumulated reward 的期望值是多少。

$$
\mathbb{E}\big[G_t^n\big]=Q^{\pi_{\theta}}(s_t^n,a_t^n)
$$

因此我們可以將 policy gradient 的通式利用 Q-function 來做修改，然後 $b$ 則以 $V^{\pi_{\theta}}(s_t^n)$ 來取代。$V$ 是不考量 action 的 cumulated reward，所以從意義上來看，$V$ 其實是 $Q$ 針對 action 的期望值，因此可以依舊可以確保 $Q^{\pi_{\theta}}(s_t^n,a_t^n)-b$ 這一項不會恆正。
 
$$
\nabla\bar{R}_{\theta}\approx\frac{1}{N}\sum\limits_{i=1}^{N}\sum\limits_{t=1}^{T}\big(Q^{\pi_{\theta}}(s_t^n,a_t^n)-V^{\pi_{\theta}}(s_t^n)\big)\nabla\log P_{\theta}(a_t^n|s_t^n)
$$

這種型態結合了 Actor 與 Critic ，便稱為 Actor-Critic。

Advantage Actor-Critic ( A2C )
---

在上述的實作上，並不會就這樣直接做，原因是因為這樣的結構我們變成要使用兩個 network 來做估測，那也相當於可能會讓誤差變成兩倍。( 兩個網路都可能會造成誤差 )

因此在實作上會稍微做一點調整

$$
Q^{\pi_{\theta}}(s_t^n,a_t^n)=\mathbb{E}\big[r_t^n+V^{\pi_{\theta}}(s_{t+1}^n)\big]\overset{let}{=}r_t^n+V^{\pi_{\theta}}(s_{t+1}^n)
$$

這裡其實是給了一個蠻強的前提在裡面，在直覺上來說稍微可以理解，$r_t^n$ 是單一個步驟的 reward ，因此 variance 較小，有沒有取期望值是可能有機會相等的。其次，在文獻上也可以由實驗發現這樣的前提下做出來的實驗是夠好的。

藉由上面這樣的強假設，我們可以再一次改寫梯度的估測通式

$$
\nabla\bar{R}_{\theta}\approx\frac{1}{N}\sum\limits_{i=1}^{N}\sum\limits_{t=1}^{T}\big(r_t^n+V^{\pi_{\theta}}(s_{t+1}^n)-V^{\pi_{\theta}}(s_t^n)\big)\nabla\log P_{\theta}(a_t^n|s_t^n)
$$

這樣的改變，我們只需要利用一個 network 來估測梯度，但換來一個不穩定的隨機變量 $r_t^n$，但前面我們也提到，$r_t^n$ 的 variance 其實不大，因此這樣的交易還算划算。

因此我們可以了解整個 A2C 的過程如下圖 : 

![](https://i.imgur.com/HmwDXk2.png)

### Tips of A2C

在 A2C 中，我們仍然會使用一些技巧來幫助訓練 : 

#### 1. 權重共享

在整個 A2C 中，我們有兩個網路結構需要訓練 : $V$ 與 $\pi$，這兩個網路結構其實都是針對 state 的輸入來進行目標的估測，因此在實務上，會將這兩個網路的前半部分做權重共享來減少訓練參數。


#### 2. 限制 $\pi$ 的輸出 entropy 必須要大於某值

主要原因其實就是為了加強整個網路的探索能力，當 entropy 越高，相對地對於不同 action 的選擇機率就會越接近，這樣可以確保探索到其他的 actions。


Asynchronous Advantage Actor-Critic ( A3C )
---

說穿了 A3C 就是一個平行運算的應用，利用不同可平行運算的單元同時進行運算並更新權重，這樣可以讓整個訓練速度加快許多。


![](https://i.imgur.com/QIAlYfR.png)
(圖片來源 : [Simple Reinforcement Learning with Tensorflow Part 8: Asynchronous Actor-Critic Agents (A3C)](https://medium.com/emergent-future/simple-reinforcement-learning-with-tensorflow-part-8-asynchronous-actor-critic-agents-a3c-c88f72a5e9f2#.68x6na7o9))

從上圖來看，我可以同時將兩個網路結構放在不同的運算單元中，同時進行計算，先計算出梯度的就可以先對參數做更新，所以如果 worker 2 先計算出梯度更新參數，那麼比較慢計算出梯度的 worker 1 就會將已經被 worker 2 更新過後的權重再進行更新。


Pathwise Derivative Policy Gradient
---

這可以看做是一個 Actor-Critic 的特例，可以作為 Q-Learning 解決 Continuous Actor 的方案之一。

前面我們在說 Q-Learning 時，$V$ 或 $Q$ function 都是在評估這樣的 actor 好還是不好，但 Pathwise Derivative Policy Gradient 不只要讓你知道 action 的好壞，還要直接告訴你可以得到最大 cumulated reward 的 action 是什麼。

![](https://i.imgur.com/4ztbFPX.png)

做法其實就是將 actor 作為 solver 來產出 action 提供給 $Q$ 然後目標要讓 Q 值越大越好。這個結構是否看起來熟悉 ? 這其實就是前一個部分課程的 GAN 架構。我們將原本要找出一個能夠讓 Q 值最大化的 action 這樣的問題轉化為一個 GAN 結構來解決。 actor 相當於 generator 的角色，而 $Q$ 就是一個 discriminator。

### Algorithm

![](https://i.imgur.com/TpU4t6H.png)


* 初始化 Q-funtion $Q$ 與 target function $\hat{Q}=Q$
* 初始化 Actor $\pi$ 與 target function $\hat{\pi}=\pi$
* 在每一個 episode 中 : 
    *  每 $t$ 個時間段 : 
        * 當遇見 state $s_t$，利用 entropy constrain 從 Actor $\pi$ 來決定 action $a_t$
        * 我們會得到 $r_t$， state 轉換成 $s_{t+1}$
        * 將資料 $(s_t,a_t,r_t,s_{t+1})$ 儲存到 buffer 中
        * 從 buffer 裡面抽樣一個 batch 的資料 $\{(s_i,a_i,r_i,s_{i+1})\}_{i=1}^{n}$ 出來
        * 利用 $\hat{Q}$ 計算這一個 batch 的 target $y_i=r_i+\hat{Q}(s_i,\hat{\pi}(s_{i+1}))$
        * 訓練 $Q$ 使 $Q(s_i,a_i)$ 盡可能地接近 $y_i$
        * 訓練 $\pi$ 使 $Q(s_i,\pi(s_i))$ 越大越好
        * 更新 $C$ 次後，用 $Q$ 來取代 $\hat{Q}$
        * 更新 $C$ 次後，用 $\pi$ 來取代 $\hat{\pi}$


GAN V.s. Actor-Critic
---

這一個部分，在論文 " *Connecting Generative Adversarial Networks and Actor-Critic Methods* " 中有非常詳盡的介紹。

其中，作者詳列了 GAN 與 Actor-Critic 中所用到的一些技巧，既然兩者的概念如此接近，或許這些技巧也能互相使用 ? 

![](https://i.imgur.com/EgyKxL0.png)
