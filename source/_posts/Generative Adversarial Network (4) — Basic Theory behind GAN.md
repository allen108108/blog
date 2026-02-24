---
title: "Generative Adversarial Network (4) --- Basic Theory behind GAN" 
date: 2019-11-06 16:29:34
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning and having it Deep and Structured
image: https://i.imgur.com/RrdITkw.png
mathjax: true
---


在正式進入 Generative Adversarial Network ( GAN, 生成對抗網路 )的理論基礎以前，我覺得必須要先簡單定義出一些概念，以利後面的理論探討。

<!-- more -->

Definition
---
* 隨機變量 Random Variable
在現實生活中，有一些是隨機且可重複出現的現象 ( 例如 : 股市指數、車流量、骰子點數、... ) ，我們將所有可能的重複現象稱作隨機試驗，而樣本空間 $S$ 就是這個隨機試驗的所有可能結果。以樣本空間為定義域的實質函數我們稱之為「隨機變量」$X$
$$
X:S\rightarrow\mathbb{R}
$$
* 機率分佈 Probability Distribution
在數學上，以隨機變量為定義域、$[0,1]$ 為對應域的函數我們稱為機率密度函數 ( Probability density function )，我們也會稱其為機率分佈函數，簡稱「機率分佈」或是「分佈」。而更廣義的來說，機率分佈指的就是一種利用事件機率來描述隨機現象的一種方式。值得注意的地方是，我們可以將「同分佈」(Identically Distributed) 視為是一種特殊的等價關係，兩個分佈相同，其隨機變量並不一定會相同。

* KL散度 Kullback–Leibler divergence (KL divergence)
可以參考 " [Unsupervised Learning – Generation ( PixelRNN、Variational Auto-Encoder ( VAE ) )](http://bit.ly/35nij9b) "一文，在VAE 的討論中，我們有對 KL 散度有簡單的特性描述。
    $$
KL( P\|Q)= KL\ Divergence\ of\ Q\ with\ respect\ to\ P\  \overset{defn}{=}-\sum P(x)\log\Big(\displaystyle{\frac{Q(x)}{P(x)}}\Big)
    $$

* JS散度 Jensen-Shannon divergence (JS divergence)
JS 散度改善了 KL 散度不對稱的缺點，並且將對應域限制在 $[0,1]$ 之間，更能清楚表述兩個不同分佈的異質關係，當 $PQ$兩個分佈越接近，JS值會越接近 $0$，反之，則 JS 會越趨近於 $1$。
    $$
    JS(P\|Q)=\frac{1}{2}KL(P\|M)+\frac{1}{2}KL(Q\|M)\\
    \text{where }M=\frac{P+Q}{2}
    $$


Before GAN
---

在 GAN 之前，人們對於「生成」這件事情已經有一定的期待，當時使用的方法大多是用最大似然估計 ( Maximum Likelihood Estimation, MLE )來進行生成。

最大似然估計的概念很簡單，任何一個機率分佈都可以由一組參數決定，那我們就利用一組抽樣來去試著去找出一組參數可以使這個抽樣的發生機率最大。

為什麼使抽樣發生機率最大就可以了呢 ?簡單來說，抽樣之所以會被抽出來，就是因為這組抽樣發生的機率「應該」要很大，所以才會這麼容易被我們抽出來。

於是我們便可以利用這樣的概念來進行 MLE

$$
\theta^*=\arg\max_{\theta}\prod\limits_{i=1}^{m}P_G(x^i;\theta)=\arg\max_{\theta}\log\prod\limits_{i=1}^{m}P_G(x^i;\theta)=\arg\max_{\theta}\log\sum\limits_{i=1}^{m}P_G(x^i;\theta)
$$

因為似然估計函數通常都是一連串機率函數的乘積，我們會取對數來將其化為連加型態。

我們繼續把上面的 MLE 式子做一點調整

