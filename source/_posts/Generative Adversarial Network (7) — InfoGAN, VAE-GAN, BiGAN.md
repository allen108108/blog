---
title: "Generative Adversarial Network (7) --- InfoGAN, VAE-GAN, BiGAN" 
date: 2019-11-17 03:25:36
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning and having it Deep and Structured
image: https://i.imgur.com/RrdITkw.png
mathjax: true
---


在 GAN 中，我們會隨機 input 一組向量，然後生成一個我們要的物件 ( 可能是圖片、文字、語句... )。直覺的，我們總會希望這一組向量的每一個維度都代表某一個特性、性質。

但是現實總是沒有想像中美好。

<!-- more -->

往往我們調整向量的某一個維度，卻很難發現其生成物件的差異。以下圖為例，我們試圖對每一個維度進行調整，生成出來的手寫數字卻幾乎沒有什麼明顯變化，甚至有時候還會出現一些奇怪的狀態 ( 如圖中間的數字 7 )

![](https://i.imgur.com/nb5B4wc.png)

一個可能的解釋是，我們會有這樣的期待，是因為我們將其視為一個規律的、易預測的分佈，但是大多數這些向量的分佈極有可能是非常不規則的。

![](https://i.imgur.com/CsqY3xH.png)


Info GAN
---

Info GAN 便是為了解決這樣的問題而產生的一種 GAN 變體。

### Architecture

首先我們將 vector $z$ 切成兩個部分，第一個部分稱為 $c$ ，而第二個部分則是 $z'$ ( 如果 $z$ 為20維向量，可以前十維視為 $c$，後十維視為 $z'$ )。再來我們將 Generator 的輸出 $x$ 接上一個分類器以及一個 Discriminator。分類器的作用主要是判別 $x$ 是從哪一個 $c$ 向量生成。

$$
Generator+Classifier = Auto-Encoder
$$

![](https://i.imgur.com/x2yPHoj.png)

光只有這樣的 Auto Encoder 很容易產生問題，Generator 只要產生一個直接含有 $c$ 資訊的 $x$ 就可以讓分類器正確分類，即使 $x$ 不夠真實也沒關係。因此 Discriminator 的作用便是要限制 $x$ 除了可以被正確分類外，還不能長得太奇怪，必須要越真實越好。

在實作上，分類器與 Discriminator 會共用參數，只有最後的 Layer 會不同 ( 因為兩者的輸出本來就不一樣 )。

Info GAN 有幾個值得思考的部分 : 

**1. Info GAN 為何可以解決 input features 不明確的問題 ?**
Info GAN 的概念是這樣，如果我們可以利用分類器將 $x$ 解回 $c$，那麼勢必促使 $c$ 的每一個維度都必須要有所代表的意義在其中。這樣我們才能明確且清楚的將 $x$ 解回 $c$。

**2. 怎麼將 $z$ 分成 $c$ 跟 $z'$ ? $c$ 跟 $z'$ 代表的各是什麼 ?**
這個問題其實我們也不會知道，但隨著你給定的切割方式，Network 便會依照你的 $c$ 來進行訓練。也就是說，隨著不同的切割方式，訓練出來的 $c$ 的每一個維度代表的意義可能會不同。既然 $c$ 中的每一個維度都可以代表某些特性，那麼 $z'$ 代表的就是那些無法被表徵的因素。

![](https://i.imgur.com/lvaX2CH.png)

( a ) : Info GAN 改變第一維的數值可以改變數字
( b ) : 一般 GAN 改變第一維的數值影響並不明顯
( c ) : Info GAN 改變第二維的數值可以改變角度
( d ) : Info GAN 改變第三維的數值可以改變手寫粗細


VAE GAN
---
VAE GAN 就是一種 VAE ( Variational Auto Encoder ) 與 GAN 互相輔助的 GAN 變體。關於 VAE 的詳細介紹可以參閱 " *[Unsupervised Learning – Generation ( PixelRNN、Variational Auto-Encoder ( VAE ) )](http://bit.ly/35nij9b)* " 一文。簡單來說，VAE 從原本 AE 內部要求取向量 Latent Code 改變成找出其分佈，來改善 AE 本身的不足。

而 VAE GAN 其實就是將 VAE 的 decoder 視為 generator，後面再接上一個 Discriminator 的結構。

![](https://i.imgur.com/MoSzB5r.png)

VAE 藉由 Reconstruction Error 的優化來生成接近輸入的圖片，但 Reconstruction Error 低並不代表整個生成圖像夠真實，大多數 VAE 生成的圖片都是模糊不清楚的，因此 Discriminator 在這裡就是為了讓生成圖片可以更像真實的圖像。

也就是說，整個 VAE GAN 除了要讓 Reconstruction Error 越小越好，也同時要讓輸出越真實越好。

一般的 GAN ，我們輸入隨機的向量讓它生成出對應的圖像，但 Generator 並沒有看過真正的圖像，因此在訓練過程中，必須花非常多的時間調整參數。然而，VAE GAN 在一開始就見過真正的圖像，因此 VAE GAN 的訓練會相對穩定不少。

### Algorithm of VAE GAN 

* 初始化 $En$ (Encoder), $De$ (Decoder) and $Dis$ (Discriminator)
* 迭代進行 : 
    * 從 Dataset 中隨機抽樣 M 個真實樣本 $\{x^1,x^2,\cdots,x^M\}$
    * 利用 Encoder 生成 M 個 CODE $\{\tilde{z}^1,\tilde{z}^2,\cdots,\tilde{z}^M|\tilde{z}^i=En(x^i)\}$
    * 利用 Decoder 生成 M 個圖像 $\{\tilde{x}^1,\tilde{x}^2,\cdots,\tilde{x}^M|\tilde{x}^i=De(\tilde{z}^i)\}$
    * 從一個先驗分佈 $P(z)$ 中抽樣 M 個 CODEs $\{z^1,z^2,\cdots,z^M\}$
    * 將這些隨機抽樣的 CODEs 利用 Decoder 生成 M 個圖像$\{\hat{x}^1,\hat{x}^2,\cdots,\hat{x}^M|\hat{x}^i=De(z^i)\}$
    * 藉由更新 $En$ 來降低 Reconstruction Error $\|\tilde{x}^i-x^i\|$ ，且使 $P(z), P(\tilde{z}^i|x^i)$ 越接近越好( $KL(P(\tilde{z}^i|x^i)|P(z))$ 越小越好 )
    * 藉由更新 $De$ 來降低 Reconstruction Error $\|\tilde{x}^i-x^i\|$ ，且使 $Dis(\tilde{x}^i)$ 及 $Dis(\hat{x}^i)$ 越大越好
    * 藉由更新 $Dis$ 來增加 $Dis(x^i)$ ，且使 $Dis(\tilde{x}^i)$ 及 $Dis(\hat{x}^i)$ 越小越好
        
PS : 我們也可以將上述 Binary Classificaiton Discriminator 改為 三元分類器。將 reconstruct image 與 隨機生成的 image 視為不同類別。


BiGAN
---

跟 VAE GAN 相同，BiGAN 也是一種更改 Auto Encoder 的 GAN 變體。

BiGAN 將 Auto Encoder 中的 Encoder 與 Decoder 拆成兩個神經網路，Encoder 輸入 image $x^i$ 輸出 CODE $\tilde{z}^i$，而 Decoder 輸入隨機 CODE $z^i$ 輸出 image $\tilde{x}^i$。然後 Discriminator 便是用來判斷 $(\tilde{x}^i,z^i),(x^i,\tilde{z}^i)$ 是來自 Encoder 還是 Decoder。

![](https://i.imgur.com/7JGHowS.png)


### Algorithm of BiGAN

* 初始化 $En$ (Encoder), $De$ (Decoder) and $Dis$ (Discriminator)
* 迭代進行 : 
    * 從 Dataset 中隨機抽樣 M 個真實樣本 $\{x^1,x^2,\cdots,x^M\}$
    * 利用 Encoder 生成 M 個 CODE $\{\tilde{z}^1,\tilde{z}^2,\cdots,\tilde{z}^M|\tilde{z}^i=En(x^i)\}$
    * 從一個先驗分佈 $P(z)$ 中抽樣 M 個 CODEs $\{z^1,z^2,\cdots,z^M\}$
    * 將這些隨機抽樣的 CODEs 利用 Decoder 生成 M 個圖像$\{\tilde{x}^1,\tilde{x}^2,\cdots,\tilde{x}^M|\tilde{x}^i=De(z^i)\}$
    * 藉由更新 $Dis$ 來增加 $Dis(x^i,\tilde{z}^i)$ ，並減少 $Dis(\tilde{x}^i,z^i)$ 
    * 藉由更新 $En,De$ 來減少 $Dis(x^i,\tilde{z}^i)$ ，並增加 $Dis(\tilde{x}^i,z^i)$ (騙過 Discriminator)

這樣的演算法在做什麼事情 ? 

我們可以把 Encoder 的 input 與 output 視為從一個聯合分佈 $P(x,z)$ 抽樣出來的樣本，也可以把 Decoder 的 input 與 output 視為從一個聯合分佈 $Q(x,z)$ 抽樣出來的樣本。

跟一般的 GAN 一樣，在 BiGAN 裡面，Discriminator 也是在計算兩個分佈的散度 (divergence)，只是計算的兩個分佈改成 $P(x,z)$ 與 $Q(x,z)$。而 BiGAN 就是根據 Discriminator 計算出來的散度，來讓 $P(x,z)$ 與 $Q(x,z)$ 越來越接近。

當 $P,Q$ 這兩個聯合分佈越來越接近時，如果 $x^i\approx\tilde{x}^i$，則勢必 $\tilde{z}^i\approx z^i$，也就是說，即使我們將 Encoder 跟 Decoder 分開來訓練，但訓練到 optimal，它們兩個會幾乎等同於一個完整的 auto encoder。

但是要注意，這是指兩者的 optimal 狀況下，兩者會相等，但我們知道，基本上 optimal 的情況幾乎不會出現，所以兩種方式訓練出來的 model 還是會有所不同，在實務上 BiGAN 有較強的語意捕捉能力，也就是生成的圖片就意義上會與原輸入圖片的概念較為相近。


Triple GAN
---

![](https://i.imgur.com/aPAkgEv.png)

在論文 " *Triple Generative Adversarial Nets* " 中提出的這個是 基於 Semi-Supervised Learning 的一種 GAN 變體。

Generator $G$ 跟 Discriminator $D$ 構成了一個 Conditional GAN，這邊倒是沒有什麼問題。Triple GAN 最特別的就是引入了一個 Classifier $C$。一般來說，我們手邊有 Label 的資料通常不會很多，我們可以利用這些已經標記好的資料先訓練出一個分類器，這個分類器的作用就是將生成的資料進行機器標註，在跟原本已經標註好的資料一起丟進 Discriminator。

Domain Adversarial Neural Network
---

![](https://i.imgur.com/02dhSS4.png)

DANN 在之前的課程 " *[Transfer Learning 遷移學習](http://bit.ly/2OExmWb)* " 中有過比較詳細的介紹。

簡單來說，就是要訓練一個特徵萃取器可以不被 Domain 干擾 (所以要試圖騙過 domain classifier) ，另外，萃取出來的特徵還要確認可以被正確分類 ( label predict 的工作 )。

DANN 的訓練過程可以同時進行，也可以如同 GAN 一樣利用迭代的方式進行。( 當然，GAN要同時訓練也是沒有問題，但是穩定性會變得較差 )


Feature Disentangle
---

上面的 DANN 主要就是想引入 Feature Disentangle 這樣的概念，Feature Disentangle 在語音、影像上的使用十分廣泛。

一個特徵向量，其維度通常包含了所有面向的所有特徵資訊。但有時候我們只會對某一個面向的特徵有興趣，因此衍生出 Feature Disentangle 這樣的概念，試圖將不同面向的特徵藉由訓練時就將其作區別。

舉例來說，一段聲音訊號勢必包含了語者的資訊以及發音的訊息...等等。當我們想要做語音轉換時，我們在乎的是語者的資訊，希望只調整跟語者相關的特徵來進行不同語者的模擬。

我們會試著各自訓練一個 encoder 來進行特徵萃取。

![](https://i.imgur.com/lB9474a.png)

光只是這樣，我們還是很難確認、保證一定可以把不同面相特徵精準的分開， 因此還需使用類似 DANN 的概念。另外 train 一個語者分類器，接在發音的 encoder 之後，要盡可能地在訓練過程中，讓分類器分不出來發音這個 Encoder 萃取出來的特徵是哪一個語者發出的。經過這樣的訓練後，我們可以盡量讓發音的 encoder 不會萃取出有關語者的特徵。

![](https://i.imgur.com/JUrbO7G.png)

而下圖則是在 LibriSpeech 訓練的結果 

![](https://i.imgur.com/ZzDZXb5.png)

左上 : 表示不同的發音訊息做分類
右上 : 如果以語者訊息做分類可以知道這些語音是兩個不同人講出來的