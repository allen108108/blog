---
title: "Reinforcement Learning"
date: 2019-10-11 10:40:20
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning
image: https://i.imgur.com/i5jt7QR.png
mathjax: true
---


Reinforcement Learning
---



<img width=500 src="https://i.imgur.com/Y5bpDMV.png" >


Reinforcement Learning 整個概念大概就如上圖 : 

環境狀態 (State of Environment) 經由 機器觀察 (Observation of Machine) 後會做出一些行為 (Action) 來改變環境狀態。而改變後的環境狀態除了依舊會經由機器觀察促使 Machine 做出一些行為外，同時也會對 Machine 給予其獎賞 (Reward)，因此，Reinforcement Learning 就是經由這些正負獎賞使 Machine 可以去學習如何採取 Action，讓 Reward 最大化。

<!-- more -->

Applications of Reinforcement Learning
---

AlphaGo 其實是一個具有多種學習演算法的機器，其內部含有Superviced Learning 與 Reinforcement Learning 在其中。

從 Reinforcement Learning 的角度來看，經由不斷跟對手下棋的過程中，會得到一些獎賞 (吃掉別人的子、自己的子被別人吃掉)，經由這些過程來學習如何下圍棋。

![](https://i.imgur.com/JuN3Hjw.png)

但在這樣的過程中，真正困難的點在於，絕大多數的過程都是沒有 Reward 的。如何在這樣沒有 Reward 的過程中學習怎麼將子落在正確位置則是很大的挑戰。

從 Superviced Learning 的部分來看，可以給許多棋譜，讓機器可以學習在什麼樣的狀況下落怎麼樣的子，這樣的方式可以在沒有 Reward 的情形下，機器也可以落子在正確的位置上。

綜觀以上，我們可以說 Superviced Learning 是 Learning by Teacher，而 Reinforcement Learning 是 Learning by Experience。


相同的，Reinforcement Learning 也可應用在 Chatbot 聊天機器人上，藉由聊天過程從對方的回饋取得 Reward 來調整對話。甚至我們可以利用兩個聊天機器人互相進行對話來學習。

最近幾年來，許多人將 Reinforcement Learning 應用在 Vedio Game 上，以下我們用 1978年的一款街機遊戲 《太空侵略者》（日語：スペースインベーダー／英語：Space Invaders）來概略說明 Reinforcement Learning 在遊戲上的應用。


<img width=500 src="https://i.imgur.com/0s3ko4y.png" >


這款遊戲藉由擊殺外星人而獲得分數 (Reward)，玩家可使用的操作 (Action)則是簡單的左右移動以及開火射擊，三種動作。遊戲過程中外星人也會對我們進行攻擊，而當外星人完全被消滅或是我們被外星人摧毀則遊戲結束。

這一整過程我們可以視為是一連串的互動所串連後的結果 : 

觀察到 State $s_1$，機器獲得 Reward $r_1$，並且做出 Action $a_1$
$\longrightarrow$  State 變為 $s_2$，機器獲得 Reward $r_2$，並且做出 Action $a_2$
$\longrightarrow$  State 變為 $s_3$，機器獲得 Reward $r_3$，並且做出 Action $a_3$
$\longrightarrow\cdots\cdots$ 
$\longrightarrow$ 經過 T 次迭代後接收到最後一次 $r_T$ 後遊戲結束。

整個過程稱為一個 Episode，而機器便是要學習到如何在這個 Episode 中獲得最高的 Reward。

雖然說這樣的過程我們是以遊戲為舉例，但這樣的過程也就是所有 Reinforcement Learning 的過程。


Difficulties of Reinforcement Learning
---

### 1. Reward Delay
Reinforcement Learning 中，Reward 的獲得通常是會有延遲的。
舉前面的 Space Invaders 來說，Reward 的獲得必須要擊殺外星人才能獲得，因此要獲得 Reward 的直接方式就是不停開火。但有玩遊戲的人一定會知道，雖然左右移動並不會馬上得到 Reward，但這樣的動作卻可能會在幾個 Action 之後有助於 Reward 的獲得。

下棋亦是如此，有時候必須藉由犧牲掉自己的子來試圖取勝。

### 2. Exploration
從上面的敘述我們也可以知道，Reinforcement Learning 的過程中，每一個 Action 都會影響到後面幾步的結果。因此機器本身必須要懂得探索，必須要嘗試不同的Action，才能知道這樣的動作會有什麼影響。


Methods of Reinforcement Learning
---

![](https://i.imgur.com/8HEQR7E.png)

在本次課程中，李宏毅主要講解的是 Model-Free Approach 裡面 Policy-Based 的部分。

### Policy-Based Approach

* ### Define an Actor

Policy-Based Approach 基本上就是將整個 NN 視為一個 Actor，而這一個 NN 的輸入是 State of Environment ( 可能是一組向量或是一個矩陣 )，而輸出的是對應於這個 State 的每一個 Action 之機率分布。

![](https://i.imgur.com/fRBgiOt.jpg)

一般來說，在採取 Action 時會直接採取機率較高的那個行為，但在 Policy-Based Approach 我們會假設 Action 是隨機的，也就是說，Actor 會依照這樣的機率分布隨機採取不同的 Action，因此，輸入相同的 State ，Actor 會採取的 Action 並不見得會相同。 

最傳統的方式通常會使用一個 Table 來做檢索，當什麼 State 進來，Actor 就從 table 裡面找出對應的 Action。但我們如果利用 NN 來取代 Table，一來可以處理 image 這種 table 無法處理的畫素資料，二來是沒見過的 State 於 table 中檢索不到，便無法有 Action 輸出，但 NN 可以有舉一反三的能力，即使沒遇過的 State 也能有適當的反應出來，泛化性較高。

* ### Define the goodness of the Actor

一般我們要衡量一個 model 的優劣，就是定義一個 Loss function 來計算預測值與真實值的差距。然而在 Reinforcement Learning 中，我們定義 model 優劣的方式則是計算 Reward 大小。

假如我們給定一組參數 $\theta$，則可決定一個 Actor $\pi_{\theta} : \pi_{\theta}(s_k)=a_k$，而 $a_k$ 會提供一個 Reward $r_k$，並且改變 State 成為 $s_{k+1}$。經過一整個 Episode 之後的 Total Reward 應為

$$
R_{\theta}=\sum\limits_{t=1}^Tr_t
$$

但我們要關心的並非 Total Reward，原因在於上面我們曾經說到的，Actor 會隨機採取不同的 Action，同樣的場景可能會有不同的 Action，也會有不同的 Total Reward。所以我們要關注的應該是 $R_{\theta}$ 的期望值 $\overline{R_{\theta}}$。

如果我們將整個 Episode 歷程 ( trajectory ) 以下列來表示

$$
\tau=\left\{s_1,a_1,r_1,s_2,a_2,r_2,\cdots\cdots s_T,a_T,r_T\right\}
$$

那麼在這 Actor 形成這一個歷程的條件機率為 : 

$$
P(\tau\mid\theta)=P(s_1)P(a_1\mid s_1,\theta)P(r_1,s_2\mid s_1,a_1)\cdot P(a_2\mid s_2,\theta)P(r_2,s_3\mid s_2,a_2)\cdots\\
=P(s_1)\prod\limits_{t=1}^{T}P(a_t\mid s_t,\theta)P(r_t,s_{t+1}\mid s_t,a_t)
$$ 

由上式我們可以計算出 $\overline{R_{\theta}}$

$$
\because R(\tau)=\sum\limits_{t=1}^{T}r_t\\
\therefore \overline{R_{\theta}}=\sum\limits_{\tau} P(\tau\mid\theta)R(\tau)
$$

然而要窮舉所有的 $\tau$ 通常是不太可行的，以 Video Game 為例，$\tau$ 是一個連續空間，要窮舉無窮多種可能根本不可能。因此我們可以進行 N 次 Episode，產生 N 個歷程，這就像是在 $P(\tau\mid \theta)$ 這樣的分布內 經過 N 次抽樣產生 $\left\{\tau^1,\tau^2,\cdots,\tau^N\right\}$

$$
\overline{R_{\theta}}=\sum\limits_{\tau} P(\tau\mid\theta)R(\tau)\approx\displaystyle{\frac{1}{N}\sum\limits_{n=1}^{N}R(\tau^n)}
$$


* ### Find a best Actor

既然我們已經可以對 Reward 進行估算，那麼最好的 Actor 勢必能讓 Reward 最大化

$$
\theta^*=arg\max_{\theta}\overline{R_{\theta}}
$$

一樣地，我們可以用 BP 來進行優化

$$
\theta^{t+1}\longleftarrow \theta^t+\eta\nabla \overline{R_{\theta^t}}
$$

其中

$$
\nabla\overline{R_{\theta}}=\sum\limits_{\tau} \nabla P(\tau\mid\theta)R(\tau)\\
=\sum\limits_{\tau}R(\tau)P(\tau\mid\theta)\cdot\displaystyle{\frac{\nabla P(\tau\mid\theta)}{P(\tau\mid\theta)}}\\
=\sum\limits_{\tau}R(\tau)P(\tau\mid\theta)\cdot\nabla\log P(\tau\mid\theta)\\
\approx\displaystyle{\frac{1}{N}\sum\limits_{n=1}^{N}R(\tau^n)}\nabla\log P(\tau^n\mid\theta)\\
=\displaystyle{\frac{1}{N}\sum\limits_{n=1}^{N}R(\tau^n)}\sum\limits_{t=1}^{T_N}\nabla\log P(a^n_t\mid s^n_t,\theta)\\
\Big(\because \log P(\tau^n\mid\theta)=\log P(s_1)+\sum\limits_{t=1}^{T}\log P(a_t\mid s_t,\theta)+\log P(r_t,s_{t+1}\mid s_t,a_t)\Big)\\
=\displaystyle{\frac{1}{N}\sum\limits_{n=1}^{N}\sum\limits_{t=1}^{T_n}R(\tau^n)\nabla\log P(a_t^n\mid s_t^n,\theta)}\cdots\cdots(1)
$$

推導到這邊有幾個地方需要注意 : 

**1. $R(\tau)$ 的正負，有什麼影響 ?**

$R(\tau)>0\Longrightarrow$ 機器會調整 $\theta$ 來試圖增加機率 $P(a_t^n\mid s_t^n)$
$R(\tau)<0\Longrightarrow$ 機器會調整 $\theta$ 來試圖減少機率 $P(a_t^n\mid s_t^n)$

**2. $R(\tau)$ 會造成什麼影響 ? 可以怎麼調整 ?**

在 Space Invaders 中，$R(\tau)$ 絕對不會出現負值，最少就是 0 分，但這樣其實會造成一些問題如下 

在理想狀況下 ( $窮舉所有的\tau$ )

![](https://i.imgur.com/sRBCQ3V.png)

因為 $R(\tau)>0$，所以每一種情況 (不管機率高低) 的 $P(a_t^n\mid s_t^n)$ 都會被機器適當地增強，意即 $R(\tau)$ (綠色部分) 大的情況 $P(a_t^n\mid s_t^n)$ 就會上升的較小，反之。因此被機器調整過後，$R(\tau)$ 大的情況相對來說機率就是降低的，有一種 normalization 的概念。

然而，我們說過窮舉所有的 $\tau$ 是不可能的，如果我們從 $\tau$ space 中進行抽樣，便會有可能有某些情況沒有被抽樣到的。

![](https://i.imgur.com/e2n4gsh.png)

在這樣的情況下，可能會出現一個大的 $R(\tau)$ 但沒有被抽樣到的情況，經過 normalization 之後相當於機率被降低，這樣在 gradient 更新上是會出現問題的。

要解決這樣的問題所採取的方法是給定 $R(\tau)$ 一個 Baseline (也就是一個 Bias )，藉由這個 b 值來使 $R(\tau)$ 有正有負。

$$
\overline{R_{\theta}}=\displaystyle{\frac{1}{N}\sum\limits_{n=1}^{N}\sum\limits_{t=1}^{T_n}\big(R(\tau^n)-b\big)\nabla\log P(a_t^n\mid s_t^n,\theta)}
$$



**3. 為何在特定的 State 下輸出的 Action 之機率，乘上的並非當下獲得的 Reward，而是整個歷程下的 Total Reward?**

這個部份其實還算蠻直覺的，如果我們將 (1) 式中的 $R(\tau^n)$ 改為 $r_t^n$ ，以 Space Invaders 來說，最後 Actor 只會一直開火，因為只有開火才會使 $r_t^n$ 增加。


**4. 取 log 的原因 ?** 

取 log 的好處最明顯的是在上述推導過程中，可以把連乘項轉為連加項，降低演算法的複雜度。但真正最重要的不僅於此。

$$
\nabla\log P(a_t^n\mid s_t^n,\theta)\cdot
R(\tau)=\displaystyle{\frac{\nabla P(a_t^n\mid s_t^n,\theta)}{P(a_t^n\mid s_t^n,\theta)}}\cdot R(\tau)
$$

從上式來看，如果我們不取 log 單純只看 $\nabla P$ ，那麼機器在進行權重更新時，一般來說會往機率高的部分進行更新。但是，我們希望權重更新應該是往 $R(\tau)$ 大的部分來進行更新。

 ![](https://i.imgur.com/G6QqjeX.jpg)

以上圖為例，進行抽樣時，假設我們抽樣到比較高機率 $R(\tau)=1$，而極小機率會出現較高的 $R(\tau)=2$，那麼進行權重更新時，我們可以發現權重會往 $R(\tau)$ 小但機率高的方向移動，這並不符合我們想要的結果，我們期望的是 $R(\tau)$ 盡可能的大。

因此在這邊除以機率值，在某種程度上來說就是一種 normalization，讓權重更新的重點放在 $R(\tau)$ 上。









