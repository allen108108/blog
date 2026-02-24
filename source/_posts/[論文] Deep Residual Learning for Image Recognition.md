---
title: "[論文] Deep Residual Learning for Image Recognition" 
date: 2019-10-29 08:25:14
categories:
- 論文 Paper
- 卷積神經網路 Convolutional Neural Network
image: https://i.imgur.com/sPkItgn.jpg 
mathjax: true
---


概要 Abstrct
---

當深度逐漸增加，神經網路的訓練就會越來越困難。在這篇論文中，作者們提出了一個殘差學習 (residual Learning) 框架來使極深層的網路結構訓練變得更簡單。原本神經網路中的層 (layer) 目的在學習出一個未知的函數，研究團隊則根據輸入來將其目的改成學習一個殘差函數。論文中許多的實驗證明了無論在 ImageNet,CIFAR-10 或是 COCO 資料集上，這樣的方式可以使網路更容易被優化，而且可以得到更好的表現。

<!-- more -->

簡介 Introduction
---

Deep CNN 引領了圖像分類一系列的突破，深層網路整合了低/中/高級特徵進入一個 end2end 分類器，而藉由層堆疊可以使這些層級的特徵更加豐富、多元。近來的許多研究也指出「深度」是非常重要的因素，許多優異的研究成果都建築在極深層的網路結構下。

既然深度如此重要，那麼便沿生出另外一個問題 : 「網路的學習是否跟增加深度一樣的簡單 ?」，從最常見的梯度消失/爆炸情狀況來看顯然並非如此。要解決這樣的問題就必須從層的 Normalized Initialization 以及 Intermediate Normalization 來做，而這些方法可以使較深層的網路結構在隨機梯度下降 ( Stochastic Gradient Descent, SGD ) 及反向傳播法 ( Backpropagation, BP )下收斂。

但是當深層網路開始收斂時，退化 (degradation) 問題就會開始出現。

