---
title: "Unsupervised Learning (2) -- AutoEncoder"
date: 2019-10-08 00:36:33
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning
image: https://i.imgur.com/2mVam3c.png
mathjax: true
---

## Auto-Encoder

Auto-Encoder 也是 Dimension Reduction 的一種應用，當我們資料位處高維度，我們會希望可以將其縮到低維度的空間，當然也希望可以將其還原到原本的維度空間中。而這中間則利用一個 Neural Network Encoder / Decoder 來進行轉換。

<!-- more -->

![](https://i.imgur.com/mNDt7Of.png =400x)



既然是 NN 結構，那我們就乾脆把 Encoder/Decoder 串在一起，利用 Backpropagation 一起訓練。

![](https://i.imgur.com/YlvhSgI.png)

理想的狀態，我們會希望 Encoder 的 weight matrix 與 Decoder 的 weight matrix 互為轉置，但實務上我們基本上不用給這樣子的條件，反正就丟進去一起訓練就好。

了解這樣的架構之後，我們可以知道 Auto-Encoder 有幾個特點 : 
*  Auto-Encoder 其實就是在學習出一個近似 identity 的 function，亦即 $g(x)=\hat{x}\approx x$
*  輸入的神經元個數 = 輸出的神經元個數
*  最中間的這一層，有人稱之為 " Coding layer " 則帶有輸入資料的重要資訊在其中，我們稱之為 information preserving。

## Deep Auto-Encoder

當然，我們也可以將整個 Auto-Encoder 的結構增加更多 Layer 進去，便成為我們所稱的 Deep Autp-Encoder

![](https://i.imgur.com/2mVam3c.png)

有些資料會說 Deep Auto-Encoder 結構必須要是對稱的，但李宏毅老師在這邊有特別強調，結構並不一定要對稱。

Deep Auto-Encoder 事實上是一個非常強的模型，decoding 後可以非常接近輸入資料

<img width=500 src="https://i.imgur.com/ECKfgx1.png" >


## Denoising Auto-Encoder

<img width=500 src="https://i.imgur.com/ygojQXm.png" >


Denoising Auto-Encoder 就是將原本的資料加了一些 Noise 進去，在丟到 Auto-Encoder 內進行訓練。這邊稍微要注意的是，雖然輸入是有雜訊的資料，但我們輸出要近似的對象應該要以原始資料為主。

$$
g(x')=\hat{x}\approx x\ ,\ where\ x'=x+Noise
$$

利用 Denoising Auto-Encoder 便可以做出一個可以處理 Noise 的 Model 了。

## Auto-Encoder Application

### Text Retrieval

在 Natural Language Processing (NLP) 中，處理文本很常會用到的方式就是 Bag of Word (其運作方式就類似 One-Hot Encoding)，利用這樣的方式將 Document 向量化後便可以去跟我們的 Query　進行　Similarity 的比較。


![](https://i.imgur.com/1tPYLb1.png =250x)
![](https://i.imgur.com/BdxZbWm.png =250x)



通常一個 Document 的維度都會非常高，因此我們可以利用 Auto-Encoder 來降維。我們再前面有提到說 Auto-Encoder 有 information preserving 的特色，那麼我們理應可以直接利用 Document 以及 Query 降維後的 code 進行比較。


### Similar Image Search

相同的，我們也可以利用 information preserving 的特性進行相似圖檢索。

![](https://i.imgur.com/hgYiWko.jpg)

假如我們今天單純利用像素的歐式距離來計算相似度，就會得到上圖上方的那些結果。很明顯的，這些結果並不是我們想要的。

如果我們今天換成利用 code 來進行相似度檢索，雖然說都沒檢出 Michael 的照片，但是檢索出來的圖片就會合理許多。

### Auto-Encoder for CNN



<img width=500 src="https://i.imgur.com/JXZ051Z.png" >

我們當然也可以將 Auto-Encoder 應用在 CNN 上，但問題就在於，怎麼做 Decode ? 我們分別從 Pooling 跟 Convolution 兩個部分分開來討論 Decode 的方式。

#### Max Pooling (subsampling) -- Unpooling (upsampling)

現在 CNN 比較常用的是 Max Pooling，因此我們這裡就討論 Max Pooling 的狀況 ( 其實 Aveerage Pooling 的方式並不會有什麼差別 )。

![](https://i.imgur.com/5mm4WZg.png)

在 Pooling 的過程中，我們只要記得取 Maximun 的位置，在 Unpooling 時，將上一層的值分別補回 Pooling 取值的位置即可 ( 其他位置補 0 )。



![](https://i.imgur.com/6CuKVq8.png =300x)



這樣的方式，可以確保能夠最大限度的還原 information


#### Deconvolution

" Deconvolution " 這樣的名詞其實很有問題，就字面上來看，它應該是一個「逆卷積」的運算，然而我們在這一個部分用的其實也是一種 upsampling，而運算仍然是一般的卷積。( 因此，現在有許多人使用 Transpose Convolution 這樣的稱呼來取代 Deconv )

![](https://i.imgur.com/sqQCeq8.jpg)

我們從 Neural Network 的角度來看 ( 而非以滑窗的概念來看 )，在 Deconv 階段，直覺的我們會希望在這個階段的 Neurons 可以藉由當初 Conv 的連結，連結到下一層的　Neurons，而重疊的部分也是順理成章的相加即可。(上圖中)

然而，其實可以發現到，Deconv 的運算也就只是 Padding 後再做一次 Convolution 而已，它跟 Convolution 的差異只是在於權重的順序相反。

也因為從 NN 角度來理解這樣 Deconv 的概念，我們可以很清楚地了解權重的改變，這樣也不難理解 " Transpose Convolution " 為什麼比 " Deconvolution " 更適合代表這樣的操作了。

### Pre-training

我們一樣要再來強調一次 Auto-Encoder 的重點 " information preserving "。

在 Neural Network 中，我們要從頭訓練一個 model，有時候必須要耗費很大的時間、計算成本，很多時候我們會盡量去找一組比較好的初始值，讓整個訓練可以比較快速完成。

而 Auto-Encoder 便是可以幫助我們順利完成這樣工作的工具之一。

因為 Auto-Encoder " information preserving " 的特性，因此我們如果可以保留作為初始值，讓每一層都能 preserve information ，這樣的網路應該不算太差，那我們在後續的訓練就不用浪費太多時間，而這樣的方式我們稱之為 " Pre-Training "

![](https://i.imgur.com/RJRsomN.jpg)


### Decoder Part .....

前面所有的應用，我們都是使用 Encoder 部分，但後半部的 Decoder 也是有一些有趣的特性可以來應用。拿 MNIST 的例子來看，Decoder 的部分就可以生成出各種數字。

![](https://i.imgur.com/aGaysLs.png)

上圖右，為上圖左紅框每隔一定距離的資料點生成出來的圖形。因為 Decoder、Encoder 都是串在一起做訓練，所以 Decoder 部分也會有 information preserving 的特性，利用這樣的性質，想當然爾我們可以將 code 生成接近 input 的圖像。