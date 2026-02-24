---
title: "[ 論文 ] Gradient-Based Learning Applied to Document Recognition" 
date: 2019-07-28 22:16:58
categories:
- 論文 Paper
- 卷積神經網路 Convolutional Neural Network
image: https://i.imgur.com/BXEWBsY.png
mathjax: true
---

前言
---

機器學習或是深度學習，雖說發展已久，但在近幾年的發展突飛猛進，不管是 google , Facebook 還是 Amazon 的論文發表真的是一篇接著一篇，新技巧、新概念的開展著實讓人覺得不可思議，要了解整個發展以及新技術的應用，閱讀論文想必是最快速的方法。

在台灣 Pytorch Taiwan 以及許多讀書會也都開始有論文閱讀的部分都是正在進行式，我也是期待自己養成閱讀論文的習慣，因此便著手第一篇論文的閱讀。

第一篇論文，就從這 Convolutional Neural Network 的發源論文 *"Gradient-Based Learning Applied to Document Recognition"* 作為起頭吧 ! 這篇論文介紹了經典模型 LeNet-5 進行手寫數字辨識，許多人都將此論文視為 CNN 的起源，雖然說此論文年代稍為久遠，與現今的 CNN 模型稍有不同，但仍可藉由此論文體會、了解 CNN 的架構及精神。

<!-- more -->

此論文內容非常充實，足足有46頁之多，本篇文章主要著重在論文兩個核心部分，分別是 Introduction 以及 Convolutional Neural Network for Isolated Character Recognition。

論文摘要
---

## I.  Introduction

本論文最重要的是建構了一個更自動化的學習機器來進行手寫辨識，而減少人為手把手設計的辨識系統。

過去的模型，在特徵萃取的技巧上，都被限制在低維度空間上進行操作才能有比較好的表現，但近幾年來卻能夠提升高維度空間上的表現，且可以處理相對大量的資料集，主要的原因有三 : 

(1) 低價高效的計算單位容易取得，使得我們可以利用暴力的數值方法硬處理。
(2) 資料的取得更加容易，可以輕易地取得任何我們有興趣的資料。
(3) 最重要的一點是，機器學習技巧的發展，使我們可以在高維度上對大量的資料及進行處理。

### A. Learning from Data

若我們手中有一組訓練資料 $\left\{(Z^1,D^1),(Z^2,D^2),\cdots,(Z^P,D^P)\right\}$，我們可以使用機器學習方式對所有的輸入資料 $Z^p$ 學習出一套模式、一組權重 $W$ ，進行預測得到 $Y^p$

$$
Y^p=F(Z^p,W)
$$

進而我們可以針對預測值 $Y^p$ 與真實值 $D^p$ 計算誤差 $E^p$，也可以計算出整個訓練資料集的平均誤差 $E_{train}$

$$
E^p=D(D^p,Y^p)=D(D^p,F(Z^p,W))\\
E_{train}=\displaystyle{\frac{1}{P}\sum\limits_{p=1}^{P}E^p}
$$

之後我們在針對測試資料上的誤差期望值 $E_{test}$ 與 $E_{train}$ 進行比較已確認模型的準確度。而在一些研究工作上顯示出此二者的誤差會接近 $k(h/P)^{\alpha}$，$P$ 為訓練資料的數量， $h$ 是模型複雜度， $\alpha$ 則是介於 0.5 到 1 的常數值，而 $k$ 亦為常數值。
 
$$
E_{test}-E_{train}=k(h/P)^{\alpha}
$$

### B. Gradient-Based Learning

既然我們計算出對於任何一組權重 $W$ 資料所產生的誤差 $E(W)$ ，那我們便希望可以藉由調整權重 $W$，來讓誤差越來越小。而 Gradient-Based Learning 針對平滑可微分的連續函數提供了一個較為簡單的優化方式。

在 Gradient Descent 的方法中，我們不斷迭代調整 $W$，來得到更小的誤差值