![](https://i.imgur.com/uiZKSUo.png)

當深度逐漸增加，準確度卻開始出現飽和，而且退化迅速。這樣的退化問題並非來自於 overfitting，而是因為深度增加連帶著使得 training error 增加所致。

退化問題指出了並不是所有的網路結構都可以容易被優化的。我們假設手上有一個淺層網路結構。利用這個淺層結構再多加幾層恆等映射層 (identity mapping layers) 形成一個更深層的網路結構，我們可以確定，這一個深層結構的表現其實是和淺層結構的表現一樣的，也不會多增加 training error。但經由實驗發現，深層網路經過訓練後，根本無法逼近于恆等映射。

這裡可能有點難理解，我拿上圖左的例子來說好了。

20層的網路結構是黃綠色的那條線，照理來說，56層的網路結構只要後面的36層通通是恆等映射就可以讓整個網路表現跟 20 層的表現一樣好，但是實驗卻發現，經過訓練我們根本無法達到這樣的狀態，也就是說，要讓一個網路經由學習逼近成恆等映射是幾乎不太可能的事情。( 可能是因為深層網路參數量太大導致難以優化到極致 )

我們都知道可以將一個網路結構視為一個函數 $\mathcal{H}(\mathbb{x})$ ( 忽略 activation function 的情況下 )

從上面的討論來看，希望訓練後 $\mathcal{H}(\mathbb{x})\rightarrow \mathbb{x}$ 這樣子的逼近似乎很困難，那試試看 $\mathcal{H}(\mathbb{x})-\mathbb{x}\rightarrow 0$ 如何 ? 我們將網路結構從 $\mathcal{H}(\mathbb{x})$ 轉變成 $\mathcal{F}(\mathbb{x})=\mathcal{H}(\mathbb{x})-\mathbb{x}$ 如何 ?

實驗結果，這樣的逼近變得簡單，應用在數百層的網路結構中也不會有上述退化的狀況。


我們將這樣的型態 $\mathcal{F}(\mathbb{x})=\mathcal{H}(\mathbb{x})-\mathbb{x}$ 稱為 residual mapping，經過訓練後，我們可以發現這樣的 residual mapping　變得更容易優化，極端的狀況，可以將 residual 通通趨近到 0，也就是達到我們一直想要的目標 「將整個網路結構趨近於 identity mapping」。

**[補充]**

這裡我想補充兩個部分是論文中沒有提到的，residual mapping 其實衍生兩個問題 : 

1. 為什麼 $\mathcal{F}(\mathbb{x})=\mathcal{H}(\mathbb{x})-\mathbb{x}$ 會比較容易優化 ?
2. 利用這樣的 residual mapping 我們可以讓網路結構趨近於恆等映射，那不也只是代表整體網路的 performance 可以表現得跟淺層網路 「一樣好」，為何實驗結果可以更優於 「淺層網路」 ?

關於第一個問題，有人提出的想法是 $\mathcal{H}(\mathbb{x})\rightarrow \mathbb{x}$ 這樣的優化，箭頭的兩端都含有未知數，也就是同時兩端都在更新，所以這樣的狀態下要達到平衡是困難的，相反的，$\mathcal{F}(\mathbb{x})=\mathcal{H}(\mathbb{x})-\mathbb{x}\rightarrow 0$ 是將整個函數經過不斷調整至一個 constant，相對來說較容易達到平衡狀態。

而第二個問題，可以利用下圖來解釋 : 

![](https://i.imgur.com/Xo67I5n.png)

事實上整個 shortcut connections 擴大了解空間 ( solution space )，讓更優解變的可能，因此，整個模型的表現便不限制只會趨近於淺層網路，更可能找到更好的解。

Residual mapping 我們也可以藉由網路結構的變更來實現


<img width=500 src="https://i.imgur.com/a1eMhwP.png" >



旁邊這一條稱之為 "Identity Shortcut connection" ，作用在於讓信息可以跨過數層往下傳遞，這樣的設計不須額外的參數也不會增加計算複雜度，且可以使用一般的 Library 利用 SGD 來進行 BP，而不需修改 solver。

論文中利用 Residual Network 針對 ImageNet、CIFAR-10、COCO dataset 上進行實驗都能取得相當好的表現，尤其在 ImageNet 上深度達到 152 層的 Residual Net 不僅表現上優於 VGG Net，整個複雜度也較 VGG Net 還要低，這些都顯示了 Residual Learning 在各種任務上的通用性。


相關工作 Related Work
---

### Residual Representations

VLAD ( Vector of Locally Aggregated Descriptors )，是一種圖像的特徵表示法[^註1]，而 Fisher Vector 則可視為 VLAD 的機率版本，這兩種方式是圖像檢索及分類非常有用的表示法。

而偏微分方程的求解是低層次電腦視覺及電腦圖像上經常使用的方法，其中多重網格法 ( Multigrid method ) 將多尺度任務切成子問題，每一個子問題都可以視為求解大小尺度間的殘差解 ( residual solution ) 。

另一個取代多重網格法的方法是層級基底預處理 ( Hierarchical Basis Preconditioning )，這樣的方式依賴兩個尺度間殘差向量的變量。

不管是上述哪一個方法，都被證明其收斂速度比其他不依賴殘差性質的方法來的快，且這樣子的預處理都可以簡化優化過程。


### Shortcut Connections

Shortcut Connection 這樣的方法已經經過了很長一段時間的實作及理論研究過程。早期的多層感知器 ( Multilayer perceptrons, MLPs ) 在輸入、輸出中間接上線性層 ; 一些研究中於中間層接上輔助分類器 ( auxiliary classifier ) 來避免梯度消失/爆炸的狀況 ; 也有一些研究利用了 Shortcut Connection 來集中層響應 ( layer responces )、梯度以及誤差傳導 ; 在知名的 "Inception" 架構中也是由各分支、shortcut 分支所組成。

在論文 " Highway Network " 中將 Shortcut Connection 與 gate 機制結合。與 Residual Net 不同的地方在於，Highway Net 的 gate 會由輸入控制，也因而會產生參數，但 Residual Net 完全由 identity shortcut connections 來進行跨層的信息傳輸，並不會產生任何參數，不像 Highway Net，這樣的 shortcut connections 永遠不會被關閉。

因此在深層網路結構下， Residual Net 可以有效提升模型的表現，而 Highwat Net 並沒有辦法在極深層網路上有夠好的表現。


深度殘差學習 Deep Residual Learning
---
### Residual Learning

如果把 $\mathbb{x}$ 視為輸入層，我們可以將數層 ( 局部結構，不一定是整個網路結構 )視作是一個函數 $\mathcal{H}(\mathbb{x})$，這樣的假設在前面我們已經有提過 ( 但其實在論文 "*On the number of linear regions of deep neural networks.*" 中有提到，這樣的假設仍然是一個 open question )，那也等同於這些層我們可以用另外一個函數來代替 $\mathcal{F}(\mathbb{x})=\mathcal{H}(\mathbb{x})-\mathbb{x}$，則原來的函數則為


$$
\mathcal{H}(\mathbb{x})=\mathcal{F}(\mathbb{x})+\mathbb{x}
$$

![](https://i.imgur.com/TNwzuNP.png)

從數學式來看這兩個形式可以說是等價的，但是就機器學習的角度來看，學習過程可能大不相同。

這樣的重構是因為違反直覺的退化問題發生而激發的想法。前面我們提過，如果多餘的層通通是恆等映射，那麼整個深層模型的表現應該不會比淺層模型還要差，然而退化問題指出事實上這些多出來的非線性層根本很難去逼近恆等映射。而換個方式利用殘差學習，反而容易讓殘差去趨近於 0，讓整個函數去逼近恆等映射。

![](https://i.imgur.com/600joXK.png)

上圖是利用 CIFAR-10 針對各層做 Batch Nprmalization 之後的輸出值，可以觀察到 ResNet 在各層的標準差 ( Standard deviations, std ) 都比一般網路 ( plain Net ) 來的低，這說明了各層的輸出都更集中在 0 附近，也就是最優解會更接近恆等映射而非零映射。


### Identity Mapping by Shortcuts

在論文中，作者們對數個層的堆疊做殘差學習，這些堆疊的層我們可以視為一個一個的 build block，可用一個函數表示為

$$
\mathbb{y}=\mathcal{H}(\mathbb{x})=\mathcal{F}(\mathbb{x}, \{W_i\})+\mathbb{x}
$$

從論文一開始兩層的 build block 來看，可以將各個輸出計算出來 (忽略 bias以簡化運算)

![](https://i.imgur.com/7fDJQwO.png)

其中 $\sigma$ 代表的是 ReLU 函數，最後透過一個 element-wise 的加法，再通過一個 ReLU 函數

$$
\mathbb{y}=W_2\cdot\sigma(W_1\mathbb{x})+\mathbb{x}\\
\sigma(\mathbb{y})=\sigma(W_2\cdot\sigma(W_1\mathbb{x})+\mathbb{x})
$$

從 shortcut connection 可以看出來，我們並沒有增加任何的參數，當然也不會增加任何的計算複雜度，這樣的結構在實踐上是非常吸引人的，也因為參數量沒有增加，我們可以公平的與相同參數的網路結構做比較。

然而，shortcut connection 有一個問題，就是在輸入與輸出向量必須具有相同的維度才能進行 element-wise 的加法運算，為了解決這樣的問題，論文內的解決方法是在 shortcut connection 上乘上一個投影矩陣 $W_s$ 來對 $\mathbb{x}$ 做維度的調整。

$$
\mathbb{y}=\mathcal{F}(\mathbb{x}, \{W_i\})+W_s\mathbb{x}
$$

當然，我們可以在一開始所有的 shortcut connection 上都呈上一個 $W_s$，但是基本上單純的恆等映射已經可以處理退化問題，實在無須多增加參數，因此僅在調整維度上才會使用 $W_s$。

我們要對多少層進行堆疊並加上 shortcut connection 均可，$\mathcal{F}$ 是很彈性的，但是在實驗上可以看出，如果只對單一層做 shortcut connection，Residual Net 並沒有特別好的表現。

雖然上面的舉例都是使用全連接層，但事實上這可以適用在卷積層上也沒有問題。$\mathcal{F}(\mathbb{x}, W_i)$ 可以用來表徵多個卷積層，而 element-wise 加法則是針對兩個 feature maps 來進行。


### Network Architectures

研究團隊針對各種一般網路以及殘差網路架構進行實驗，觀察到一些一致性的現象，以下我們描述兩種應用在 ImageNet 的網路架構以供後續討論

#### 一般網路 Plain Network
論文中的一般普通的網路架構來自於 VGG Net。

* 具有 $3\times 3$ filters 的卷積層，且 stride 為 2
* feature map 尺寸相同者，filter 數量均相同
* feature map 尺寸減半者，filter 數量加倍以確保時間複雜度不變。
* 最後接上一個全局平均池化層 (Global Average Pooling Layer) 以及一個 1000 個類別，以 Softmax 為 activation function 的全連接層分類器。
* 含有參數的層數為34。

#### 殘差網路 Residual Network

基於上述一般網路架構，我們加入 Shortcut connections，轉換成殘差網路架構。

* 輸入與輸出尺寸相同時，直接使用 Shortcut connections (下圖實線 Shortcut connection)
* 輸出尺寸增加時，我們利用兩種方式來解決維度不同的問題，並後續進行實驗比較 : 
    (1)增加的維度利用 zero-padding 來處理
    (2)利用投影矩陣來進行維度調整 
    
![](https://i.imgur.com/X7Oj5Bv.png)   

### 具體實現 Implementation

此論文遵循著 " *Imagenet Classification with Deep Convolutional Neural Networks* " 以及 " *Very deep convolutional networks for large-scale image recognition.* " 兩篇論文來進行網路架構的實現。

* 從 $[256,480]$ 中隨機抽取進行尺寸增強
* 針對圖像或其翻轉後之圖像隨機選取 $224\times 224$ 區域進行資料增強，並同時將每一個 pixel 減去所有 pixel 之平均值。
* 進行標準顏色增強。
* Activation function 之前進行 Batch Normalization
* 從 zero-mean 且 high variance 的分佈中對權重進行初始化，以確保神經元能夠有正 input 使學習可以繼續。
* Batch size = 256
* Learning Rate = 0.1
* Weight Decay 正則化 = 0.0001
* momentum = 0.9
* 不使用 dropout

在測試資料上，為了可以進行比較，採取了

* 標準 10-crop 測試
* 全卷積型態
* 針對 $\{224,256,386,480,640\}$ 這些尺寸( 短邊 )上計算平均分數

實驗 Experiment
---

( 依照慣例省略，有興趣者請直接參閱論文 )



參考資料
---

1. [resnet（残差网络）的F（x）究竟长什么样子？](https://www.zhihu.com/question/53224378)
2. [【論文筆記】《Aggregating local descriptors into a compact image represenatation》](http://mrulafi.blogspot.com/2016/03/aggregating-local-descriptors-into.html)
3. [Image Retrieval之VLAD](https://zhuanlan.zhihu.com/p/36022126)


註釋
---
[^註1]: 
VLAD 的概念是 2015年的論文 "*Aggregating local descriptors into a compact image representation*" 中提出的概念，結合了 BOF (Bag of features) 與 Fisher Kernel 的優點，利用 k-means 對特徵子 (descriptor) 進行 clustering，並找出中心作為 visual word。VLAD 不僅利用 visual word 來代替特徵子，而且同時記錄下每個特徵子與中心的距離，一方面達到圖像特徵表示方式的優化，另一方面也可以使圖像特徵的降維及檢索的優化變得更容易。