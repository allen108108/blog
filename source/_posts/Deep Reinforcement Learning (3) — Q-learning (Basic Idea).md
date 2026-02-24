---
title: "Deep Reinforcement Learning (3) --- Q-learning (Basic Idea)" 
date: 2019-12-04 12:13:34
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning and having it Deep and Structured
image: https://i.imgur.com/MdVC5JZ.png
mathjax: true
---

State-Value function $V^{\pi}(s)$
---

Q-Learning 是 Reinforcement Learning 中一種 Value-Based 的方法，在 Q-Learning 中，並不直接去學習一個 Policy 而是要學習一個評價函數 Critic，來評價某一個 policy $\pi$ 在某一個 state $s$ 中採取行為後，後續得到的期望 Reward 有多少。

因此，我們可以知道，一個 Critic $V^{\pi}(s)$ 必須配合一個 policy $\pi$ ，才能來計算後續的 Reward 有多少。

<!-- more -->

<img width=500 src="https://i.imgur.com/j81cCdN.png" >



在 $V^{\pi}(s)$ 的訓練上，我們要怎麼訓練 ? 一般來說有兩種方法 : 
1. Monte Carlo (MC) based approach
2. Temporal difference (TD) approach

### Monte Carlo (MC) based approach

在 " Deep Reinforcement Learning (2) — Proximal Policy Optimization (PPO) " 一文中有提到， Monte Carlo approach 就是利用抽樣來針對一個無法估測的函數進行逼近。在這邊也是類似的概念，我們無法窮舉所有的 $s$ 來訓練 $V^{\pi}(s)$，因此讓 $\pi$ 與 Environment 作互動來獲得有限筆資料，直接對 $V^{\pi}(s)$ 訓練。