$$
W_k=W_{k-1}-\epsilon\displaystyle{\frac{\partial E(W)}{\partial W}}
$$

而另一種更受歡迎的方法 : Stochastic Gradient Descent ( SGD ) ，改善 Gradient Descent 每一次都要丟全部的資料進去算梯度，SGD 利用隨機單筆資料的梯度拿來更新參數，就計算成本上來說會節省不少。

$$
W_k=W_{k-1}-\epsilon\displaystyle{\frac{\partial E^{p_k}(W)}{\partial W}}
$$

### C. Gradient Back-Propagation

在1950年代後期，Greadient-Based Learning 開始被人們使用，但也幾乎都僅限於線性系統。近年來幾件事情的發展使得 Gradient Descent 被廣泛的應用 : 

1. Loss function 的局部最小值並不會是實務上的主要問題 ( 李宏毅課程中也有講到，有些學者甚至認為高維度出現局部最小值的機率其實很小，我們使用 Gradient Descent 很可能找到的都是全局最小值。 )
2. Backpropagation (BP) algorithm 在多層非線性系統中可以有效率的計算梯度。
3. Backpropagation algorithm 也可以處理在多層神經網路中的複雜學習任務。

BP 的核心概念就是從輸出層往前推導到輸入層，有效率的計算梯度，雖然在 60 年代初期就有類似的概念出現，但還沒應用在機器學習的問題中。

### D. Learning in Real Handwriting Recognition Systems

( 略 )
手寫辨識中，最困難的並非辨識單獨的字母或數字，而是要對一整個句子或單字進行切割、分段。"Heuristic Over-segmentation" 便是處理這樣子問題的標準模式，它要能夠生成各種可能的切割方式，並從中找出最合適的組合。

而在此論文的後面幾個部分會提到如何實現 Segmentation ，但這不在本篇文章的討論範圍，因此不再贅述。

### E. Globally Trainable Systems

( 略 )
這一個部分，主要在談一個完整辨識系統的各個模組是如何運作、如何互相串接，這部分也非本篇文章的討論範圍，因此也先不詳細介紹。

## II. Convolutional Neural Network for Isolated Character Recognition

傳統的辨識系統，利用人工定義的 feature extractor 取得相關聯的資訊並且去除不相關的變數，配合著全連接層神經網路分類器來進行分類。但我們更在意的是，是否能夠讓機器自己學習出一套 feature extractor，如此以來我們便可以直接餵原始資料進全連接層的分類器，直接進行辨識。

然而這樣會產生一些問題 : 

1. 原始圖片的像素過大，在第一層的全連接層可能就會產生數以千萬計的參數(權重)，參數越多，我們就需要更多的訓練資料才能訓練出一個好的 model。當然，我們所需的計算成本也必須隨之增加。
2. 一個非結構化的圖片、語音的問題就在於它們並沒有結構上的不變性，輸入分類器的型態可能千變萬化。以手寫文字來說，即使我們可以做一些前處理 --- 將圖片尺寸標準化、將手寫字置於圖片中央 --- 但不管怎麼做總是無法做的夠完美，手寫文字的大小不會固定、也有文字傾斜...等問題。理論上，只要我們提供足夠多的 neurons 應該也可以訓練得夠好，但仍會有計算成本及訓練樣本不夠多的狀況。
3. 全連接層的架構忽略了圖片整體的拓樸性質，將輸入圖像轉化成為互相獨立的變數，但圖像本身具有很強的空間結構，某些像素跟周圍的像素間應該具有高度的相關性，這些相關聯的像素便組合成一個個局部特徵。而 CNN 便是掌握了這樣的概念進行有效的特徵萃取。

### A. Convolutional Network

Convolutional Network 架構包含了三個重要的概念 : 局部感受野 (Local receptive fields )、權重共享 ( Shared weights ) 以及 子(二次)採樣 ( Sub-sampling )



**Local receptive fields**

