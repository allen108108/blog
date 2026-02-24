---
title: "Generative Adversarial Network (3) --- Unsupervised Conditional GAN" 
date: 2019-10-29 08:45:04
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning and having it Deep and Structured
image: https://i.imgur.com/RrdITkw.png
mathjax: true
---

當我們在使用 GAN 進行生成時，其實有一個很大的問題是，帶有 label 的訓練資料並不容易取得。

舉例來說，如果我們想要讓一張照片利用 Generator 生成梵谷畫風格，訓練資料根本無法取得，因為我們可以有很多的照片，但卻沒有梵谷畫這些風景的資料 ; 又或者，我們想做一個A轉換成B人聲的模型，通常很難讓 B 照著 A 所念的文字照著念一次 ( 可能不同語言、或是收集上有困難...等因素 )，這時候的 GAN 就如同 Unsupervised Learning 任務一般。

<!-- more -->

![](https://i.imgur.com/F8nWYYg.png)

現在實務上大多使用兩種方式來達成這種 Unsupervised Conditional GAN

1. Directed Transformation
比較簡單的轉換 ( input 及 output 差別不大 ) 在實務上利用這種直接轉換的方式是很可能成功的，像是畫風轉換、質地上的轉換或文本的轉換這種都屬於比較簡單的轉換。
2. Projection to Common Space
倘若 input 及 output 差別很大，那我們就要先訓練一個 encoder 那我們就要先訓練一個 encoder 萃取出 input 特徵後再利用一個 decoder 來進行轉換。

Directed Transformation
---

利用 GAN 直接轉換，有時候會遇到之前提到的問題 : Generator 只要騙過 Discriminator 即可，並不用考慮 input。因此我們勢必要在 GAN 架構下做一些處理。

### 1. Ignore

不管上述的問題，直接訓練，這在某些狀況下其實是有機會可以成功的。原因在於，我們經過 Generator 生成的圖像，正常應該不會跟 input 相差太多才是。因此直接處理應該有機會可以得到還算不錯的成果。

### 2. Embedding

將輸入及輸出通過一個 pre-train 好的 encoder ，讓它們各自產生一組 CODE。在整個訓練過程中，除了要騙過 Discriminator 之外，也要讓兩個 CODE 的 Loss 越小越好。

![](https://i.imgur.com/s5LUyst.jpg)

### 3. Cycle GAN ( Disco GAN, Dual GAN )

Cycle GAN 在許多領域中都被廣泛使用，其概念還算直覺。

既然怕生成出來的圖像跟 input 不同，那麼我們就利用另外一個 Generator 來將生成圖像還原回去。最後我們最小化輸入圖像以及最後還原的圖像之間的 Loss 來限制中間的生成圖像要跟 input 相似。

![](https://i.imgur.com/kq9QcQ0.png)

計算還原後圖像與 input 的 Loss 我們稱之為 Cycle Consistency。

兩個 Generators $G_{X\rightarrow Y},G_{Y\rightarrow X}$ 確保轉換後的圖像不會生成出與 input 完全無關的圖像。而 Discriminator $D_Y$ 則確保生成出來的圖像會是我們想要的目標 ( 風格 )。

有趣的是，在同時間有另外兩篇 Paper 提出了幾乎同樣概念的 GAN 變體 :  Disco GAN, Dual GAN，名稱雖不同，但做法幾乎一樣。

我們也可以利用雙向的 GAN 來訓練出一組兩 domains 之間互轉的 Generators
，以及可以判定是否為兩個 domains 圖像的 Discriminators。

![](https://i.imgur.com/pz4yjXV.png)

但，Cycle GAN 在使用上其實有一個疑慮 : Steganography

![](https://i.imgur.com/IFRPguM.png)

上圖是一個空拍圖轉成地圖再轉回空拍圖的 Cycle GAN，我們可以發現紅框處竟然可以幾乎無差別的還原出原本 input 的細節。這不免讓人懷疑是否在生成的時候，並不是真正的對圖像進行 reconstruct (重構)，而只是對 input 的資訊進行隱藏而已。

這樣有什麼問題 ?

我們原本的假設是，如果 cycle consistency 很低，那麼我們應該可以確保中間生成的圖象跟原圖像很接近。但如果 Generator 善於把資訊藏起來待還原時再恢復，那它便可以在中間生成時隱藏許多的資訊，讓中間生成的圖像跟 input 相差很多，但還原出來還是可以夠接近 input。

這還是一個尚待解決的問題。

### 4. Star GAN

如果我們要進行兩個以上的 Domain 互轉，使用 Cycle GAN 則必須要訓練出 $2\cdot{ {n} \choose {2} }$ 個 Generators。但 Star GAN 則是利用一個 Generator 便可以進行互轉。



![](https://i.imgur.com/Y12U9wi.png =400x)



整個 Star GAN 的過程大概如下圖所示 : 

![](https://i.imgur.com/xgPxQaK.png)

* 先初始化一個 Generator 及 Discriminator
* 先利用初始化的 Generator 隨意生成圖片然後對 Discriminator 做訓練，可以判斷圖像的真偽並且進行 domain 的分類判定。
* 此 Discriminator 不動，讓 Generator 訓練對輸入圖像進行 domain 的轉換 ( 輸入是一張圖像及指定的 domain )，並且再利用同一個 Generator 把轉換後的圖像再還原回去 ( 輸入為轉換後的圖像以及最初的 domain )，最後計算還原後的圖片以及最初的圖片之間的 Loss，並使其最小化。( 這不就是 Cycle Consistency 嗎? )
* 最後把轉換後的圖片丟給 Disriminator 來做判定，試圖要騙過這個 Discriminator，藉此訓練 Generator。
* 重複上述步驟進行對抗生成的訓練。

從上面可以知道，其實跟 Cycle GAN 差不多，最主要的差異就是 Generator 變成一個可以一對多的網路結構。


下圖是一個性別以及外觀上互換 (Celeb label) 的實例

![](https://i.imgur.com/WoGa5Jd.png)

我們先要了解，每一個 Celeb label 相當於是一個 domain，因此真正要計算的話，這一個例子裡面應該有 32 個 domains 可以進行互換。

在這個例子中，目標是要將一個棕色髮色的女人轉換成黑色頭髮的年輕男子，因此先訓練一個可以進行辨別的 Discriminator，再訓練可以多個 domain 互轉的 Generator 試圖騙過 Discriminator。

同樣的方法也可以用來生成表情 ( RaFD label )。

![](https://i.imgur.com/A0WnnNA.png)



Projection to Common Space
---

如果 input domain 跟 output domain 相差很大，使用上述直接轉換的方式要成功並不容易，因此這邊介紹第二種方法 : Projection to Common Space

這個方法的想法是，將兩個 domains 各自圖像利用 Auto Encoder 將隱含特徵抽取出來，並且希望這兩個隱含特徵向量可以一樣，這樣我們就可以藉由這樣相同的隱含特徵進行轉換。

當然，我們也可以加上一個 Discriminator 來使輸出圖像更加清楚。( VAE GAN )

![](https://i.imgur.com/NWxs70y.png)


現在的問題就是，兩個 domains 各自訓練出來的 Auto Encoder 怎麼將中間的 CODE 使其一樣，或是越接近越好 ? 

### 權重共享 : Couple GAN

一個方法是將各自的 Encoder 跟 Decoder 在 CODE 附近的權重共享

![](https://i.imgur.com/SoWAq20.png)

藉由這樣的方式，我們可以期待產生的 Latent Space 會是夠接近的，這樣的方式最極端的狀況就是共用同一個 Encoder，只是在輸入圖像時，同時給一個 flag，讓 Machine 知道這張圖片是來自哪一個 domain。

### 在 CODE 處加上 Discriminator

我們可以在 CODE 地方加上一個 Discriminator 來便是這一個 CODE 是來自哪一個 domain。在各自訓練 Auto Encoder 的同時也可以一起訓練，試圖讓各自 CODE 可以騙過 Discriminator。

![](https://i.imgur.com/PBdASOI.jpg)


### Cycle Consistency : Combo GAN

或是利用 Cycle GAN 的概念，將兩個 Auto Encoder 視為 $G_{X\rightarrow Y},G_{Y\rightarrow X}$ 串在一起做訓練，並計算最後的 Cycle Consistency。

![](https://i.imgur.com/V49Dr5e.jpg)


### Sementic Consistency : X GAN

Cycle Consistency 計算的是 Pixel level 的差異，對於圖像來說，計算的是「表象」上的差異性。但如果將 consistency 的計算改在 Latent CODE 上面進行，則會偏向於計算「語義上」、「特徵上」的差異性。

所以大致上跟上述的 Cycle Consistency 概念差不多，只是將 consistency 計算移至 CODE 上計算，稱為 Sementic Consistency

![](https://i.imgur.com/BXwWDgF.jpg)

Projection to Common Space 這樣的方法，我們可以應用在聲音轉換的領域，保留被轉換者的語調、聲音特徵，進行人聲的轉換。




參考資料
---
1. [异父异母的三胞胎：CycleGAN, DiscoGAN, DualGAN](https://zhuanlan.zhihu.com/p/26332365)
2. [GAN论文研读(四)-----Cycle GAN 与 Star GAN](https://blog.csdn.net/openSUSE1995/article/details/79235374)