![](https://i.imgur.com/KmOrfBx.png)


### Temporal difference (TD) approach

Monte Carlo approach 的一個問題是，我們必須要讓 $\pi$ 與 Environment 作一次又一次的完整互動，這樣所需要的時間實在太長，於是有了 Temporal difference approach。

TD 的概念是這樣，我們不再需要讓 $\pi$ 與 Environment 作一次又一次的完整互動，只需要在某一個時間點 $t$ 就可以來做 $V^{\pi}(s)$ 的 estimate。

從 $V^{\pi}(s)$ 的定義來看，在時間 $t$ 時，很顯然的下列式子會成立 : 

$$
V^{\pi}(s_t)=r_t+V^{\pi}(s_{t+1})\\
\Longrightarrow V^{\pi}(s_t)-V^{\pi}(s_{t+1})=r_t
$$

利用上面的式子，我們估測 $V^{\pi}(s)$ 的方式不再是看看最終 Reward 是否接近，而是只需要看看在 $t$ 時，得到的 reward $r_t$ 是不是接近即可。

![](https://i.imgur.com/CihBNxm.png)


### MC v.s. TD

Monte Carlo approach 與 Temporal difference approach 最主要的差異是目標函數的 Variance 差異。

在 MC method 中，目標函數為後續的最終 Reward 期望值，這是一個一個 $r_t$ 累加起來的結果，其實可以直觀的理解，當每一個 $r_t$ 都有所變動時，最後得到的 Reward 就會差異很大。而 TD method 關注在某一個時間點的 reward $r_t$ ，當然相對來說 variance 會小很多。

但 TD method 有一個比較麻煩的問題在於往往估測出來的 $V^{\pi}$ 可能會是不準的。

#### Example

以下是一個實際例子來看出兩者間的不同，假設有一個 policy $\pi$ 與 Environment 進行了 8 輪的互動。

![](https://i.imgur.com/mVAvTip.jpg)

這八次過程，面對的 state 就只有 $s_a,s_b$ 兩種，得到的 reward 不是 $0$ 就是 $1$。

從 MC method 角度來看，$s_a$ 僅出現過一次，就會依照這一次來計算 $V^{\pi}(s_a)$

$$
V^{\pi}(s_b)=\frac{6}{8}=\frac{3}{4}\\
V^{\pi}(s_a)=0
$$

但從 TD method 的角度來看就會不太一樣，$V^{\pi}(s_b)=\frac{3}{4}$ 這是不變的，但 $V^{\pi}(s_a)$ 的計算方式就有差別

$$
V^{\pi}(s_a)=r+V^{\pi}(s_b)=0+\frac{3}{4}=\frac{3}{4}
$$

嚴格說起來，這兩個答案其實並不衝突，端看我們怎麼定位 $s_a$ 這樣的 state，如果你認為 $s_a$ 就是導致 reward$=0$ 的 state，那 $V^{\pi}(s_a)=0$ 是很合理的 ; 但若 reward$=0$ 並不一定是 $s_a$ 所造成，可能跟 $s_b$ 的隨機性有關，那 $V^{\pi}(s_a)=\frac{3}{4}$ 也是可以接受的不是嗎 ?

State-action Value function $Q^{\pi}(s,a)$
---

$Q^{\pi}(s,a)$ 跟 $V^{\pi}(s)$ 都是 Critic，但是評價的東西不太一樣。

$V^{\pi}(s)$ 的輸入只有 state，因此他只評價這個 state，當看到這個 state 後，我們後續會累積的期望 reward 是多少。 
$Q^{\pi}(s,a)$ 是指在 state $s$ 中，強制採取 action $a$ ，後續累積的期望 reward 是多少。

![](https://i.imgur.com/8owvxSl.png)

上圖是 Q-function 的兩種結構，但要注意的是，右邊的方法只有離散 action 才可以使用，如果 action 是連續的、無法窮舉的，就只能使用右邊的結構。

下面是文獻上的結果，當四種不同 state 出現時，我們可以利用 Q-function 來算出強制使用三種不同 actions 後會獲得的期望分數。

![](https://i.imgur.com/2oyQEY1.png)


Q-Learning
---

在這邊，我們先講結論 : 當我們隨機初始一個 policy $\pi$，並利用這個 policy 學習出一個 Q-function $Q^{\pi}(s,a)$ 時，必然可以找到一個 $\pi'$ 滿足 $V^{\pi'}(s)\geq V^{\pi}(s)$ , $\forall\ s$。

翻譯蒟蒻 (嚼) : 當我們隨機得到一個 policy $\pi$，我們可以利用其 Q-function 來找到一個比 $\pi$ 還要更好的 $\pi'$，這個 $\pi'$ 不管在哪一個 state 上，都可以得到比較高的累積期望 reward。

當上面的結論可以成立，我們便可以利用 Q-function 不斷的更新 policy,最終得到一個最好的 policy 可以在與 environment 互動中得到最高的 reward，這就是所謂的 Q-Learning。

**Claim : $\pi'=\arg\max\limits_{a}Q^{\pi}(s,a)\Longrightarrow V^{\pi'}(s)\geq V^{\pi}(s)$, $\forall\ s$**

**< Proof >**

$$
V^{\pi}(s)=Q^{\pi}(s,\pi(s))\leq\max\limits_{a}Q^{\pi}(s,a)=Q^{\pi}(s,\pi'(s))\\
\Longrightarrow V^{\pi}(s)\leq Q^{\pi}(s,\pi'(s))\\
=\mathbb{E}\big[r_t+V^{\pi}(s_{t+1})\big]|_{s_t=s,a_t=\pi'(s_t)}\\
\leq\mathbb{E}\big[r_t+Q^{\pi}(s_{t+1},\pi'(s_{t+1}))|_{s_t=s,a_t=\pi'(s_t)}\\
=\mathbb{E}\big[r_t+r_{t+1}+V^{\pi}(s_{t+2})\big]|_{s_t=s,a_t=\pi'(s_t)}\\
\leq\cdots\\
\leq\mathbb{E}\big[r_t+r_{t+1}+r_{t+2}+\cdots+r_T\big]|_{s_t=s,a_t=\pi'(s_t)}\\
=V^{\pi'}(s)
$$


Tips of Q-Learning
---

### Target Function

在基本的 Q-Learning 中要學習一個 Q-function 通常使用的是前面提過的 TD method : $Q^{\pi}(s_t,a_t)=r_t+Q^{\pi}(s_{t+1},a_{t+1})$，但 TD 在實務上處理往往會有訓練不穩定的狀況，因為訓練過程中等號左右兩邊都是在變動的值，在訓練過程中要收斂變得非常困難。要解決這樣的問題，我們會讓等號右邊的 "Target" 先固定，只 train 等號右邊的 Q-function。待 Q-function 迭代更新數次後再用其替換掉 Target Network 繼續更新 Q-function。

![](https://i.imgur.com/HGkoTaN.jpg)


### Exploration

$$
\pi'=\arg\max\limits_{a}Q^{\pi}(s,a)
$$

當我們在進行 Q-Learning 時，policy 的更新會依照可以使 Q-function 產生最大值的 action 來更新。這樣其實會有一點問題，假使初期都沒有 action 被選取到，每一個 action 可以獲得的 Q 值都為 0，那麼該選擇哪一個就會是隨機的。但是，一旦我們選到一個 action 可以獲得正 Q 值，那麼後面的更新都「只」會採取這個 action。這樣的訓練很有問題，因為我們可能因此錯過其他可能得到更高 Q 值的 action。

處理這樣的問題在實務上大概概念上就是必須要給予其他選擇一定的機率被選取到，這樣才能夠「探索」到更多可能性。下面兩個方法基本上都是基於這樣的概念來設計的。

#### Epsilon Greedy

$$
\pi'=\begin{cases}\arg\max\limits_{a}Q^{\pi}(s,a),&\mbox{with probability }1-\epsilon_t\\random,&\mbox{with probability }\epsilon_t\end{cases}
$$

Epsilon Greedy 這個方法很直覺，反正就是給予其他可能性一個機率被選取到，唯一要注意的是， $\epsilon_t$ 應該隨著 $t$ 的增加而減少。


#### Boltzmann Exploration

$$
P(a|s)=\frac{\exp(Q^{\pi}(s,a))}{\sum\limits_{a}\exp(Q^{\pi}(s,a))}
$$

這是利用 Q 值來給定 action 的分佈，技巧上取 $\exp$ 再進行 normalization。

### Replay Buffer

在實務上，我們會給予 Q-Learning 一個足夠大的 buffer。這樣的緩衝區可以收集 $\pi$ 與 environment 互動的資料，甚至可以放進兩三輪不同 policy 互動的資料。

Buffer 的主要功能有幾個 : 
1. 讓訓練更有效率 : 一般我們再進行 RL 的時候，最耗時間的就是再進行資料的收集 ( policy 與環境的互動 )，有了這樣的 buffer 設置後，我們可以放進很多不同 policy 產生的資料，就可以進行比較有效率的 Q-function 訓練。
2. 確保訓練資料的分散性 : 若訓練資料過於集中，往往訓練出來的模型效果都不是太好，但現在我們的 buffer 裡面有許多不同 policy 產生的資料，每一次在取 batch 訓練時，可以確保資料的分散性，讓訓練出來的模型表現更好。

在這裡其實有一個問題 : 不同的 policy 產生的資料用來訓練特定 policy 的 Q-function 到底有沒有問題 ?
其實理論上是可行的，雖然說不同的 policy 產生的資料，但其實仔細想想，我們根本不用管資料本身是從哪個 policy 出來的不是嗎 ? 我們重點只放在 「當遇到 state $s$，採取了 action $a$，得到的 reward 是多少。」即使不同 policy ，其實不會影響這個資料的可用性。


Typical Q-Learning Algorithm
---

* 初始化 Q-funtion $Q$ 與 target function $\hat{Q}=Q$
* 在每一個 episode 中 : 
    *  每 $t$ 個時間段 : 
        * 當遇見 state $s_t$，利用 epsilon greedy 的方式從 Q-function 來決定 action $a_t$
        * 我們會得到 $r_t$， state 轉換成 $s_{t+1}$
        * 將資料 $(s_t,a_t,r_t,s_{t+1})$ 儲存到 buffer 中
        * 從 buffer 裡面抽樣一個 batch 的資料 $\{(s_i,a_i,r_i,s_{i+1})\}_{i=1}^{n}$ 出來
        * 利用 $\hat{Q}$ 計算這一個 batch 的 target $y_i=r_i+\max\limits_{a}\hat{Q}(s_{i+1},a)$
        * 訓練 $Q$ 使 $Q(s_i,a_i)$ 盡可能地接近 $y_i$
        * 更新 $C$ 次後，用 $Q$ 來取代 $\hat{Q}$