局部感受野的概念在早期的醫學、生物學上面就有許多的研究，在這樣的局部館策中，視覺神經元可以直接萃取出圖像上的特徵 ( 如 : 邊緣、端點、角落...)，而這些特徵可以組合成夠高層次的特徵。而這些特徵仍然保有其拓樸性質。

而這樣特徵萃取的概念應用在 Neural Network 上便是利用一組 identical weight vector ( 在 CNN 中稱之為 kernel 或是 filter ) 掃過整張圖片，然後輸出一張 feature map。這樣的過程對應到論文中的 LeNet-5 模型即為 卷積層 ( Convolution Layer )。一個完整的卷積層會有多個 kernel ( weight vector ) 組成，然後轉出成若干個 feature maps。

之所以稱為卷積，主要是因為 kernel 利用滑動的方式掃過整張圖的每一個位置，並且記錄下每一個位置的狀態然後對應到 feature map 的相對位置，這樣的操作與數學上的 「卷積」 相同。[^註1]



經過卷積層的操作後，一旦特徵被偵測出來後，這個特徵的精確位置便不是那麼重要，重要的反而是這個特徵與其他特徵的相對位置。

**Sub-sampling**

既然我們在意的是特徵間的相對位置，而不在意其精確的位置，最簡單的方式就是直接減少 feature map 的空間分辨率 ( Spatial Resolution )，做法便是將局部感受野進行平均化及子採樣，這過程便對應到 LeNet-5 的 Subsampling Layer ( 又稱作池化層 )。

在 LeNet-5 的 Subsampling Layer 中，利用 2X2 的 kernel，掃過 Convolution Layer 所產生的 feature maps，遍歷的方式與卷積層有所不同，每一個相鄰感受野均不重疊，將每一個局部區域數值取平均後再經過一個 sigmoid function 。經過 subsampling 過後的 feature maps 長寬均會減少一半。


**Shared weights**

由於卷積的操作特性，我們不難發現，每一個 kernel 滑過整張圖的過程中，參數都是固定的，這樣權重共享的概念讓我們在計算上減少非常多的負擔。而且也會使 test error 更接近 train error。


### B. LeNet-5

這個部分主要再詳細介紹整個 LeNet-5 的各層架構及運作。$C$ 表示卷積層，$S$ 表示池化層，而 $F$ 則是全連接層，下標數字則是標示位於第幾層。由此可見，LeNet-5 是一個有六層主結構的辨識系統 ( 不含輸入輸出層 )。

