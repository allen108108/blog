---
title: 卷積神經網路 (Convolutional Neural , CNN) 
date: 2019-10-07 03:08:31
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning
image: https://i.imgur.com/z4wLQJn.png
mathjax: true
---


>* **本文內容以李宏毅 Machine Learning (2017) 第十講 : Convolutional Neural Network ( CNN ) 為主，搭配參考資料編輯而成。**
> 
>*  **本篇圖片部分出自 Machine Learning (2017) 課程內容講義**
>

<!-- more -->
當我們剛開始接觸深度學習的時候，最常看到的例子便是使用 MINST 資料庫進行手寫數字的辨識。概念如下圖所示，將所有像素灰階數值壓成一維資料後再丟進全連接層進行學習。


<img width=500 src="https://i.imgur.com/zVa11Vr.png" >
(圖片來源 : 3Blue1Brown Youtube : [究竟神經網路是什麼？  第一章 深度學習](https://youtu.be/aircAruvnKk))


如果在進行一般的圖片辨識時，我們不會使用上面的方式，因為這樣子做會有幾個問題 : 

1. 在一般的圖片辨識問題中，事實上會有一些 pattern 可能會出現在圖片中的某個部位，且這樣的 pattern 可能由許多個鄰近像素構成，如果依照上面的方式會破壞這樣的 pattern 結構。

2. 全連接層搭配高像素的圖片，會讓整個計算成本大幅增加。

基於上面幾個理由便衍伸出 Convolutional Neural Network ( CNN ) 卷積神經網路來進行圖像辨識。

 

整個 CNN 結構主要分成幾個部分 : 
**卷積層 ( Convolution layer )、池化層 (Pooling layer) 以及最後一個全連接層 ( Fully Connected layer )**。

<img width=500 src="https://i.imgur.com/77bPBr3.png" >


Convolution Layer 卷積層
---

卷積層主要是由許多不同的 kernel 在輸入圖片上進行卷積運算。

什麼是卷積 ? 在這邊卷積其實就是兩個步驟組成的運算 : 滑動 + 內積，利用 filter 在輸入圖片上滑動並且持續進行矩陣內積，卷積後得到的圖片我們稱之為 feature map。[^註1]



![](https://i.imgur.com/JTqr2Yw.png)
![](https://i.imgur.com/v4VM3qu.gif)
(圖片取自 : [[ 機器學習 ML NOTE ] Convolution Neural Network 卷積神經網路](https://medium.com/%E9%9B%9E%E9%9B%9E%E8%88%87%E5%85%94%E5%85%94%E7%9A%84%E5%B7%A5%E7%A8%8B%E4%B8%96%E7%95%8C/%E6%A9%9F%E5%99%A8%E5%AD%B8%E7%BF%92-ml-note-convolution-neural-network-%E5%8D%B7%E7%A9%8D%E7%A5%9E%E7%B6%93%E7%B6%B2%E8%B7%AF-bfa8566744e9))

kernel 我們也稱為 filter ， 我認為 filter 這樣的概念可以讓人更體會卷積層的作用，一般我們使用修圖軟體的各種濾鏡功能即是不同的 kernel 在圖片上作用後的結果。
( 想知道許多不同的 filter 造成的效果可以參考 Wikipedia -- [Kernel ( image processing ) ](https://en.wikipedia.org/wiki/Kernel_%28image_processing%29)  )

在全連接層神經網路中經由學習不斷更新的權重，在 CNN 這邊指的就是 filter ，如上圖 3X3 的 filter 內就相當於有 9 個權重。我們可以想像，CNN 訓練的過程就是不斷地在改變 filter 來凸顯這個輸入圖像上的特徵。

不過，CNN 還是有一些值得注意的地方 : 

**1. 每一層卷積層的 filter 不會只有一個**

我們引用 CNN 的經典論文 GradientBased Learning Applied to Document Recognition 中的 LeNet-5 結構，第一層的卷積層就給了 6 個 3X3 kernel，也就是說在這同一層中就有 54 個權重需要同時更新，而這六個 filter 也會相對應給出六個 feature map 。


![](https://i.imgur.com/z4wLQJn.png)
(圖片來源 : Yann LeCun Leon Bottou Yoshua Bengio and Patrick Haffner.(1998).*GradientBased Learning Applied to Document Recognition*[^註2])



**2. 彩色的圖片同時會有 RGB 三個 channel**

也就是說，我們每一張圖片都會有深度，既然輸入圖有深度，那麼我們的 filter 也相同的會有一個深度，每一個 channel 會對應著 filter 的深度做卷積，filter 每一個深度的數值可能會不一樣。



![](https://i.imgur.com/0xP0Ins.png =350x)



攤開看大概會像這樣子



![](https://i.imgur.com/3YHkC4o.gif =400x)
(圖片取自 : [Undrestanding Convolutional Layers in Convolutional Neural Networks (CNNs)](http://machinelearninguru.com/computer_vision/basics/convolution/convolution_layer.html))



**3. 經過卷積層過後的 feature map 會變得比原尺寸小**

這其實不難看出來，只要親自做一輪卷積運算便可以很容易地發現這個結論，如果不想要讓整個 feature map 縮小，我們可以使用 padding 的方法 ( 於輸入圖片的周圍補上一圈 0 )，來避免 feature map 縮小的狀況。



![](https://i.imgur.com/m1qlenn.jpg)





**4. filter 滑動的步伐 (Stride) 不一定必須是 1**

當我們滑動的步伐加大，那麼出來的 feature map 尺寸會相對縮得更小



![](https://i.imgur.com/UST7EeO.png =350x)




### 卷積層的特色

**1. 可以保留圖片中的空間結構，並從這樣的結構中萃取出特徵**

![](https://i.imgur.com/xRiuMSa.png)


每一個 filter 我們可以視為是一個圖像上的 「pattern」，經過卷積運算的滑動過程，其實就是在檢驗每一個圖片的部分是否有符合這個 pattern 的區域，有相同 pattern 出現的區域便在 feature map 上有相對高的分數，最後我們也是經由整體分數的高低來檢驗並調整 filter。



![](https://i.imgur.com/8nFL49Q.png =350x)
( feature map 中的數值總和我們可以當作這個 filter 的 degree，我們希望找到的 filter 能讓 feature map 的數值和最大 )


**2. 利用權重共享的方式減少參數**

從李宏毅 Machine Learning 課程中的講義可以看出來「權重共享」的概念是什麼意思。當我們進行 filter 滑動的時候，也是表示我們同一組權重正作用在不同區域，這樣的過程我們這個 filter 並沒有改變，也就是權重是不會變動的。

下圖中，李宏毅將整個卷積層排列成類似全連接層的型態，這樣可以很明白的看出共享權重的過程。



![](https://i.imgur.com/EyuzvUr.png =350x)



*在這裡我想補充一點，我自己在自學卷積這一部分時，一直對於卷積過後的數值與呈現出來的 feature map 之間的關聯性沒有很明確的理解，以上圖來看，經過卷積運算後的 3X3 矩陣表示的到底是什麼東西 ?*

*以下幾個卷積層操作影片可以幫助我們對於整個卷積層的運作有更深刻的理解*[^註3]


{%youtube KiftWz544_8%}
(影片來源 : Henry Warren -- [The Convolution Layer (CNN Visualization)](https://www.youtube.com/watch?v=KiftWz544_8))



{%youtube z6k_RMKExlQ%}
(影片來源 : Gene Kogan -- [ml4a @ itp nyu :: 03 convolutional neural networks](https://www.youtube.com/watch?v=z6k_RMKExlQ&) )








---
### [ 補充 ]

最近在看 " *Network In Network* " 以及 " *Going deeper with convolutions.* " 這兩篇論文的時候，又重新審視了一下自己對於 CNN 的了解，這才發現，如果沒有辦法理解 $1\times 1$ Convolution Layer 的作用的話，對於 CNN 的了解確實是不足的。因此特別在這裡做了一個補充。

**「你認為，經過一個卷積層後的輸出，究竟有幾張圖像 ?」**

![](https://i.imgur.com/YdM0KdM.png)

我們拿 " *Gradient-Based Learning Applied to Document Recognition* " 論文內的 LeNet-5 model 來看看。就單看第一層卷積層吧，一張 $32\times 32$ 的輸入，經過了五個 $28\times 28$ 的 Kernel，形成了五張 feature maps，所以第一層卷積層應該是輸出五張圖像，看上圖也似乎是如此沒有錯。

不過真的是這樣嗎 ? 

事實上，這五張 feature maps 的確是卷積後的結果，但並非整個卷積層最後的輸出，卷積層還必須將這五張 feature maps 疊合在一起，這才是真正第一層卷積層的輸出。

**也就是說，輸出仍然為一張圖像，但是變厚了**

我自己會把它想像成，一張完整的圖像，會是許多特徵疊加後的結果，而卷積層也會做到這個工作。

所以，當我們卷積層的 Kernel 數量越多，輸出的圖像就會越厚 ( 專業的講法是這圖像的深度變得更深了 )，相對的參數也會越多。

![](https://i.imgur.com/laTqCxB.jpg)
( 圖片來源 : [卷積神經網路(Convolutional neural network, CNN): 1×1卷積計算在做什麼](https://medium.com/@chih.sheng.huang821/%E5%8D%B7%E7%A9%8D%E7%A5%9E%E7%B6%93%E7%B6%B2%E8%B7%AF-convolutional-neural-network-cnn-1-1%E5%8D%B7%E7%A9%8D%E8%A8%88%E7%AE%97%E5%9C%A8%E5%81%9A%E4%BB%80%E9%BA%BC-7d7ebfe34b8) )

### $1\times 1$ 卷積層到底在做什麼 ?

如果我們把卷積層只看作是一個逐步向量內積的過程，那一定會覺得 $1\times 1$ 卷積層只是單純的純量相乘的結果而已。當然，就運算上的確只是如此，但是如果我們了解了真正卷積層的運作，你就不難發現，$1\times 1$ 卷積層還能有一個非常好用的功能。

**降維 Dimension Reduction**

![](https://i.imgur.com/7iq1A6K.jpg)
( 圖片來源 : [卷積神經網路(Convolutional neural network, CNN): 1×1卷積計算在做什麼](https://medium.com/@chih.sheng.huang821/%E5%8D%B7%E7%A9%8D%E7%A5%9E%E7%B6%93%E7%B6%B2%E8%B7%AF-convolutional-neural-network-cnn-1-1%E5%8D%B7%E7%A9%8D%E8%A8%88%E7%AE%97%E5%9C%A8%E5%81%9A%E4%BB%80%E9%BA%BC-7d7ebfe34b8) )

上圖作者認為這樣的表示方式比較容易理解，但我個人還是比較建議以下圖的方式來呈現，原因無它，下圖比較能真正表現出圖像「深度」的概念。



![](https://i.imgur.com/88V2T2X.png =400x)



經過了一個 $1\times 1$ 卷積層後，圖像大小不變，但卻能有效的控制參數的增加，這樣的方式在 " *Network In Network* " 以及 " *Going deeper with convolutions.* " 這兩篇論文中有效的控制了整個模型的複雜度。

---

 
Pooling Layer 池化層
---

![](https://i.imgur.com/gs0K20B.png)

池化層的主要概念是，當我們在做圖片的特徵萃取的過程中，圖形的縮放應該不會影響到我們的目的，經由這樣的 scaling 我們也可以再一次的減少神經網路的參數。

常用的 pooling 方式有 Max pooling 與 Average pooling : 
如同卷積層，pooling 也有一個 kernel，也是在輸入圖像上進行滑動運算但和卷積層不同的地方是滑動方式不會互相覆蓋，且運算是以 kernel 涵蓋範圍的最大值 ( Max pooling ) 或平均值 ( Average pooling )。

![](https://i.imgur.com/Ok5FuhJ.png =450x)
(圖片來源 : Quora -- [What is the benefit of using average pooling rather than max pooling?](https://www.quora.com/What-is-the-benefit-of-using-average-pooling-rather-than-max-pooling))

許多研究都發現，Max pooling 的效果會比 Average pooling 來的好，因此現在大多使用 Max pooling 。( LeNet-5 則是使用 Average pooling )

### 池化層的特色

**1. 藉由對輸入圖片的 subsampling 來減少參數，減少計算成本**

從 pooling 過程來看，如果原本的輸入圖片，經過 2x2 pooling kernel 之後，整個圖片的尺寸將會縮小 $\displaystyle{\frac{1}{2}}$，如此一來，計算成本也將節省一半。

**2. 具有特徵不變性**

以下面三個圖來看看，經過池化後如何保持特徵不變性，圖一平移，圖二旋轉以及圖三是縮小。

我們可以發現，經過池化後，這些特徵最後呈現的 feature map 會是相同的，這也符合了我們當初對於池化層的期待與概念，經過 subsampling 後相對於原圖並不會有什麼太多的改變。

![](https://i.imgur.com/crpxYlI.jpg)
(圖片來源 : 知乎 -- [CNN网络的pooling层有什么用？](https://www.zhihu.com/question/36686900))


**3. 提高 Receptive Field (中國翻譯為 : 感受野)**

Receptive field 指的是我們在 feature map 中的一個像素內能看到輸入圖像區域

![](https://i.imgur.com/QCJRHPL.png)

不管卷積層或是池化層都能夠有提高 Receptive field 的作用，所謂的「提高 Receptive field」，依上圖來看就是當我們經過池化或是卷積層後，feature map 對應的原圖區域就會越大，也就是我們可以「見微知著」，從一個點看到整個原圖蘊含的資訊。

這個有趣的影片我認為也可以展示 pooling layer subsampling 的概念

{%youtube fApFKmXcp2Y%}


Fully Connected Layer 全連接層
---

這邊的全連接層跟我們進行手寫辨識的方式一樣，說穿了就是一個分類器，把我們經過數個卷積、池化後的結果進行分類。

這邊可能會有人有一個問題 : 「 不是說全連接層會破壞圖片的空間結構嗎 ? 」

是的，但是別忘了，我們已經經過數個卷積池化層了，現在每一個節點已經包含了空間結構的資訊在裡面，這個時候進行全連接層的分類就不會有破壞空間結構的問題。

在全連接層有趣的地方是，如果我們把這一層的輸出視覺化，直覺上會認為他輸出的視覺化圖形會接近我們認知的分類。舉例來說，我們丟手寫數字進去經過 CNN 最後全連接層的輸出，理應出來的圖形會接近我們認知的數字型態。然而我們實際將全連接層最後輸出視覺化會是下圖。



![](https://i.imgur.com/VeWOyba.png =350x)



問題在於，CNN 與人腦的視覺模式有很大的不同，這樣的結論在許多的論文中都有被提及。[^註4]



我們如果將上圖加上ㄧ些 constrain (Regularization)，來告知系統說有些點不是我們需要的部分，就可以稍微看出一些端倪。



![](https://i.imgur.com/EwFeKmh.png)




CNN 的應用
---

我們了解到 CNN 對於空間結構上有非常好的表現，因此，需要保留空間結構的學習模式， CNN 就有機會在這樣的條件下訓練出非常好的 model。

### 應用一 : Alpha Go

![](https://i.imgur.com/TDpCK3e.png)

如圖所示，棋盤本身就具有一定的空間結構，而 CNN 也的確在下棋這件事上面有著很不錯的表現，我們可以餵進許多的棋譜讓它學習到各種狀況下的下棋策略。

然而，我們在 CNN 上面有經過 Max pooling subsampling 來縮小尺度保留特徵，棋盤似乎無法這樣做。

我們從 Aloha Go 的結構也發現，他們的確將 pooling layer 拿掉了。

![](https://i.imgur.com/IjfZQbT.png)


### 應用二 : 語音辨識

![](https://i.imgur.com/xhPyOiU.png)

由於人類的說話本身也具有結構，因此利用語音建立的 Spectrogram 來進行 CNN 的訓練。

### 應用三 : 文本分析

![](https://i.imgur.com/gwyPhGV.png)

跟上面語音的概念相似，文本也具有類似的空間概念，因此也可以利用 CNN 來訓練出一個不錯的文本分析模型。

註釋
---

[^註1]:
在數學上對於卷積的定義更加抽象且準確，而卷積這樣的概念也被廣泛的應用在許多領域中，而在 " 知乎 -- [如何通俗易懂地解释卷积？](https://www.zhihu.com/question/22298352) " 這篇文章中對於卷積有多面向的討論，非常建議大家可以閱讀。

[^註2]: 
論文連結 : Yann LeCun Leon Bottou Yoshua Bengio and Patrick Haffner.(1998).[*GradientBased Learning Applied to Document Recognition*](http://yann.lecun.com/exdb/publis/pdf/lecun-01a.pdf)

[^註3]:
除此之外，對於卷積層的運作也可參考以下文章 :
     (1) CH.Tseng -- [OpenCV的捲積操作](https://chtseng.wordpress.com/2019/06/02/opencv的捲積操作/?fbclid=IwAR0S1Kdf3pAbrwmiRrXNNSqxIf_GUnat0IcMFkCMxjAy6Z8HYi8kFLXwK0I)
     (2) Steven Shen -- [入門深度學習 — 2](https://medium.com/@syshen/%E5%85%A5%E9%96%80%E6%B7%B1%E5%BA%A6%E5%AD%B8%E7%BF%92-2-d694cad7d1e5)

[^註4]: 
參考論文 :
     (1) Anh Nguyen,Jason Yosinski and Jeff Clune .(2014).[*Deep Neural Networks are Easily Fooled: High Confidence Predictions for Unrecognizable Images.*](https://arxiv.org/pdf/1412.1897.pdf)
     (2)Karen Simonyan, Andrea Vedaldi and Andrew Zisserman .(2013).*[Deep Inside Convolutional Networks: Visualising Image Classification Models and Saliency Maps.](https://arxiv.org/pdf/1312.6034.pdf)*