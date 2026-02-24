---
title: "Generative Adversarial Network (6) --- WGAN, EBGAN" 
date: 2019-11-11 01:15:45
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning and having it Deep and Structured
image: https://i.imgur.com/RrdITkw.png
mathjax: true
---

前面提到了 GAN 利用 JS-Divergence 來進行量測並且逐步改進模型，這樣的闡述看起來似乎很合理，但事實上卻會遇到一些困境。

<!-- more -->

JS-Divergence 困境
---

利用 JS-Divergence 的量測來推進模型的生成，這是有一個前提條件的。如果兩個分佈之間有某一個程度以上的 overlap， JS-Divergence 才能有所變化，這樣的推進才能持續進行。

![](https://i.imgur.com/zayZiut.png)


換一個角度來看，一個訓練好的 Discriminator 其實就是輸出一個 JS-Divergence，而 Discriminator 其實本身就是一個二元分類器 ( 我們從其 Loss function 也可以發現，其實跟一個二元分類的 Loss function 相同 )。

如果兩個分佈完全沒有 overlap，一個二元分類器達到 100% 準確度是完全沒有問題的。( 舉例來說，要分辨男人女人要達到 100% 會稍微困難，但如果要分辨人跟車子的話就會相對簡單 )，這樣的狀況下，二元分類器的訓練會變得相當困難。

然而，就實際狀況來說，$P_{data}$ 跟 $P_G$ 這兩個分佈可以視為是高維空間中的兩個流型 ( manifold )，高維空間中兩個流型的交集通常是很低的。就算分佈的確有某個程度上的重疊好了，但我們手邊的取樣通常不會夠多到足以反映這樣的重疊狀況。

因此，在這樣的狀況下要訓練是有其困難度。


Least-Square GAN ( LSGAN )
---

當我們在訓練一個 Discriminator 的時候，最後會期待 machine 可以針對兩個分佈進行準確的分類，通常我們使用的會是 sigmoid function 作為 Loss function。

![](https://i.imgur.com/sBvjYwN.png)

但這樣的 Discriminator 會導致在極端的部分微分值為 0，我們沒有辦法引導 $P_G$ 往 $P_{data}$ 的方向來改善。

LSGAN 是一個還算蠻簡單的改善方式，既然 Discriminator 是一個二元分類器，那麼我們把 sigmoid function 利用 linear regression 來取代即可改善這樣的問題。( 其實就是把一個分類問題轉換成迴歸問題 )

Wasserstein GAN (WGAN)
---

### Earth Mover's Distance

這是一個比較簡單的說法。我們想要將 $P_G$ 轉換成 $P_{data}$ 那就必須從 $P_G$ 的樣本中進行轉移，高的分一些給低的，低的分一些給高的。

當然轉移的方法有千百種。

![](https://i.imgur.com/Uj1TRTY.jpg)
(圖片來源 : [Wasserstein GAN and the Kantorovich-Rubinstein Duality](https://vincentherrmann.github.io/blog/wasserstein/))

這些針對一個分佈 $P_G$ 中的每一個「轉移計劃」 (move plan)，我們都能計算出每一個樣本平均轉移的距離 ( 或每一個樣本轉移距離的期望值 )，最後取這些平均(期望)值中的最小值。而這個值就可以定義成 $P_{data}$ 跟 $P_G$ 的距離，這樣的距離我們也稱作 " Wasserstein distance "。

$$
W(P_G,P_{data})=\min\limits_{\gamma\in mp_{P_{G} } }\mathbb{E}_{(x_g,x_{data})\in\gamma}\big( \|x_g-x_{data}\| \big)\\
\text{where } mp_{P_G}\text{ is move plans of }P_G\\
\text{and } \gamma=\{(x_g,x_{data})\ is\ the\ moving\ from\ x_g\ to\ x_{data}|x_g\in P_G,x_{data}\in P_{data}\}\in mp_{P_G}
$$

(這邊我用跟李弘毅課程裡不太一樣的表示式，我覺得會稍微清楚一點。)

我們也可以做成一個 "move plan matrix" 來將整個 move plan 的每一個移動可視化。

![](https://i.imgur.com/wCzIvjw.png)
(圖片來源 : [Wasserstein GAN and the Kantorovich-Rubinstein Duality](https://vincentherrmann.github.io/blog/wasserstein/))
 
利用 move plan 的概念來定義 $P_{data}$ 跟 $P_G$ 的距離好處是，不管兩個分佈距離多遠或多近，這個距離都會有量值，除非完全一樣，不然模型都可以繼續往前推進。

該怎麼樣訓練一個 Discriminator 讓他最後的結果可以是一個 Wasserstein distance ? 
Kantorovich-Rubinstein duality 理論告訴我們可以得到下列的結果 : 

$$
V(G,D)=\max\limits_{D\in 1-Lipschitz}\{ {\mathbb{E}_{x\sim P_{data} } }\big(D(x)\big)-\mathbb{E}_{x\sim P_G}\big(D(x)\big)\}
$$

其中，這裡對 $D$ 有一個比較強的條件限制 : $D\in 1-Lipschitz$[^註1]，$D$ 必須要是一個足夠平緩的連續函數，這樣的限制可以避免上式在訓練過程中 $\mathbb{E}_{x\sim P_{data} }\big(D(x)\big)$ 值被拉到無限大，而 $\mathbb{E}_{x\sim P_G}\big(D(x)\big)$ 被壓到無限小的狀況。

在數學理論上這樣沒問題，但在實作上，要限制 $D$ 一定要是 $1-Lipschitz$ 則是非常困難的。因此 WGAN 的原始論文上利用了 weight clipping[^註2] 來促使 $D$ 有「類似」於 $1-Lipschitz$ 函數的光滑性質。

然而，這樣的方式卻也造成訓練過程中會出現一些問題。

WGAN-GP
---
在 "*Improved Training of Wasserstein GANs*" 論文中提出了一個改善原始 WGAN 的方法 : Gradient Penalty，而這樣的方法也被稱作 WGAN-GP。

論文中利用了 Lipschitz continuity 一個特別的性質 : 

$$
f\in K-Lipschitz\Longleftrightarrow \|\nabla f\|\leq K,\ \forall x
$$
 
所以我們可以將 $V(G,D)$ 改寫成

$$
V(G,D)=\max\limits_{\|\nabla D\|\leq 1}\{ {\mathbb{E}_{x\sim P_{data} } }\big(D(x)\big)-\mathbb{E}_{x\sim P_G}\big(D(x)\big)\}
$$

我們可以用 penalty 的概念來做逼近

$$
V(G,D)=\max\limits_{\|\nabla D\|\leq 1}\{ {\mathbb{E}_{x\sim P_{data} } }\big(D(x)\big)-\mathbb{E}_{x\sim P_G}\big(D(x)\big)\}\\
\approx\max\limits_{D}\{ {\mathbb{E}_{x\sim P_{data} } }\big(D(x)\big)-\mathbb{E}_{x\sim P_G}\big(D(x)\big)-\lambda\int\limits_{x}\max(0,\|\nabla D(x)-1\|)dx\}\\
$$

然而，我們手中只有少許的資料，根本無法對所有的 $x$ 做積分[^註3]，因此在 WGAN-GP 中提出了另一個分佈 $P_{penalty}$ ，將後面 penalty 的積分值用這個分佈的梯度期望值來取代

$$
V(G,D)\approx\max\limits_{D}\{ {\mathbb{E}_{x\sim P_{data} } }\big(D(x)\big)-\mathbb{E}_{x\sim P_G}\big(D(x)\big)-\lambda\mathbb{E}_{x\sim P_{penalty} }\big(\max(0,\|\nabla D(x)-1\|)\big)\}
$$

這個 $P_{penalty}$ 就是指 $P_{data}$ 跟 $P_G$ 中資料點倆倆連線中間的點的分佈。

![](https://i.imgur.com/KrNocR3.png)

但實驗上反映，條件不僅要限制 gradient $\leq 1$ (過大要懲罰)，gradient $\geq -1$ 也應該要懲罰，也就是說，gradient 應該要越接近 1越好。 

所以我們必須再一次改寫上面的式子

$$
V(G,D)\approx\max\limits_{D}\{ {\mathbb{E}_{x\sim P_{data} } }\big(D(x)\big)-\mathbb{E}_{x\sim P_G}\big(D(x)\big)-\lambda\mathbb{E}_{x\sim P_{penalty} }\big(\|\nabla D-1\|^2\big)\}
$$

我們可以給出整個 WGAN 的演算法 : 

* Discriminator 的 sigmoid 拿掉
* 初始化 $G$ 與 $D$ 的參數 $\theta_g$ 與 $\theta_d$
* 循環迭代進行

    * Updeate $D$
        * 從 database 中隨機抽取 $m$ 個真實樣本 $\{x^1,x^2,\cdots,x^m\}$
        * 從我們給定的分布中隨機抽取 $m$ 個向量 $\{z^1,z^2,\cdots,z^m\}$
        * 藉由 $\tilde{x}^i=G(z^i)$ 生成 $\{\tilde{x}^1,\tilde{x}^2,\cdots,\tilde{x}^m\}$
        * Loss Function $\tilde{V}=\displaystyle{\frac{1}{m}\sum\limits_{i=1}^{m}D(x^i)-\frac{1}{m}\sum\limits_{i=1}^{m}D(\tilde{x}^i)}$
        * 藉由最大化 $\tilde{V}$ ( Gradient Ascent ) 來更新權重 $\theta_d$ : 
$$
\theta_d = \theta_d + \eta\cdot\nabla\tilde{V}(\theta_d)
$$
    * Update $G$
        * 從我們給定的分布中隨機抽取 $m$ 個向量 $\{z'^1,z'^2,\cdots,z'^m\}$
        * Loss Function $\tilde{V}=-\displaystyle{\frac{1}{m}\sum\limits_{i=1}^{m}D(G(z'^i))}$
        * 藉由最小化 $\tilde{V}$ ( Gradient descent ) 來更新權重 $\theta_g$ : 
$$
\theta_g = \theta_g - \eta\cdot\nabla\tilde{V}(\theta_g)
$$

Energy based GAN (EBGAN)
---

Junbo Zhao 等人於 2016年發表的論文 " *Energy-based Generative Adversarial Network* " 中，將 Discriminator 視為一個能量函數。其概念上是真實的圖形能量為 $0$，越靠近真實圖像的能量就越接近 $0$，越不像真實圖像的能像就會越遠離 $0$。這樣的方式讓 Discriminator 不限於只是一個二元分類器，而可以是一個可訓練的能量函數。

訓練的過程，由 Generator 生成圖像，讓 Discriminator 去算其能量，跟 GAN 的概念一樣，藉由 Discriminator 輸出的能量值來推進 Generator 的進化，彼此互相訓練，讓 Generator 可以生成越來越低能量的圖像。

論文內利用一個 Auto Encoder 來實現 EBGAN 的 Discriminator。


![](https://i.imgur.com/LiTcrDD.jpg)

EBGAN Discriminator 的輸出不再是分類分數，而是一個能量值，而這樣的分數可以拿來表徵與完美圖片的「距離」。

EBGAN 有什麼好處 ? 

以往傳統的 GAN 上面訓練，Discriminator 必須要經過多輪訓練後 Generator 才能夠生成比較好的圖片，但由於 EBGAN 使用的是 Auto Encoder ，不需要 training data 即可訓練，因此我們可以先做 pre-train，再拿來結合 Generator 來做 EBGAN，也因為 Pre-train 的關係，在初期的訓練過程中，Generator 已經可以生成品質很不錯的圖像。

這樣的 EBGAN 仍然有兩個部分需要注意

1. 我們要將 Discriminator 拿來結合 Generator，還是必須要讓 Discriminator　可以分辨出真實及生成的圖片，才能推進 Generator 的「進化」。然而我們訓練的 Discriminator 不就正是盡可能地將輸入圖像能量值趨近於 $0$ 嗎?
為了解決這樣的問題，論文內在 Auto Encoder 中間會加上 Regularization，相當於限制 Auto Encoder 的表述能力，這樣也限縮了真正能使能量值趨近於 $0$ 的資料分佈，

2. 因為我們將 Discriminator 的輸出變成能量值，越接近真實圖像的能像值越接近 $0$，但那些生成的圖片，真的很不像真實圖像的，它的能像值計算可以到 $\infty$，這樣也的確會讓訓練過程出現問題。
因此在論文中，會給予能量值一個 Margin $m$，相當於將 Discriminator 輸出的能量值限制在 $[0,m]$

![](https://i.imgur.com/QhtPZOv.png)


參考資料
---
1. [WGAN-GP方法介绍](https://zhuanlan.zhihu.com/p/52799555)
2. [Generative Adversarial Nets[EBGAN]](https://www.cnblogs.com/shouhuxianjian/p/8476695.html)
3. [EBGAN, LSGAN, BEGAN](https://saicoco.github.io/EBGAN/)
4. [LS-GAN（损失敏感GAN）](https://zhuanlan.zhihu.com/p/26007545)
5. [DCGAN、WGAN、WGAN-GP、LSGAN、BEGAN原理总结及对比](https://blog.csdn.net/qq_25737169/article/details/78857788)

    
註釋
---
[^註1]: 
Lipschitz continuity 是一個比一般連續還要更強的光滑性條件，它限制了函數的起伏。$f:X\subseteq\mathbb{R}\rightarrow\mathbb{R}\ s.t.
 \forall x,y\in X\ ,\ |f(x)-f(y)|\leq K|x-y|,then\ f\in K-Lipschitz$
從函數圖形來看，符合 Lipschitz continuity 的函數，一定存在一個雙圓錐，其頂點沿著函數圖形平移時，函數曲線(面)都會在此雙圓錐之外。而 K 值決定了此雙圓錐的開口大小。

<img width=400 src="https://i.imgur.com/BnRfbAx.gif" >

[^註2]: 
Weight Clipping 技術其實意義上等同於 Gradient Clipping，限制 $-c\leq weight\leq c$，實踐的方式就是如果 $weight\geq c$，則 $weight=c$ ; 如果 $weight\leq c$，則 $weight=-c$

[^註3]: 
事實上在ICRL 2018 年的論文 "*Spectral Normalization for Generative Adversarial Networks*" 中提到可以利用 Spectral Normalization 使所有 $x$ 的 gradient 都為 1。