![](https://i.imgur.com/BXEWBsY.png)

以下我們依照各層分別介紹: 

#### Input

LeNet-5的輸入格式為 $32\times 32$ 的手寫圖像。

#### $C_1$ (卷積層)

第一層是有 6 個 $5\times 5$ 的 kernel 的卷積層，每一個kernel經過卷積會輸出一張 feature map，因此在這一層會有六張 $28\times 28$ 的 feature maps。
( 以現在的 CNN 來說，這是 stride=1 且沒有 padding 的卷積層 )


<img width=500 src="https://i.imgur.com/WDxti0L.png" >
( 圖片來源 : [LeNet-5详解与实现](https://www.charleychai.com/blogs/2018/ai/NN/lenet.html) )

我們也可以輕易地計算，第一層一共有 $(25\times 6)+(1\times 6)=156$ 個權重，
及 $(25\times 28^2\times 6)+(28^2\times 6)=122304$ 個連結數。

#### $S_2$ (池化層)

第二層是有 $2\times 2$ kernel 的池化層，先經過平均池化後給予一個權重及 偏置 ( bias ) 再丟進 sigmoid function 形成 feature maps。feature maps的尺寸經過池化後會減少一半 ( stride=2 )，亦即此層生成的 feature maps 尺寸會是 $14\times 14$。



<img width=500 src="https://i.imgur.com/U1Qy4xM.png" >
( 圖片來源 : [LeNet-5详解与实现](https://www.charleychai.com/blogs/2018/ai/NN/lenet.html) )

這一層一共有 $(1+1)\times 6=12$個權重，且有 $4\times 14^2\times 6+14^2\times 6=5880$ 個連結數。

#### $C_3$ (卷積層)

此層較為特別，作者採用的方式是將數個 $S_2$ 的 feature maps 一起做卷積，這樣會產生一共 16 個 feature maps，且每一個 feature map 大小為 $10\times 10$。( 連結方式如下圖右 )

![](https://i.imgur.com/kPUkWKL.png =250x)  ![](https://i.imgur.com/7tGTZHB.png =400x)

為何不採用全連接的方式 ? 首先，這樣可以減少參數。其次，$C_3$ 每一個輸入的 feature map 都是 $C_1$ 的 kernel 針對輸入圖像所萃取出來的特徵圖，利用這樣不對稱的連結方式，反而可以組合成更高層級的特徵。 

這一層中我們有 $6\times(25\times 3+1)+9\times(4\times 25+1)+1\times(6\times 25+1)=1516$ 個參數及 $151600$ 個連結。
 
#### $S_4$ (池化層)

操作方式與 $S_2$ 相同，經過 $2\times 2$ 的 kernel 池化過後，feature maps 的大小是 $5\times 5$。

此層具有 $(1+1)\times 16=32$個參數，及 $(4\times 25\times 16)+(25\times 16)=2000$ 個連結。

#### $C_5$ (卷積層)

這裡跟 $C_3$ 卷積層不同的地方在於這邊是採全連接的方式產出一共120個 feature maps，由 $5\times 5$ 的 kernel 一次對所有 ( 16個 ) $S_4$ 的 feature maps 卷積過後的 feature map 大小會變成 $1\times 1$。

這一層會有 $(5\times 5\times 16+1)\times 120=48120$ 個參數。

#### $F_6$ (全連接層)

這一層是一個簡單的全連接層，共有 84 個神經元，與上一層的 120 個 feature maps 進行全連接。( 為何取 84 ? 我們在 Output 層會說明。 )

如同一般的神經網路，這 84 個神經元都是 $C_5$ 層中 120 維向量與權重向量內積後，經過 activation function 轉換後再加上偏置而得。


<img width=500 src="https://i.imgur.com/XJ5yPms.jpg" >




因此這邊一共會有 $(120\times 84)+(1\times 84)=10164$ 個參數。

這裡使用的 activation function 是 $A\cdot tanh(Sa)$，其中 $S$ 是原點的斜率，則 $A=1.7159$。

#### Output

最後的輸出層，使用 Euclidean Radial Basis Function ( RBF ) 計算出每一個類別 (此例就是 0-9) 的數值

$$
y_i=\sum\limits_{j}(x_j-w_{ij})^2
$$

這個式子以及 $y_i$ 代表的意義是什麼呢 ?

在輸出層以前，我們輸入一張手寫圖片，最後會得到一個 84 維的向量，利用這 84 維向量跟各個類別的權重向量計算這兩者的歐式距離。

為什麼當初在 $F_6$ 層的輸出會取 84 個神經元 ?
因為我們要針對每一個類別給予一個標準權重值，好跟 $F_6$ 層的輸出來計算距離，那麼這個標準權重值就是從 ACSII 而來

![](https://i.imgur.com/zG8PMM3.png)

這裡面每一個字母 / 數字 / 符號 都是 $7\times 12$ 的大小，每一個字符我們都可以化成一組 84 維的標準權重，其中每一維的值不是 +1 ，就是 -1。這樣的設計可以避免 $O,o,0$ 這種容易混淆的字符造成判別錯誤。

我們可以將各層視覺化出來，或許更能了解每一層的作用

![](https://i.imgur.com/o49ic8B.png)


### C. Loss Function

一般來說我們的 Loss Function 會採用 Maximum Likelihood Estimation (MLE)，但在本論文中其實 MLE 等價於 Minimum MSE ( Mean Square Error )

$$
E(W)=\displaystyle{\frac{1}{P}}\sum\limits_{p=1}^{P}y_{D_{p}}(Z^p,W)
$$

$y_{D_{p}}$ 指的就是第 $p$ 筆資料輸入後所得的「正確類別」 RBF 值，所以訓練資料的 error 就是所有訓練資料所得「正確類別」 RBF 值之平均。( 從 RBF 的型態來看其實就是 mean square error )

但這樣的 Loss Function 有一些缺點 : 

1. RBF 的參數不可拿來訓練。倘若 RBF 的權重也可拿來訓練，會導致 RBF 權重會因不斷調整而趨於相等，而 $F_6$ 層的結果也會趨近於 RBF 權重，最後 RBF輸出均為 0。亦即不管輸入什麼資料，最後訓練出來的 RBF 輸出都會是 0 。

2. 缺乏考慮各類別間的競爭關係。這樣的競爭關係我們可以用 Maximum a posteriori ( MAP , 最大後驗機率法 ) 來表示。假定一個樣本，我們希望他的預測可以接近真實的 label ，那我們從機率的角度來看就是要試著最大化它的後驗機率[^註2]。( 或是最小化正確類別的機率值取 $\log$ )。因此從這個方向我們可以修改一下我們的 Loss Function


$$
E(W)=\displaystyle{\frac{1}{P}}\sum\limits_{p=1}^{P}y_{D_{p}}(Z^p,W)+\log(e^{-j}+\sum\limits_{i}e^{-y_i(Z^p,W)})
$$

從 $\sum\limits_{i}e^{-y_i(Z^p,W)}$ 這一項不難發現，若錯誤類別的 RBF 值越小，進行懲罰的就越多。如此一來，整個模型便能同時考量正確類別的誤差及其他類別的交互影響。

後記
---

這是一偏較早期的論文，所使用的方法其實在現在的 CNN 架構中很多都不被使用，舉例來說 $C_3$ 層使用的特殊特徵配對方式，由於現在的硬體效能都提升很多，現今已不再使用這樣的方式進行配對。各層所使用的 Sigmoid function 現在也大多被 ReLU 所取代。雖然如此，這篇論文的價值仍然不容被忽視，仍然可以提供完整圖像辨識的架構給讀者。

此篇論文的內容非常多，我也無法一次全部看完，目前就先以最重要的部分進行閱讀，待日後有時間再來將後面的部分補完。

參考資料
---

1. LeCun, Y., Bottou, L., Bengio, Y., & Haffner, P. (1998). Gradient-based learning applied to document recognition. Proceedings of the IEEE, 86(11), 2278-2323. DOI: 10.1109/5.726791 (論文[下載](http://yann.lecun.com/exdb/publis/pdf/lecun-01a.pdf))
2. [LeNet-5详解与实现](https://www.charleychai.com/blogs/2018/ai/NN/lenet.html)
3. [PyTorch Taipei 2018 week1: LeNet-5](https://mattwang44.github.io/en/articles/PyTorchTP-LeNet/)
4. 論文筆記 --- [Gradient-Based Learning Applied to Document Recognition](https://hackmd.io/@K3QB5C-YSsyBr6dBBo6IkQ/rka5m35NX)


註釋
---

[^註1]: 
在CNN上卷積操作其實就是 Slide + Inner Product，利用滑動的方式遍歷所有位置，並記錄下 kernel 與相對位置的內積值於 feature map 的相對位置上。


[^註2]:
條件機率 : 在事件 $y$ 發生的前提下，事件 $x$ 發生的機率 $\boldsymbol{P}(x\mid y)$
後驗機率 : 事件 $x$ 事件發生後的反向條件機率 $\boldsymbol{P}(y\mid x)=\displaystyle{\frac{\boldsymbol{P}(x\mid y)\cdot\boldsymbol{P}(y)}{\boldsymbol{P}(x)}}$