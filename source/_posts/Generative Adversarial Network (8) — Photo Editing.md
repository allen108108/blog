---
title: "Generative Adversarial Network (8) --- Photo Editing" 
date: 2019-11-18 10:43:51
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning and having it Deep and Structured
image: https://i.imgur.com/RrdITkw.png
mathjax: true
---

當我們訓練出一個 GAN 之後，在之前的部分有說過，我們總會希望可以藉由調整 Vector 的某些維度來改變生成圖像的一些特徵。例如 : 金髮轉黑髮、年輕便年老或是男人變女人...等等。但我們也有說過，我們很難知道每一個維度所代表的意義為何，也有可能根本每一個維度都無法區分出屬於哪一個圖像特徵。

<!-- more -->

除了前面幾種藉由改變 GAN 的結構來強迫讓機器學得一種特徵屬於一個維度的表徵方式之外，其實也可以從我們手邊的資料跟固有的 GAN 模型來嘗試去回推圖像特徵的調整方式。

GAN + Auto Encoder
---

我們可以將既有的 Generator 作為 Decoder (固定)，重新訓練一個 Encoder ( 可以拿 Discriminator 作為初始值 )　串成 Auto Encoder。

![](https://i.imgur.com/FvWrRIZ.png)

利用這樣的 Auto Encoder 來將圖像進行特徵萃取。如果我們想知道長短髮轉換的方式，我們可以將現有的長短髮圖像集中，並利用 AE 找出它們的隱含特徵，進行下列方式運算 : 

![](https://i.imgur.com/qGa2jNe.png)

將長短髮向量各自平均後相減，計算出由短髮轉換成長髮的向量 $z_{long}$，便可將短髮向量加上 $z_{long}$ 轉換成長髮向量。

![](https://i.imgur.com/t67GKuo.png)

智能 Photoshop
---

以下是一種智能 Photoshop 的 Demo，在一張現有的圖片上加上一些條件限制，它可以直接轉換成符合我們要求的模樣。

{%youtube 9c4z6YsBGQ0 %}

背後的概念大概是如下 : 
1. 原始圖片先進行 embedding 找出它在 CODE spsce 的位置。
2. 移動一小步使其跟原本的 CODE 距離不會太遠，但又要符合使用者給出的條件。
3. 再來進行圖像生成。

但現在的問題是 (1) 怎麼做 Embedding 找出 CODE ? (2) 怎麼樣能夠符合使用者條件下又能與原圖像接近 ?


### Back to Latent CODE 

**方法一 :** 

直接用 Grqdient Descent 解一個直接解一個優化問題

$$
z^*=\arg\min\limits_{z} L(G(z),x^T)
$$

要解這樣的問題除了用 Pixel-wise 的方式計算兩個向量間的距離外，也可以用一個 Pre-train 好的 Neural Network 來做 embedding，並且希望 $G(z)$ 跟 $x^T$ 越接近越好。


**方法二 :**

利用我們前面 GAN+AutoEncoder 的方式來處理。


**方法三 :**

結合上述兩種方式，將方法二的結果視為方法一的初始值再進行優化。


### How to editing photo ?


假設，原圖片經由上面的方式找出 Latent CODE $z_0$，接下來我們希望的是找出一個 $z^*$ 可以滿足使用者下的條件 $U$，又可以盡可能地接近 $z_0$，更重要的是生成的圖片還要夠真實。


$$
z^*=\arg\min\limits_{z} U(G(z))+\lambda_1\|z-z_0\|-\lambda_2 D(G(z))
$$

( 兩個 $\lambda$ 的符號相反，因為這兩個條件的優化目標、方向是相反的 )

![](https://i.imgur.com/yK5cgx7.jpg)


其他 GAN 應用
---

### Super-resolution

將低解析度的圖片變成高解析度，主要利用低解析度跟高解析度的 Piar 利用 GAN 來訓練。一般來說，大概都直接拿高解析度圖片來改成低解析度資料。

以下是論文 " *Photo-Realistic Single Image Super-Resolution Using a Generative Adversarial Network* " 的實驗結果 : 

![](https://i.imgur.com/mE8IZva.png)



### Image Completion

Image Completion 技術是將圖片挖空，然後讓電腦自行補上挖空的部分，方法跟 Super resolution 的方式雷同。

以下是論文 " *Globally and Locally Consistent Image Completion* " 的實驗結果 : 

![](https://i.imgur.com/s2NJrse.jpg)

{%youtube 5Ua4NUKowPU %}

