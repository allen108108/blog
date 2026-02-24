---
title: "Unsupervised Learning (6) -- Generation ( PixelRNN、Variational Auto-Encoder ( VAE ) )"
date: 2019-10-08 00:47:56
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning
image: https://i.imgur.com/hUFoG9E.gif
mathjax: true
---


> " What I cannot create, I do not understand. " --- Richard Feynman

我們想要知道機器是不是真的能學到東西，直覺得就是他能不能夠根據它所學到的東西進行創造，創造出可以以假亂真的成品出來。

<!-- more -->

![](https://i.imgur.com/YUBN2qQ.png =400x)



## PixelRNN

### Recurrent Neural Networks ( RNN , 遞歸神經網路 )

在講 PixelRNN 以前，我們要先簡單的了解 Recurrent Neural Networks ( RNN , 遞歸神經網路 )是什麼。由於在後面的課程中，李宏毅老師會有一整個 Lecture 來講 RNN，這邊我們大概解釋一下 RNN 的精神即可。

簡單的來說，在現實生活中，會有許多決策會由於時間序列推移，導致前一次的決策影響下一次的決策，如此遞迴下去，而 RNN 正是處理這樣問題的神經網路系統。以下是很常看到的例子 : 

![](https://i.imgur.com/gwkOoAb.png)
(圖片來源 : [How Recurrent Neural Networks and Long Short-Term Memory Work](https://brohrer.github.io/how_rnns_lstm_work.html))

每一天的晚餐都會因為前一天晚餐而有所影響，利用 RNN 我們就可以做出當天晚餐的預測。

### PixelRNN

PixelRNN 是一種直觀的方式，利用前一個 pixel 讓機器預測下一個 pixel，藉此完成一整張圖片。



![](https://i.imgur.com/0bpPzmF.png =450x)



雖然說這樣的方式太過直覺，但實際上進行預測，仍能生成出蠻合理的圖片。以下是李宏毅老師利用神奇寶貝 ( 寶可夢，Pokémon ) 來試圖生成新的神奇寶貝圖片。

實務上進行生成會有一些問題，因此李宏毅老師進行了一些原始資料的預處理。

* 原本圖片都是 $40\times 40$ ，這樣維度太高，因此截取中間的 $20\times 20$ 作為輸入
* 進行生成時會發現圖片都偏灰暗，表示其 RGB 預測值對比度低，所以先對原始圖像 pixel 進行 1-of-N Encoding，再將相似色 clustering，最後一共有 167 種顏色分類。[^註1]

最後利用一個 1-layer LSTM 不調太多參數來生成圖片。


![](https://i.imgur.com/JoNfLCr.png)
( 第一列為原始圖片，第二列是給上半部50%讓機器預測下面50%，第三列則是給上半部25%讓祭器預測下面75% )

當然也可以不給機器前一部分，讓它自己畫出整個圖片

<img width=500 src="https://i.imgur.com/eePORC5.png" >


然而，PixelRNN 不只是只能用於圖像上，我們可以利用這樣的概念進行語音或影片的生成。

![](https://i.imgur.com/hUFoG9E.gif)






## Variational Auto-Encoder ( VAE )

先前我們提到 Auto-Encoder 意圖讓原本的圖片 encoder 壓縮成更低維度但仍保有原始資料資訊的 " code " 再由 decoder 還原回去。如果訓練出一組 Auto-Encoder，是否可以輸入一組隨機的 code 利用 decoder 畫出更接近真實的圖片呢 ?

很遺憾的，這樣子的方式生成的圖片都不會太理想。
原因其實可以簡單的這樣解釋，我們隨機給予的 code 並不是由圖片 encoder 而得，所以不見得會帶有 information，那想當然爾，經過 decoder 生成的圖片便不見得會有我們期待的結果出現，甚至可能生成出完全沒有意義的圖片。[^註2]



為了要解決這樣的問題，出現了 " Variational Auto-Encoder ( VAE ) " 

![](https://i.imgur.com/uLklWnd.jpg)

### 從 Gaussian Mixture Model 角度來看 VAE

要理解 VAE 以前，我們先回顧一下，原本的 AE 是將輸入轉換成一個 information preserving " CODE "，也可以看成是一組低維度的隱藏特徵，每一個特徵上給予一個數值。

![](https://i.imgur.com/ZYG02PE.png)
(圖片來源 : [Variational autoencoders.](https://www.jeremyjordan.me/variational-autoencoders/))

而 VAE 則可以看做是這些隱含特徵的 Gaussian Mixture Model，也就是說，我們將每一個隱含特徵上都給予其機率分布，而這些分布經由加權總合後形成一個輸入的分布 。我們也可以從另外一個角度來看 VAE，VAE 相當於在輸入加入一些 Noise ，讓整個模型有更好的泛化能力，能夠解決生成圖像不連續的狀況。

![](https://i.imgur.com/sKl5XPV.png)

### 從 Regularization 角度來看 VAE

我們可以從正則化 (regularization) 的角度來看 VAE。一般的 AE 再進行訓練的時候，為了將 reconstruct error 壓低，因此很容易讓整個模型產生 overfitting 的狀況，這也是為什麼我們隨機給予一個 CODE ，卻很可能會生成出沒有意義的圖片。

![](https://i.imgur.com/bWYFJnx.png)
(圖片來源 :[ Understanding Variational Autoencoders (VAEs)](https://towardsdatascience.com/understanding-variational-autoencoders-vaes-f70510919f73))

 VAE 的產生便是為了加強模型的正則化，讓整個 model 的泛化能力更強。VAE 採取的正則化方式就是讓整個 encoder 不再將輸入 map 到單一個點，而是 map 到一個具有某些限制的常態分佈 (normal distribution)。


<img width=500 src="https://i.imgur.com/V78Tv6m.png" >
(圖片來源 :[ Understanding Variational Autoencoders (VAEs)](https://towardsdatascience.com/understanding-variational-autoencoders-vaes-f70510919f73)) 



### Mathematics in VAE ( 整合許多資料後的筆記 )

VAE 的概念發想是這樣，我們假設有一堆隱含特徵 ( Laten Features ) $z$ 可經由 $P_{\theta}(x\mid z)$ 生成出觀測資料 $x$，其中 $\theta$ 表示生成 $x$ 的模型中所需要的參數。在自然界中，我們可以合理的假設 $z$ 的分布是一個常態分布。

$$
z\sim N(0,I)
$$



![](https://i.imgur.com/phsgs2U.png)

從 Gaussian Mixture Model 來看，我們可以利用各種不同的 Gaussian Distribution 來逼近真實的 $x$ 的分布。

$$
P(x)=\int\limits_{z}P_{\theta}(z)P_{\theta}(x\mid z)dz
$$

$x$ 是我們觀察到的資料，就機率而言，能讓我們看到的資料必然是出現的機率很大才會這麼容易讓我們觀察到，所以我們要做的就是將上式最大化。

$P_{\theta}(z)$ 是一個常態分佈，而給定一個 $z$ 我們要找出 $P_{\theta}(x\mid z)$ 也是沒有問題的。但麻煩的是這個積分我們無法計算，如果 $z$ 的維度很高，我們必須要處理很大量的多重積分，大部分的狀況都無法處理。

不能直接對資料的 Likelihood 作最大化，那換個方向，處理後驗分布 $P_{\theta}(z\mid x)$ 好 了。

$$
P_{\theta}(z\mid x) = \displaystyle{\frac{P_{\theta}(x\mid z)\cdot P_{\theta}(z)}{P_{\theta}(x)}}
$$

這個方向仍然要面對 $P_{\theta}(x)$ 也是死路一條。

在 VAE 原始論文[^註3]中利用 Log data likelihood ，並試圖用另外一個分布 $q_{\phi}(z\mid x)$ 來逼近 $P_{\theta}(x\mid z)$。




![](https://i.imgur.com/Rgoqgqq.png =300x)




$$
\log P_{\theta}(x)=\max\log P_{\theta}(x)\cdot\int\limits_{z} q_{\phi}(z\mid x)dz\\
=\int\limits_{z} q_{\phi}(q_{\phi}z\mid x)\log P_{\theta}(x)dz\\
=\int\limits_{z} q_{\phi}(z\mid x)\log\Big( \displaystyle{\frac{P_{\theta}(x,z)}{P_{\theta}(z\mid x)}}\Big)dz\\
=\int\limits_{z} q_{\phi}(z\mid x)\log\Big( \displaystyle{\frac{P_{\theta}(x,z)}{q_{\phi}(z\mid x)}}\cdot\displaystyle{\frac{q_{\phi}(z\mid x)}{P_{\theta}(z\mid x)}}\Big)dz\\
=\int\limits_{z} q_{\phi}(z\mid x)\log\Big( \displaystyle{\frac{P_{\theta}(x,z)}{q_{\phi}(z\mid x)}}\Big)dz+\int\limits_{z} q_{\phi}(z\mid x)\log\Big( \displaystyle{\frac{q_{\phi}(z\mid x)}{P_{\theta}(z\mid x)}}\Big)dz\\
=\int\limits_{z} q_{\phi}(z\mid x)\log\Big( \displaystyle{\frac{P_{\theta}(x,z)}{q_{\phi}(z\mid x)}}\Big)dz+KL\big(q_{\phi}(z\mid x)\|P_{\theta}(z\mid x)\big)\\
=\int\limits_{z} q_{\phi}(z\mid x)\log\Big( \displaystyle{\frac{P_{\theta}(x\mid z)\cdot P_{\theta}(z)}{q_{\phi}(z\mid x)}}\Big)dz+KL\big(q_{\phi}(z\mid x)\|P_{\theta}(z\mid x)\big)\\
=\int\limits_{z}q_{\phi}(z\mid x)\log P_{\theta}(x\mid z)dz+\int\limits_{z} q_{\phi}(z\mid x)\log\Big( \displaystyle{\frac{P_{\theta}(z)}{q_{\phi}(z\mid x)}}\Big)dz+KL\big(q_{\phi}(z\mid x)\|P_{\theta}(z\mid x)\big)\\
=\int\limits_{z}q_{\phi}(z\mid x)\log P_{\theta}(x\mid z)dz+(-KL\big(q_{\phi}(z\mid x)\|P_{\theta}(z)\big))+KL\big(q_{\phi}(z\mid x)\|P_{\theta}(z\mid x)\big)\\
=\int\limits_{z}q_{\phi}(z\mid x)\log P_{\theta}(x\mid z)dz-KL\big(q_{\phi}(z\mid x)\|P_{\theta}(z)\big))+KL\big(q_{\phi}(z\mid x)\|P_{\theta}(z\mid x)\big)\\
=\int\limits_{z}q_{\phi}(z\mid x)\log P_{\theta}(x\mid z)dz-KL_a+KL_b\\
=\boldsymbol{E}_{q_{\phi}(z\mid x)}\big(\log P_{\theta}(x\mid z)\big)-KL_a+KL_b\\
\geq \boldsymbol{E}_{q_{\phi}(z\mid x)}\big(\log P_{\theta}(x\mid z)\big)-KL_a\overset{let}{=}\mathcal{L}(x,\theta,\phi)\cdots\cdots(1) 
$$
(這裡牽扯到 KL Divergence (KL散度) 的幾個性質，我就放在註釋內 [^註4])



(1) 式中，$\mathcal{L}$ 即為 $\log P_{\theta}(x)$ 的下界，如果我們要最大化 $\log P_{\theta}(x)$ ，我們只要專注在最大化它的下界 $\mathcal{L}$ 即可。

$\mathcal{L}$ 的第一項，我們可以利用 Monte Carlo gradient estimator ( 蒙地卡羅梯度估計 ) 來進行推估，而第二項則因為我們對 $P_{\theta}(z)$ 已經先做了它是 Gaussian Distribution 的假設，所以這項就是在計算 $q_{\phi}(z\mid x)$ 與一個 Gaussian Distribution 有多相似，這會存在一個 Close-Form Solution，因此要對 $\mathcal{L}$ 進行優化是沒有問題的。

關於 $\mathcal{L}$ 我們還可以有一些很直覺的觀察 : 

當我們要對 $\mathcal{L}$ 進行最大化的同時，我們也同時在對於它的第一項最大化，第二項最小化。而第一項代表的意義就是 likelihood 的期望值，第二項則代表的是 encoder network 的分布與 Gaussian Distribution 之間的相似度。

所以當我們在訓練的時候，對於 $\mathcal{L}$ 進行梯度優化的過程中，相對地希望根據 $z$ 生成的 $x$ 可以與我們的觀測越接近越好，同時也希望這個 encoder 分布與 Gaussian Distribution 越接近越好。

理論到這邊差不多告一個段落，我們也可以利用上面的這些論點來製造一個 VAE model 

![](https://i.imgur.com/6glyfff.png)
(圖片來源 : Fei-Fei Li & Justin Johnson & Serena Yeung Lecture 13 - 88 May 18, 2017)


這是一個理論上簡單的VAE model，但仔細看這樣的 model 跟李宏毅課程中 VAE model 還是有幾個不一樣的小地方 : 

1. 在 Encoder NN 後產生的 $\sigma$ 取指數在與一個隨機的 $e$ 相加，目的在對輸入加入一些 Noise，讓整個 model 有抗噪的能力。
2. 在 Decoder NN 後我們一般直接以 $\mu_{x\mid z}$ 來生成 $x$，也因此在很多的資料上，我們在 Decoder 的部分都不會看見其同時生成 $\mu$ 與 $\sigma$ 再來生成 $x$。



### Controlability of VAE 

了解 VAE 的運作之後，可以知道 VAE 的圖像生成具有遷移性，那麼我們便可以調整隱含特徵來了解每一個維度個代表著什麼意義。

若我們只取其中兩維，其他維度均固定

![](https://i.imgur.com/Nrxug09.jpg)

根據這兩維的調整我們可以瞭解其中一個維度代表著可能是微笑程度，另一維度則可能代表的是頭的角度。整個 VAE 便具有比 PixelRNN 更強的解釋性。



### Problems of VAE 

然而，整個 VAE 學習生成圖片的方式是經由數值方式優化的結果，整個 model 其實並沒有學習到 「怎麼生成圖片」這件事情。所以只要在數值優化的過程中，符合優化的結果，圖像如何呈現、是否合理，VAE 本身並不會在意。

![](https://i.imgur.com/IenlZLA.png)

上途中， " Realistic " 與 " Fake " 都與我們現實的圖像僅一個 pixel 的差異，以 VAE 的角度來看，這兩者都是可以接受的成像，但就現實來看 " Fake " 的成像其實很有可能是另外一個類別的圖像。

註釋
---

[^註1]:
李宏毅老師將其資料都放在下列網址，可以進行使用
* Original image (40 x 40):
(http://speech.ee.ntu.edu.tw/~tlkagk/courses/ML_2016/Pokemon_creation/image.rar)
* Pixels (20 x 20):
(http://speech.ee.ntu.edu.tw/~tlkagk/courses/ML_2016/Pokemon_creation/pixel_color.txt)
* Each line corresponds to an image, and each number corresponds to a pixel (http://speech.ee.ntu.edu.tw/~tlkagk/courses/ML_2016/Pokemon_cre)

[^註2]:
可參考 " [花式解释AutoEncoder与VAE](https://zhuanlan.zhihu.com/p/27549418) " 一文

[^註3]: 
Kingma, Diederik P. and Max Welling. “*Auto-Encoding Variational Bayes.*” CoRR abs/1312.6114 (2014): n. pag.

[^註4]: 
KL Divergence ( KLD，KL散度，相對熵 ) : 
簡單來說，這是一個衡量兩個分布相似度的方式，有些人會解釋成兩個分布的「距離」，但如果從 KLD 的性質來看會發現用距離來形容它的確不太恰當。 
    
$KL( P\|Q)=$ KL Divergence of Q with respect to P $\overset{defn}{=}-\sum P(x)\log\Big(\displaystyle{\frac{Q(x)}{P(x)}}\Big)$ 
(a) $KL\geq 0\cdots KL$恆大於零
(b) $KL(P\|Q)\neq KL(Q\| P)\cdots KL$ 不具對稱性