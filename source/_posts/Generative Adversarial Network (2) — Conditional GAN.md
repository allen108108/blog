---
title: "Generative Adversarial Network (2) --- Conditional GAN" 
date: 2019-10-29 08:43:43
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning and having it Deep and Structured
image: https://i.imgur.com/RrdITkw.png
mathjax: true
---

Text to Image
---

在某一些情境下，我們會希望有一個 model 可以吃進一個字或是一串句子，然後生成一張相對應的圖片。這樣的 Text to image 的目標，我們其實有機會可以用 NN 來做一個 Superviced Learning 來達成。

<!-- more -->

![](https://i.imgur.com/EESwg8e.jpg)

然而這裡其實有一個問題，利用這種方式機器所生成的圖往往都是模糊的，其原因是因為通常要符合輸入的圖片可能有很多種，那麼在不確定要用哪一張圖做為生成的對象持，機器會傾向於生成一張圖片是這些可能圖片的「平均」。

![](https://i.imgur.com/EjKXS1m.jpg)

想要解決這樣的問題，這邊引進了一種 GAN 的變體 : Conditional GAN

### Conditional GAN


CGAN 與一般的 Normal GAN 不同的地方在於餵給 Generator 與 Discriminator 的輸入不同。

![](https://i.imgur.com/AgjCU1e.png)

* Generator 的輸入除了本來就有的 $z$ 之外，還多了一個 constraing $c$。Constrain 的作用便是要讓電腦知道我們想要一個什麼樣子的圖。
* 而 Discrimiantor 的輸入也不只有 generator 的輸出 $x$，一樣要加進 Constrain $c$。如果我們只有輸入 $x$ 給 discriminator，那麼 generator 就會學到只要生成一個足夠好的照片即可，並不用管 constrain 是什麼。

Generator 的 constrain $c$ 不難理解，但為什麼在 Discriminator 上也要加上一個 constrain 呢 ?

可以試想一下，如果沒有 Discriminator 的 $c$，那麼整個 machine 只要讓 Generator 生成一張跟 constrain 完全沒有關係但是足夠好的圖就可以騙過 Discriminator 了。所以，在 Discriminator 上不僅要檢查這張照片夠不夠好，也同時要檢查是否符合 constrain。

值得注意的是，在整個 Discriminator 訓練上，當然要讓它除了可以將 constrain 跟正確的圖像正確判定為 True



![](https://i.imgur.com/qmaDWoT.png)



然後將不符合 constrain 的圖像判定為 False 



![](https://i.imgur.com/CrG8NuV.png)



除此之外，還要讓正確圖像配上隨機的文字判定為 False



![](https://i.imgur.com/CfahGx2.png)



我們可以將上述的整個過程寫成演算法如下 ( 其實就是調整一下基本的 GAN 演算法 )

* 初始化 $G$ 與 $D$ 的參數 $\theta_g$ 與 $\theta_d$
* 循環迭代進行
    
    * Updeate $D$
        * 從 database 中隨機抽取 $m$ 個真實樣本 $\{(c^1, x^1),(c^2,x^2),\cdots,(c^m,x^m)\}$
        * 從我們給定的分布中隨機抽取 $m$ 個向量 $\{z^1,z^2,\cdots,z^m\}$
        * 藉由 $\tilde{x}^i=G(c^i,z^i)$ 生成 $\{\tilde{x}^1,\tilde{x}^2,\cdots,\tilde{x}^m\}$
        * 從 database 中隨機抽取 $m$ 個真實物體 $\{\hat{x}^1,\hat{x}^2,\cdots,\hat{x}^m\}$
        * Loss Function $\tilde{V}=\displaystyle{\frac{1}{m}\sum\limits_{i=1}^{m}\log D(c^i,x^i)+\frac{1}{m}\sum\limits_{i=1}^{m}\log\big(1-D(c^i,\hat{x}^i)\big)}$
        * 藉由最大化 $\tilde{V}$ ( Gradient Ascent ) 來更新權重 $\theta_d$ : 
$$
\theta_d = \theta_d + \eta\cdot\nabla\tilde{V}(\theta_d)
$$
    * Update $G$
        * 從我們給定的分布中隨機抽取 $m$ 個向量 $\{z'^1,z'^2,\cdots,z'^m\}$
        * 從 database 中隨機抽取 $m$ 個 constain $\{c^1,c^2,\cdots,c^m\}$
        * Loss Function $\tilde{V}=\displaystyle{\frac{1}{m}\sum\limits_{i=1}^{m}\log D(G(c^i,z'^i))}$
        * 藉由最小化 $\tilde{V}$ ( Gradient descent ) 來更新權重 $\theta_g$ : 
$$
\theta_g = \theta_g - \eta\cdot\nabla\tilde{V}(\theta_g)
$$


目前絕大多數的 paper 設計的 Discriminator 架構如下
![](https://i.imgur.com/7TmDMRb.jpg)

這樣的架構雖然很直覺，但是卻不太合邏輯，當我們將 object 跟 constrain 合在一起訓練，機器給的高低分數很難確定是來自什麼原因，可能是 object 圖像並不夠好，也可能是跟 constrain 不搭配導致。

![](https://i.imgur.com/F09phne.jpg)

少數的 paper 將架構設計成這樣，看起來比較符合邏輯，將 object 品質與 constrain 和 object 符合的程度脫鉤處理，這樣可以確定究竟是哪一個因素導致高/低分，也可以藉此調整其中一個網路即可。


### Stack GAN

這是另一個 GAN 的變體，利用兩組 Generator-Discriminator 來輸出大尺寸圖像。

![](https://i.imgur.com/MtnuoCr.png)


Image to Image
---

藉由上面的討論，我們可以將 Constrain 從文字改為圖像，來達到 Image to Image 的目標。

![](https://i.imgur.com/Qbsk9IR.png)

* 上圖左二 : 單純利用 NN 輸出跟真實圖片的 loss 最小化而得，這樣的圖通常會很模糊，原因在於最後的輸出會採各種可能的平均值。
* 上圖右二 : 利用 Conditional GAN 來做會得到清晰的圖片，但仍然可能會有一些奇怪的輸出，例如圖中左上角類似煙囪的奇怪圖形。
* 上圖右一 : 同樣利用 GAN，但在 Generator 輸出多加了一個部分 : 最小化其與真實圖片的 loss。這樣的方式可以視為上面兩種方式的結合，可以確保輸出的圖片夠清晰，並且不會有一些奇怪的產出。

![](https://i.imgur.com/Ha80tAh.jpg)


### Patch GAN / Pixel GAN

在圖片尺寸稍大的狀況下，如果用整張圖片丟進 Discriminator 來做檢查很容易導致訓練時間過長、Overfitting 或是 performance 不好等狀況，因此 Patch GAN 便是只採取整張圖片的 patch 做檢查。

這邊我想補充一點，單就課程內容來看，好像很容易會誤以為我們只要針對某一個 patch 丟給 Discriminator 就好，但是我查了一下似乎並非如此。

<img width=500 src="https://i.imgur.com/nfEjU6M.png" >


在 "[关于PatchGAN的理解](https://blog.csdn.net/xiaoxifei/article/details/86506955)" 一文中有談到， Patch GAN 事實上是輸出一個 $N\times N$ 矩陣，而每一個矩陣內元素是代表著一個大的感受野，亦即代表一個圖像的 patch。這樣的實現其實也就是藉由 CNN 來完成。

Patch GAN 的 Discriminator 輸出尺寸就是一張 CNN 的 feature map 尺寸，然後內部元素對應的就是對於此 feature map 的相似度判定。


Image to Image 的模式，可以適用於任何以圖像表示的任務上，例如 Speech Enhancement ( 語音增強 ) 上。目前的語音任務大多利用 spectrum 來進行分析，而 spectrum 也跟一般圖像任務一樣具有結構性，使用 conditional GAN 當然沒有什麼問題。

![](https://i.imgur.com/CSVORtT.png)

當然對於影片( 連續 frame )來說亦是如此，利用 conditional GAN 可以預測連續 frame 後的下一個 frame 是什麼。

![](https://i.imgur.com/bZLUy5p.png)

拿來打電動也是可以的啦 

![](https://i.imgur.com/uBrQRgZ.gif)
![](https://i.imgur.com/1T8FYwy.gif)

不過還是會有點 bug，譬如說突然消失或是變出分身之類的，主要是因為遇到分叉路時，小精靈就會有走不同道路的可能性，於是下一步就可能會分身走不同方向的狀況出現。