$$
\arg\max_{\theta}\log\sum\limits_{i=1}^{m}P_G(x^i;\theta)\\\approx \arg\max_{\theta}\mathbb{E}_{x\sim P_{data}}[\log P(x;\theta)]\\
=\arg\max_{\theta}\int\limits_{x}P_{data}(x)\log P_G(x;\theta)dx\\
=\arg\max_{\theta}\int\limits_{x}P_{data}(x)\log P_G(x;\theta)dx-\int\limits_{x}P_{data}(x)\log P_{data}(x)dx\\
=\arg\min_{\theta}KL(P_{data}\|P_G)
$$

從這裡我們可以知道，最大似然估計其實就是等價於最小化 KL 散度。


MLE 的一個衍生應用就是用來做圖像生成。單純以圖像生成的任務來說，早在 80年代就開始有人進行相關研究，當時使用 Gaussian Mixture Model[^註1]來對手邊有的圖像進行 MLE 估測真實的分佈。

但這樣的方式其實並無法準確地逼近真實的分佈，導致生成出來的圖像無法做得太清楚。一個可能的原因是我們使用了 Gaussian Mixture Model 實在不太能逼近太複雜的分佈函數，不管我們怎麼調整 Mean 跟 Variance 都很難貼近真實的分佈。



GAN
---

然而，MLE 卻有一些困難點是很難克服的，舉例來說，在一個極為複雜的任務中，$P_G$ 這樣的生成分佈是非常難以估計的，即使我們利用 Gaussain Mixture Model 也仍然無法逼近真實分佈。既然 $P_G$難以計算，那我們整個 MLE 也就無法計算出來，因此，在「生成」這件事情上面，我們需要有更一般化的作法。

### Generator of GAN

在 GAN 中，Generator 其實就是一個 network，而其概念就是希望將一個先驗分佈 ( prior distribution ) 經過 network 進行轉換後，變成另外一個不同的分佈，然後期待這個轉換後的分佈可以逼近真實的分佈。

