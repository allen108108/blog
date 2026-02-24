---
title: "Generative Adversarial Network (1) --- Introduction" 
date: 2019-10-14 23:45:41
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning and having it Deep and Structured
image: https://i.imgur.com/RrdITkw.png
mathjax: true
---


在之前李宏毅的 Machine Learning 中已經有對 [ Generative Adversarial Network ( GAN, 對抗生成網路 )](/yq6MGU5USb68EkOGlApe0g) 有了初步的介紹，而本課程接續著這些基礎，將會對 GAN 有比較深入的討論。

<!-- more -->

## Algorithm

在 Machine Learning 中並沒有對整個演算法有太多的著墨，而留在這裡做比較詳盡的解釋。

![](https://i.imgur.com/b8foMIJ.png)

既然 GAN 在概念上就是利用 $G$ ( generator ) 跟 $D$ ( discriminator ) 的對抗及演化來增強 performance，那我們便可以把整個過程分成兩個步驟然後持續循環進行。

雖然說是說兩個互相對抗，但其實實踐的方式就是講這兩個 Neural Network 串聯在一起訓練這樣。

### initial $G$ & $D$

首先對整個 GAN 進行 weight 及 bias 的初始化，得到 $G_{v_0}$ 與 $D_{v_0}$

### Step 1. : Fix G , update D

初始化後的 $G_{v_0}$ ，如果給予一個隨機向量，必然會產生表現不是很好的圖片，在這樣的狀況下，我們可以把真實的圖片與產生出來的圖片作為訓練資料來訓練 $D_{v_0}$，最後我們可以得到一個進化後的 $D_{v_1}$

### Step 2. : Fix D , update G

接著我們固定住 $D_{v_1}$ 來對 $G_{v_0}$ 進行訓練，訓練的目的就是要讓 $G_{v_0}$ 產生的圖片可以騙過 $D_{v_1}$，這樣的訓練後我們可以得到一個進化後的 $G_{v_1}$，如此循環進行下去。

### Algorithm

我們可以將上面的過程得到下列 GAN 演算法 : 

* 初始化 $G$ 與 $D$ 的參數 $\theta_g$ 與 $\theta_d$
* 循環迭代進行
    
    * Updeate $D$
        * 從 database 中隨機抽取 $m$ 個真實樣本 $\{x^1,x^2,\cdots,x^m\}$
        * 從我們給定的分布中隨機抽取 $m$ 個向量 $\{z^1,z^2,\cdots,z^m\}$
        * 藉由 $\tilde{x}^i=G(z^i)$ 生成 $\{\tilde{x}^1,\tilde{x}^2,\cdots,\tilde{x}^m\}$
        * Loss Function $\tilde{V}=\displaystyle{\frac{1}{m}\sum\limits_{i=1}^{m}\log D(x^i)+\frac{1}{m}\sum\limits_{i=1}^{m}\log\big(1-D(\tilde{x}^i)\big)}$[^註1][^註2]
        * 藉由最大化 $\tilde{V}$ ( Gradient Ascent ) 來更新權重 $\theta_d$ : 
$$
\theta_d = \theta_d + \eta\cdot\nabla\tilde{V}(\theta_d)
$$
    * Update $G$
        * 從我們給定的分布中隨機抽取 $m$ 個向量 $\{z'^1,z'^2,\cdots,z'^m\}$
        * Loss Function $\tilde{V}=\displaystyle{\frac{1}{m}\sum\limits_{i=1}^{m}\log D(G(z'^i))}$
        * 藉由最小化 $\tilde{V}$ ( Gradient descent ) 來更新權重 $\theta_g$ : 
$$
\theta_g = \theta_g - \eta\cdot\nabla\tilde{V}(\theta_g)
$$


## Regard GAN as structure Learning

從 GAN 的概念上看來，它的確是一種類似 Structure Learning 的學習過程，藉由 Discriminator 的對抗，讓 Generator 可以生成一個足具結構性的結果 ( 可以是一張圖片、一段摘要、... )

甚至我們也可以利用 GAN 來輸入文字，生成對應的圖片

![](https://i.imgur.com/lmffToS.png)

### The Challenge of Structure Learning

#### One-Shot (Zero-Shot) Learning

在 GAN 中，在 Training Data 中的每一個類別幾乎都可能只出現過一次，而且待預測的問題也通常不會出現在 Training Data 中，因此，我們可以把 GAN 看作是一個極端的 One-Shot (Zero-Shot) Learning。

這也代表著， Machine 在整個 GAN 學習過程中，必須讓自己的學習能力更強，以圖片生成來說，GAN 不是只需要根據 pixel 生成圖片這樣的局部處理能力，它更必須要有一個看清楚全局的能力，這樣才能在沒有看過的資料底下做事適當的反應。

## Can machine learn only by Generator or Discriminator ?

### Learn by Generator

這樣的方式並不是完全沒有，在 Deep Learning 中，我們碰過的 Auto Encoder[^註3] 或是 Variational Auto Encoder[^註4] 其實就是只有 Generator 的技術。

但是這樣的技術其實有一些問題 : 
1. Reconstruction Error
在進行 reconstruction error 時，通常系統考慮的是 pixel level 的錯誤程度，但以一張圖片來說，有時候的確會出現雖然錯誤的 pixel 較多，但是並不影響圖片本身的資訊，然而，有些時候僅出現極少量的錯誤 pixel 卻能讓整張圖片呈現出截然不同的資訊。


<img width=500 src="https://i.imgur.com/mhIxYqP.jpg" >



2. Component independent
在 NN 運作的過程中，最後一層輸出的往往每一個 component 之間是互相獨立的，也就是在整張圖之中的每一個 pixel 之間互不相關，這樣容易導致整張圖片生成出來會容易出現局部零碎的部分無法相互連接。這也不是沒有解決方法，在最後的 layer 後多加幾層 layers 讓訊息可以互相連結即可。這也表示，像這種單純只用 Generator 訓練的模型，往往需要更深的架構才有機會有較好的表現。


### Learn by Discriminator

Discriminator 本身是一個評價系統，要讓它做出一張好的圖片生成其實並不是它所擅長的工作，但是它可以利用它自己的評價系統來「挑」出一張分數夠高的圖片輸出。

$$
\tilde{x}=\arg\max\limits_{x\in X} D(x)
$$

[ Note ] : 我們暫且不討論這個方式的可行性，就先假設這樣的演算法是存在的。

在這樣的演算法底下，其實會出現一個比較根本性的問題 : 我們該怎麼訓練它 ?
我們手上有的 Data 就是一堆正常的、好的圖片，這樣是無法作訓練的，Machine 只會從這些資料裡面學到不管輸入什麼圖片就通通給高分就好了。

我們還需要的是一些「夠好的反例」。



<img width=500 src="https://i.imgur.com/AeUAWMj.png" >


現在的問題就是要訓練它輸出圖片，但問題卻又回到希望它先輸出一些夠好的、卻又明顯是人造的圖片來訓練它生成圖片 ? 這明顯的就是雞生蛋蛋生雞的狀況。

目前能做的頂多就是給一堆 noise ，非常差的圖片給它訓練，但這樣又容易使 Machine 對那些稍微好一點的人造圖片給予非常高的分數，這樣的判斷不是我們期待的。

修正一下，利用 iterative 的方式來進行訓練 : 

* 先隨機生成一些圖片作為反例


<img width=500 src="https://i.imgur.com/F75Z83Q.png" >

* 以這些反例與我們手邊的正常圖片訓練一個 discriminator 可以區分「目前」這種正反例型態的圖片。
* 利用這樣的 discriminator 經由 $\tilde{x}=\arg\max\limits_{x\in X} D(x)$ 來生成另外一些再好一點的圖片作為反例
* 重複上面的步驟持續訓練

整個上述訓練的過程其實就是一種分布上的調整，經由不斷的 smaple，把分布逼近於真實的分布狀況。


<img width=500 src="https://i.imgur.com/58sibZ4.png" >


### Pros and Cons

||Generator|Discriminator|
|-----|-----|----|
|Pros|便於產生圖片|可以考量到全局|
|Cons|僅學習到整個結構的表象，並沒有學習到其內涵，因此每個 pixel 間都是個別獨立的。|argmax 問題通常不易解，而且反例圖片不容易取得。|


## Generator + Discriminator

GAN 就是結合上述的優點所形成的方法，利用 Generator 來克服 Discriminator 不易產生圖片的難處，也利用 Discriminator 來幫 Generator 做到全局的掌控。

![](https://i.imgur.com/hOKQRJ5.png)

在 Mario Lucic 的論文 " [*Are GANs Created Equal? A Large-Scale Study*](https://arxiv.org/abs/1711.10337) " 中針對了幾種 GAN 方式與 VAE 一起做一個整合式的比較，從上圖可以發現，VAE 的確就其他的 GAN 來說的確表現得稍微差了一些。


## 註釋


[^註1]: 
Goodfellow, Ian J., Pouget-Abadie, Jean, Mirza, Mehdi, Xu, Bing, Warde-Farley, David, Ozair, Sherjil, Courville, Aaron and Bengio, Yoshua. "*Generative Adversarial Networks.*" (2014): .

[^註2]: 
原始論文的 Loss Function 是如此設計，事實上也不一定要用這樣的方式，可以自行設計。在這一個式子中，比較直觀的想法是這樣 : 
我們希望真實圖片可以獲得高分，而假圖片則是獲得低分 
$\Longleftrightarrow \max(D(x^i))$ 並且 $\min(D(\tilde{x}^i))$
$\Longleftrightarrow\max(\log D(x^i))$ 並且 $\max(\log(D(1-\tilde{x}^i)))$
$\Longleftrightarrow\max\Big(\displaystyle{\frac{1}{m}\sum\limits_{i=1}^{m}\log D(x^i)+\frac{1}{m}\sum\limits_{i=1}^{m}\log(1-D(\tilde{x}^i))}\Big)$
$\Longleftrightarrow\max(\tilde{V})$

[^註3]:
[Unsupervised Learning -- AutoEncoder](/fnFXsGryQr-qP4q_-p5qwA) 

[^註4]:
[Unsupervised Learning -- Generation ( PixelRNN、Variational Auto-Encoder ( VAE ) )](/K0V0cQeYRfabsgKKKPwSEg)