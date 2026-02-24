---
title: "Unsupervised Learning (5) -- Generation ( 初探 Generative Adversarial Network ( GAN ) )"
date: 2019-10-08 00:47:00
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning
image: https://i.imgur.com/UgI1J3c.jpg
mathjax: true
---


>這裡我選擇的標題是 「初探」GAN，
>因為在李宏毅的課程中，對於GAN還有許多補充的課程，
>在這篇文章就先以一些簡單的基礎概念先讓大家了解什麼是 GAN，

<!-- more -->

GAN 的概念其實就像是獵物與天敵之間的關係，獵物想要經由演化來讓自己可以避免自己被天敵攻擊，而天敵也會因為獵物的演化學會哪些才是他的獵物。



![](https://i.imgur.com/a8Lk5mK.png =450x)



在 GAN 的系統中，獵物與天敵就相當於 Generator 與 Discriminator 之間的關係。



![](https://i.imgur.com/fzY6qTh.png =450x)



那麼 Generator 與 Discriminator 之間是怎麼進行轉變的呢 ?

## Training GAN Model

### Step 1 : 

首先，Generator V1 ( 後面簡稱 $G_{V1}$ )就像是 VAE 中的 Decoder 一樣，先從一個 Distribution 中 Sample 出一組資料 $D_1$。

![](https://i.imgur.com/wMCYkTn.jpg)


### Step 2 :

將這隨機生成的資料 $Data_1$ 給予 Label=0 ( False )，另外將真實資料 $Data_T$ 給予 Label=1 ( True )。利用這些資料進行 Supervised Learning 來訓練出一個 Neural Network Discriminator V1 ( $D_{V1}$ )


![](https://i.imgur.com/q9HKJxz.png)

### Step 3 : 

將 $G_{V1}$ 與 $D_{V1}$ 串在一起形成一個大型 NN，固定住 $D_{V1}$ 的所有參數，並將這整個大型 NN 的預測都要是 1 (True)，藉此來更新 $G_{V1}$ 的參數變成新的 $G_{V2}$。這裡直觀的解釋就是，我要讓 $G_{V1}$ 所形成的所有圖像都能騙過 $D_{V1}$。



![](https://i.imgur.com/xP3P2Y7.jpg =200x)



然後重複 Step 1-3，直至收斂 ( Discriminator 再也分不出來 )。

下圖是 GAN 原始論文[^註1]中提到的訓練過程圖例 : 


![](https://i.imgur.com/jUPqZmj.png)

我們考慮一個已經接近收斂的 GAN Model : 

$x$ 與 $z$ 之間的箭頭代表著對 $z$ 做均勻抽樣，但經過一個不均勻分布作用於其上會轉變到 $x$ 的那些部分，我們可以將箭頭視為是一個函數關係 $x=G(z)$

(a) $G$ 的分布 ( 綠色 ) 已經接近資料真實的分布 ( 黑色 )，而 $D$ 是一個部分準確的分類器 (藍色) 能分辨黑綠兩分布的樣本。

(b) 開始更新 $D$ ，使其能判斷樣本 : 
    $D(x)=1$，當 $x$ 落在 $G(z)$ 低密度區
    $D(x)=0$，當 $x$ 落在 $G(z)$ 高密度區

(c) 藉由 $D$ 引導 $G$ 的分布往真實資料的分布靠近。

(d) 經過無數次迭代，$G,D$ 足夠好的時候，便無法再優化，此時 $G$ 的分布會與真實資料分布相同。此時 $D(x)=\frac{1}{2}$ , $\forall x$

## Moving on Code Space

再 VAE 的時候談到了生成圖像的遷移性質，在論文 " *Unsupervised Representation Learning with Deep Convolutional Generative Adversarial Networks* " 中也提到了 GAN 的遷移性。

![](https://i.imgur.com/UgI1J3c.jpg)


最左邊的一整行 9 個圖像，代表著我們從 Code Space 中隨機抽象 9 個樣本所生成的圖像。當我們將 code 從第一個 code 漸漸移動到第二個 code，再漸漸移動到第三個 code ....

我們可以發現整個模型有著 Smooth Transition (平滑過渡)，意即整個圖像具有遷移性質。我們從第六列的圖像上來看，可以發現圖像從一個沒有窗戶的房間逐漸變成有一個大窗戶的房間。再看看最後一列，我們可以看到一台電視怎麼轉變成為一個窗戶。


## Problems of GAN

在實務上我們會遇到幾個問題 : 

* GAN 非常難 train
* 在 GAN 中，沒有一個可以可以衡量整個生成是否足夠好的標準，可以做的就是持續地對生成圖像做監控。
* 當 Discriminator 判別不出來的時候，有時並不代表整個 Model 已經可以生成非常真實的圖片，或許只是 Discriminator 太弱，抑或是剛好 Generator 找到一個特例使得 Discriminator 無法判別。

註釋
---

[^註1]: 
Mirza, M. & Osindero, S. (2014), '*Conditional Generative Adversarial Nets*' , cite arxiv:1411.1784 .