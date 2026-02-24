---
title: "Deep Reinforcement Learning (2) --- Proximal Policy Optimization (PPO)" 
date: 2019-12-04 12:13:24
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning and having it Deep and Structured
image: https://i.imgur.com/MdVC5JZ.png
mathjax: true
---

On Policy & Off Policy
---

我們前面所用的技巧稱為 " On Policy " ，意思就是，我們用來跟 Environment 互動的 Actor 跟我們要訓練的 Actor 是同一個。

但前面有提到，這樣的訓練其實會花非常多時間，因為每一次參數更新完，就必須要重新讓 Actor 跟 Environment 互動收集新的資料，然後再進行訓練。是不是有辦法可以讓 Actor 跟 Environment 互動收集的資料拿來更新多次參數，省去一直收集資料的冗長過程 ?

這就是 " Off Policy " 的作用。在 Off Policy 中，我們利用一個不同的 Actor 、Policy $\pi_{\theta}'$  來跟 Environment 互動收集資料，然後讓我們想要訓練的 Actor 、 Policy $\pi_{\theta}'$ 利用這些資料，一次進行多次參數更新。

<!-- more -->

Important Sampling
---

重要性取樣 （Important Sampling），是一個基於蒙地卡羅法 (Monte Carlo Method) 的取樣方式。所謂蒙地卡羅法，指的是當某一個目標函數不可得時，我們可以利用取樣的方式來對這個目標函數做逼近[^註1]。

Important Sampling 的概念也是如此，假設我們有一個分佈 $P$ 其期望值

$$
\mathbb{E}_{x\sim P}\big[f(z)\big]=\int f(x)P(x)dx
$$

如果今天分佈 $P$ 無法積分，那我們能不能用另外一個已知的分佈 $Q$ 來嘗試估計這個期望值 ?

$$
\mathbb{E}_{x\sim P}\big[f(x)\big]=\int f(x)P(x)dx\\
=\int f(x)Q(x)\frac{P(x)}{Q(x)}dx\\
=\mathbb{E}_{x\sim Q}\big[f(x)\frac{P(x)}{Q(x)}\big]
$$

上面的推導證實了我們的確可以用一個已知的分佈 $Q$ 來嘗試估計 $P$ 之期望值，只要在前面乘上一個 Important weight $=\frac{P(x)}{Q(x)}$ 來做修正。

然而我們要注意的地方是，理論上，我們可以用 $Q$ 來估算 $P$ 的期望值，但是實務上兩者分布還是不能差太多。

原因在於，理論上期望值的計算是抽樣「夠多次」的理想結果，但實務上我們往往無法做到 「夠多次」的抽樣，若 $P$ $Q$ 兩分佈相差太多，這樣的話這樣的兩者期望值就可能不會相等。

舉例來說，如果 $P$ $Q$ $f$ 的分佈如下