![](https://i.imgur.com/4ke0YHI.png)

先驗分佈有沒有限制 ? 這倒是不太重要，因為整個 network 會做調整，因此照理來說，任一種先驗分佈應該最後訓練出來的 $P_G$ 應該不會相差太多。

所以我們可以把上圖這個 Generator 公式化呈現

$$
G^*=\arg\min Div(P_{data},P_G)
$$

但現在 $P_G$ 以及 $P_{data}$ 這兩個分佈我們完全不知道它們長得什麼樣子 ( 但我們可以從這兩個未知分佈中取樣 )，也當然無法計算這兩個分佈間的 divergence。GAN 解決這個困境的方式就是使用 Discriminator。

### Discriminator of GAN

前文 " [Generative Adversarial Network (1) --- Introduction](http://bit.ly/2pFLIve) " 一文中我們有介紹 Discriminator 的作用 其實就是利用一個神經網路來對 $P_G$ 以及 $P_{data}$ 的抽樣來進行判斷，看看能不能分辨輸入圖像是生成圖像還是原圖像資料。

在實際操作上，Discriminator 會針對輸入圖像給予一個分數，分數越高就表示圖像越真實，我們最終希望這個 Discriminator 無法判斷此固定的 Generator 生成的圖像。

但其實我們也可以換一個角度來看，當我們在訓練 Discriminator 時，會希望兩種圖像的分數最後會越來越接近，直至分不出來，那麼這樣的分數是不是其實也是某一種 divergence 的量化分數 ?

當我們在針對 Discriminator 做訓練時，其實就是在最大化整個 Loss ( 因為是希望 discriminator 分不出來圖像的真實度 )

$$
\max V(G, D)=\max\Big(\mathbb{E}_{x\sim P_{data}}\log D(x)+\mathbb{E}_{x\sim P_G}\log\big(1-D(x)\big)\Big)\\
=\max\Big(\int\limits_{x}P_{data}(x)\log D(x)dx+\int\limits_{x}P_G(x)\log(1-D(x))dx\Big)\\
=\max\Big(\int\limits_{x}\big(P_{data}(x)\log D(x)+P_G(x)\log(1-D(x))\big)dx\Big)
$$

因為 $D$ 是一個 neural network，所以其實可以逼近任一函數 ( 理想上 )，那麼上述的最大化式，我們可以單看隨機變量 $x$ 即可。簡單來說，我要完成上述的最大化，那其實只要讓 $D$ 在每一個 $x$ 都最大化即可。

$$
\max\Big(P_{data}(x)\log D(x)+P_G(x)\log(1-D(x)\Big)\overset{let}{=}\max\mathcal{L}(D)
$$

簡單從微積分可知，只要令 $\frac{\partial\mathcal{L}}{\partial D}=0$，我們便可以求出可以使其最大化的 $D^*$

$$
D^*(x)=\frac{P_{data}(x)}{P_{data}(x)+P_G(x)}
$$

也可以知道若 $D=D^*$ 則整個 Loss則為

$$
V(D^*,G)=\mathbb{E}_{x\sim P_{data}}\log D^*(x)+\mathbb{E}_{x\sim P_G}\log\big(1-D^*(x)\big)\\
=\int\limits_{x}P_{data}(x)\log \frac{P_{data}(x)}{P_{data}(x)+P_G(x)}dx+\int\limits_{x}P_G(x)\log(1-\frac{P_{data}(x)}{P_{data}(x)+P_G(x)})dx\\
=\int\limits_{x}P_{data}(x)\log \frac{P_{data}(x)\cdot\frac{1}{2}}{(P_{data}(x)+P_G(x))\cdot\frac{1}{2}}dx+\int\limits_{x}P_G(x)\log(1-\frac{P_{data}(x)\cdot\frac{1}{2}}{(P_{data}(x)+P_G(x))\cdot\frac{1}{2}}dx\\
=-2\log2+\int\limits_{x}P_{data}(x)\log \frac{P_{data}(x)}{(P_{data}(x)+P_G(x))\cdot\frac{1}{2}}dx+\int\limits_{x}P_G(x)\log(1-\frac{P_{data}(x)}{(P_{data}(x)+P_G(x))\cdot\frac{1}{2}}dx\\
=-2\log2+KL(P_{data}(x)\|\frac{P_{data}(x)+P_G}{2})+KL(P_G(x)\|\frac{P_{data}(x)+P_G}{2})\\
=-2\log2+2JS(P_{data}(x)\|P_G)
$$

從上面的推導我們可以確定，整個 Discriminator 與 JS-Divergence 有直接的相關，當我們在對 Discriminator 做訓練時，其實就是在對 $P_G$ 與 $P_{data}$ 做 divergence 的計算，這也是為什麼我們會說 discriminator 可以處理 generator 無法計算 divergence 的問題。

推導至此，我們可以把求解 Generator 的式子整理一下

$$
G^*=\arg\min_{G}\max_{D} V(G,D)
$$

每一個 $G_i$，它跟各種不同的 $D$，會產生各種不同的 $V(G_i,D)$，我們也可以從中找到最大的 $V_{max}(G_i,D_i^\*)$，真正我們要找的 $G^\*$，便是這些 $V_{max}(G^i,D_i^*)$ 中最小的那一個。

![](https://i.imgur.com/12A2V5P.png)

### Algorithm

我們已經了解 Generator 跟 Discriminator 的運作過程以及其意義，但整個 GAN 的演算法是藉由 $G$、$D$ 互相更新來進行，這樣的演算法跟 $G^*=\arg\min_{G}\max_{D} V(G,D)$ 是等價的嗎 ?

我們要尋找 $G^*$，上式的 $\min$ 可以用 Gradient Descent 迭代進行，但現在問題在於，要使用 Gradient Descent 的目標函數是一個 $\max$ 函數。雖然這樣的函數整體不可微分，但可以切分成為許多可微分的區間，再針對不同的區間來作微分。

因此，在每一個 Gradient Descent 前，我們都要先找出現在位在哪一個區間段確定微分式，再來做權重更新。

![](https://i.imgur.com/RulPo3N.png)



* **初始化 $G_0$、$D_0$**
* **固定 $G_0$ 訓練出一個可以最大化 $V(G_0,D)$ 的 $D_1^*$**
(*這步驟就是要確定我們該用哪一個部份來微分已進行梯度下降*)
* **利用梯度下降法更新 $G_0$ 的參數產生 $G_1$**
$$
\theta_{G_1}=\theta_{G_0}-\eta\cdot\frac{\partial V(G,D_1^*)}{\partial\theta_G}
$$
* **固定 $G_1$ 訓練出一個可以最大化 $V(G_1,D)$ 的 $D_2^*$**
(*$G$ 更新後，可能會進入另一個區間，採取不同的微分式，所以我們要重新確認一次*)
* **利用梯度下降法更新 $G_1$ 的參數產生 $G_2$**
$$
\theta_{G_2}=\theta_{G_1}-\eta\cdot\frac{\partial V(G,D_2^*)}{\partial\theta_G}
$$
* **重複上述步驟**

### In Practice

![](https://i.imgur.com/wbolNgo.png)


1. **$G$ 的訓練僅更新一次參數，$D$ 的訓練可更新 $k$ 次參數**

從上面的演算法，我們可以直覺地認為，當我們在尋找 $G^*$ 的過程中，其實也是在減少 JS-divergence。
但真的是如此嗎 ? 如果我們 $D$ 與 $G$ 的訓練在每一輪都更新多次參數，其實是很有可能導致整個 JS-divergence 是會上升的。

<img width=500 src="https://i.imgur.com/Q43Ho5d.jpg" >


從 $G$ 的角度來看，因為更新多次，$G$ 的型態可能會有很大的變化，所以 JS-divergence 會很不穩定，為了避免這樣的狀況，我們對 $G$ 的更新就不要這麼多次，以確保整個 $G$ 不會有太大的變化，可以維持 JS-divergence 的穩定下降。

從 $D$ 的角度來看，雖然理想中的 $D$ 可以是任意函數，所以我們一定可以 train 到極致來得到我們真正想要的 $D$。但現實中，Neural Network 的參數是有限的，根本不可能 train 到那個理想中的 $D$，因此在這個步驟中，或許我們根本不用 train 到極致，只需 $k$ 次更新權重即可。

換一個角度想，GAN 的概念就是要讓 $G$ 跟 $D$ 互相追趕，既然如此，每一方都不能跑得出奇地快，讓另一方追不到，這樣是無法達到 GAN 所期待的目標，所以我們必須控制 $G$、$D$ 的更新速度，讓彼此是可以互相追的到的。


2. **Generator Loss function 的調整**

$$
G^*=\arg\min_{G}\max_{D} V(G,D)
$$

從上式，我們可以知道整個 Generator 的 Loss function 應該就是 Discriminator Loss funtion 的其中一個特例。

$$
\tilde{V}=\mathbb{E}_{x\sim P_{data}}\log D(x^i)+\mathbb{E}_{x\sim P_G}\log\big(1-D(x)\big)
$$

但第一項與 Generator 無關，所以我們的目標只有第二項，Loss function 可以再簡化成

$$
\tilde{V}=\mathbb{E}_{x\sim P_G}\log\big(1-D(x)\big)
$$

然而，GAN 的創始人 Ian Goodfellow 認為這樣的 Loss function (下圖紅色曲線) 會對訓練造成困難。


<img width=400 src="https://i.imgur.com/jsUCCQW.jpg" >

從上圖來看，初期容易會有 train 不下去的狀況，因此 Ian Goodfellow 將其稍作調整

$$
\tilde{V}=\mathbb{E}_{x\sim P_G}-\log\big(D(x)\big)
$$

哪一種 Loss function 較好 ? 其實針對這個問題是眾說紛紜，調整後的 Loss function 雖然解決初期梯度趨近於 0 而導致訓練困難的狀況，但是卻容易造成整個 JS-divergence 呈現不穩定的狀態。

有人試圖針對兩種 Loss function 進行比較，結論是兩者都可以 train 得起來，而且 performance 會是接近的。

參考資料
---
1. [GAN系列文(1)–distance of distribution](https://angnotes.wordpress.com/2017/12/04/gan%e7%b3%bb%e5%88%97%e6%96%871-distance-of-distribution/)
2. [KL散度、JS散度以及交叉熵对比](https://blog.csdn.net/FrankieHello/article/details/80614422)


註釋
---
[^註1]: 
參考 " [Unsupervised Learning – Generation ( PixelRNN、Variational Auto-Encoder ( VAE ) )](http://bit.ly/35nij9b) "一文