![](https://i.imgur.com/5Hbf2Cd.png)

在抽樣數不足的狀況下，對 $P$ 進行抽樣幾乎都會抽到左邊的部分，這樣期望值計算會呈現負值。但是對 $Q$ 進行抽樣則幾乎是抽樣到右邊的部分，則期望值呈現正值。

也因此，在一般實務經驗上，抽樣數不足的前提下，$P$ $Q$ 兩者必須要相似才能確保 Important Sampling 的可行性。


Off Policy
---

回歸正題，我們希望利用一個 $\pi_{\theta'}$ 來收集資料後來一次進行多次的 $\pi_{\theta}$ 參數，那就要利用 Important Sampling 的技巧來進行。

$$
\nabla\bar{R}_{\theta}=\mathbb{E}_{x\sim P_{\theta}(\tau)}\big[R(\tau)\nabla\log P_{\theta}(\tau)\big]\\
=\mathbb{E}_{x\sim P_{\theta'}(\tau)}\big[\frac{P_{\theta}(\tau)}{P_{\theta'}(\tau)}R(\tau)\nabla\log P_{\theta}(\tau)\big]
$$

在上一篇文章的最後其實我們會認為在進行權重更新的時候，我們必須要在每一輪更新時都該給予不同的 Reward。

$$
\nabla\bar{R}_{\theta}\approx\mathbb{E}_{(s_t,a_t)\sim \pi_{\theta}}\big[\big(\sum\limits_{t'=t}^{T_n}\alpha^{t'-t} r_{t'}^n-b\big)\nabla\log P_{\theta}(a_t^n|s_t^n)\big]\\
\overset{let}{=}\mathbb{E}_{(s_t,a_t)\sim \pi_{\theta}}\big[A^{\theta}(s_t,a_t)\nabla\log P_{\theta}(a_t^n|s_t^n)\big]\\
=\mathbb{E}_{(s_t,a_t)\sim \pi_{\theta'}}\big[\frac{P_{\theta}(s_t,a_t)}{P_{\theta'}(s_t,a_t)}A^{\theta'}(s_t,a_t)\nabla\log P_{\theta}(a_t^n|s_t^n)\big]\\
=\mathbb{E}_{(s_t,a_t)\sim\pi_{\theta'}}\big[\frac{P_{\theta}(a_t|s_t)P_{\theta}(s_t)}{P_{\theta'}(a_t|s_t)P_{\theta'}(s_t)}A^{\theta'}(s_t,a_t)\nabla\log P_{\theta}(a_t^n|s_t^n)\big]\\
\Longrightarrow J^{\theta'}(\theta)=\mathbb{E}_{(s_t,a_t)\sim\pi_{\theta'}}\big[\frac{P_{\theta}(a_t|s_t)}{P_{\theta'}(a_t|s_t)}A^{\theta'}(s_t,a_t)\big]
$$

上面的推導有幾個部分要注意 : 
1. 當我們轉換從 $\pi_{\theta'}$ 進行抽樣後，$A^{\theta}$ 也應該同時改為 $A^{\theta'}$ ，因為它是利用抽樣資料進行 accumulated reward，而我們是利用 $\pi_{\theta'}$ 進行抽樣。
2. $\frac{P_{\theta}(s_t)}{P_{\theta'}(s_t)}\approx 1$，這是一個較為直覺的前提，我們可以想像 $s_t$ 的機率應該不會因為 Actor 是哪一個而有差異。

當我們確定了梯度後，便可以推回 Loss Function $J$。


Proximal Policy Optimization (PPO) 
Trust Region Policy Optimization (TRPO)
---

Trust Region Policy Optimization (TRPO) 是 roximal Policy Optimization (PPO) 的前身，這兩者的目的都是在讓 $P_{\theta}$ 與 $P_{\theta'}$ 盡可能的不要相差太大。

$TRPO :$
$$
J^{\theta'}(\theta)=\mathbb{E}_{(s_t,a_t)\sim\pi_{\theta'}}\big[\frac{P_{\theta}(a_t|s_t)}{P_{\theta'}(a_t|s_t)}A^{\theta'}(s_t,a_t)\big]\ ,\ KL(\theta,\theta')<\delta
$$

TRPO 給了一個 Constrain ，希望 $\theta,\theta'$ 這兩個 Actor 的表現可以夠接近，也就是這兩個 Actor 的輸出分佈，計算散度後要小於 $\delta$。( 並非直接計算 $\theta,\theta'$ 的距離 )。概念非常直覺，然而在實際操作上卻是非常不容易，一方面要優化 $J^{\theta'}(\theta)$，又要關心 $KL(\theta,\theta')$ 有沒有小於 $\delta$。

$PPO :$

$$
J^{\theta'}(\theta)=\mathbb{E}_{(s_t,a_t)\sim\pi_{\theta'}}\big[\frac{P_{\theta}(a_t|s_t)}{P_{\theta'}(a_t|s_t)}A^{\theta'}(s_t,a_t)\big]-\beta KL(\theta,\theta')
$$

PPO 將 Constrain 融合到 Loss Function 中直接進行優化，這樣可以直接使用 Gradient Descent 來更新參數，PPO 與 TRPO 的表現在文獻上其實差不了多少，但是在操作上 PPO 相對於 TRPO 簡單非常多。

### Algorithm of PPO

* 初始化 policy $\theta^0$，且給定 $KL_{max}$ 與 $KL_{min}$
* 迭代進行 :
    * 上一輪得到的 $\theta^k$ 與 Environment 互動收集資料 $(s_t,a_t)$，並且計算 $A^{\theta^k}(s_t,a_t)$
    * 優化 $J^{\theta^k}(\theta)=\sum\limits_{(s_t,a_t)}\frac{P_{\theta}(a_t|s_t)}{P_{\theta^k}(a_t|s_t)}A^{\theta^k}(s_t,a_t)-\beta KL(\theta,\theta^k)$ 找出最優解 $\theta^{k+1}$ 作為下一輪收集資料的 Actor 參數。
    * 若 $KL(\theta^{k+1},\theta^k)>KL_{max}$ 則調高 $\beta$，反之則調低 $\beta$。


PPO2
---

$$
J^{\theta^k}(\theta)=\sum\limits_{(s_t,a_t)}\min\Big(\frac{P_{\theta}(a_t|s_t)}{P_{\theta^k}(a_t|s_t)}A^{\theta^k}(s_t,a_t)\ ,\ clip\big(\frac{P_{\theta}(a_t|s_t)}{P_{\theta^k}(a_t|s_t)}A^{\theta^k}(s_t,a_t),1-\epsilon,1+\epsilon\big)A^{\theta^k}(s_t,a_t) \Big)
$$

很恐怖的一個式子，我們分開來解釋好了。

* $\min$ 中的第一項跟第二項其實是一樣的東西，但是第二項其實就是把 $P_{\theta}$ 與 $P_{\theta^k}$ 的比值限制在 $[1-\epsilon,1+\epsilon]$ 之間。
* 當 $A^{\theta^k}\leq 0$ ，表示這樣的情況我們並不樂見，模型會傾向壓低 $P_{\theta}$，然而有第二項的約束，表示我們再怎麼壓低 $P_{\theta}$，其與 $P_{\theta^k}$ 還是要夠接近，比值不能小於 $1-\epsilon$。
* 當 $A^{\theta^k}\geq 0$ ，表示這樣的情況我們非常樂見，模型會傾向拉高 $P_{\theta}$，然而有第二項的約束，表示我們再怎麼拉高 $P_{\theta}$，其與 $P_{\theta^k}$ 還是要夠接近，比值不能大於 $1+\epsilon$。

![](https://i.imgur.com/2YP7Wxq.png)

PPO2 巧妙的避開了計算 KL-Divergence 的問題，讓整個問題變得更加簡單 (?)



註釋
---

[^註1]: 
舉例來說，我們有一個奇形怪狀的曲線，想要求得其面積非常困難。那蒙地卡羅法就認為我們可以在整張圖上面進行均勻的取樣，只要點取得夠多，那麼我們就可以去估計這個曲線下面積 $=$ 圖形內點的數量 / 所有點的數量。