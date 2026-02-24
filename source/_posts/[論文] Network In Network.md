---
title: "[論文] Network In Network" 
date: 2019-10-07 02:55:47
categories:
- 論文 Paper
- 卷積神經網路 Convolutional Neural Network
image: https://i.imgur.com/ivySLGK.png
mathjax: true
---

概述
---

一般的 Convolutional Neural Network ( CNN ) 模型包含了卷積層以及池化層。卷積層中藉由 filters ( kernels ) 對輸入層的每一個局部域做線性卷積運算後再通過非線性的 activation function 輸出 feature maps。而池化層利用 subsampling 在不影響特徵的前提下縮減參數數量，並且提高感受野 ( receptive field )。
<!-- more -->

然而作者認為 CNN 是一個廣義線性模型 ( Generalized Linear Model,GLM )，在 GLM 中萃取出來的特徵都是抽象化程度低的特徵。所謂的抽象化程度[^註1]意即某一個概念，在進行特徵轉換的過程當中，能夠保有多少的不變性。抽象化程度越高的特徵，在一連串轉換下便能保有更高的不變性，也越不受外界干擾而產生影響。這樣也表示 CNN 是以資料轉換到特徵空間中是線性可分的條件做為前提，但在現實狀況下，轉換後的資料通常都是以 非線性流形( nonlinear manifold )存在，因此作者希望將 CNN 中 GLM 的部分利用非線性結構來取代。



Network in Network ( NIN ) 便是在這樣的概念發想中產生，利用 MLPConv Layer 來取代傳統 CNN 中的 Conv Layer。在 MLPConv Layer 中仍保留傳統 Conv Layer 中共享權重的特性對於輸入圖像進行滑窗運算。

![](https://i.imgur.com/smNdXaV.png)

而 NIN 便是由 MLPConv Layers 堆疊組合而成的模型。

在傳統 CNN 中，最後會有一個 Fully Connected Layer 作為分類器，而在 NIN　捨棄了這樣的設計，選擇以 Global Average Pooling Layer 在最後一層 MLPConv Layer 輸出的 feature maps 映射到各個分類上，再通過 softmax 計算分類結果。

若我們今天要做 C 個分類，便在最後一層 MLPConv Layer 輸出 C 個 Feature maps，而這些 Feature maps 會映射到 C 維向量的每一個維度上，最後再由 softmax 做分類。


![](https://i.imgur.com/ivySLGK.png)

Global Average Pooling Layer 解決了一直以來 FC Layer 猶如黑箱般解釋性低的狀況，Global Average Pooling Layer 為這樣的分類過程賦予意義。除此之外， FC Layer 的 overfitting 狀況也可以因此避免。


卷積神經網路 Convolutional Neural Network
---

當資料經過特徵轉換後是線性可分的，那麼傳統 CNN 的線性卷積運算應該足以對特徵進行足夠的抽象化萃取。

然而，現實情況下，資料轉換到特徵空間中都是高度非線性函數，按理來說，只要我們的 filters 足夠多，仍足以對任意非線性函數進行近似逼近，但此舉亦會造成整個模型的參數量暴增，因此，若能在局部特徵萃取時便能做到近似非線性的逼近，便能避免使用大量 filters 來逼近。

作者注意到 Maxout Network [^註2]利用可學習性的 maximum activation function 來降低 feature maps 的數量，並且這樣的方式可以逼近任意的凸函數，似乎可以改善傳統 CNN 無法有效非線性函數的缺點。

雖然 Maxout Network 看起來很強大，但卻只能逼近「凸函數」，這樣的條件在一般狀況下並不一定會成立，因此作者在此論文中提出了 NIN 這樣的網路架構來解決 CNN 無法妥善處理的問題。




Network in Network
---




### 多層感知器卷積層 MLP Convolutional Layers

由於我們對於隱含特徵並沒有任何先驗資訊，因此作者會希望利用函數來逼近進而藉此來進行特徵萃取。一般來說，用來逼近的方式有 MLP ( multilayer perceptron ) 與 RBF Network ( radio basis function network )。在論文中，作者使用 MLP 的主要原因有 : 
1. MLP 與使用反向傳播 ( Backpropagation ) 的CNN可互相兼容
2. MLP 可以是很深層的網路結構，符合 feature re-use[^1] 的精神。

從計算上來看， 傳統 CNN 中 Conv Layer 與 NIN 中 MLPConv Layer 的差別 : 
Conv Layer : ( $x_{i,j}$ 為輸入圖像中第 $i$行$j$列的像素,$k$ 為channel )
$$
f_{i,j,k}=\max(w_k^Tx_{i,j},0)\cdots\cdots(1)
$$

MLPConv Layer : ( $n$ 為 MLP 的層數 )
$$
f_{i,j,k_1}^1=\max({w_{k_1}^1}^Tx_{i,j}+b_{k_1},0)\\
f_{i,j,k_2}^2=\max({w_{k_2}^2}^Tf_{i,j}^1+b_{k_2},0)\\
\vdots\\
f_{i,j,k_n}^n=\max({w_{k_n}^n}^Tf_{i,j}^{n-1}+b_{k_2},0)\cdots\cdots(2)
$$

從 MLPConv Layer 的計算過程來看，第一層是一般的卷積運算，之後每經過一層MLP運算，等於將上一層的各個 channels、feature maps 的資訊混合加權運算 ，這相當於在傳統的 CNN Conv Layer 中進行 Cascaded Cross Channel Parametric Pooling ( 級聯跨信道參數池化，我自己理解就是對不同 Channel 進行訊息交流的池化過程 )，經由一層又一層的 MLP layer，不同 channel 間的訊息可以一次又一次的交流，便可以實現複雜且可學習的 cross channel information 交流。

值得注意的是，Cascaded Cross Channel Parametric Pooling 這樣的結構就等同於一個 $1\times 1$ convolution layer[^註3]。



所以 NIN 中 MLPConv Layer 我們可以視為是一般傳統 CNN 卷積層後再接一個 $1\times 1$ 卷積層，也是將原本 CNN 網路變得更深的方式。

我們也可以跟 Maxout Network 來做比較 : 

$$
f_{i,j,k}=\max_{m}(w_k^Tx_{i,j})\cdots\cdots(3)
$$

(3) 式其實就會形成一個分段函數，這樣的分段函數可以逼近任何一個凸函數，而MLPConv Layer 作為一個更通用的函數逼近器，在各方面都會比 Maxout 這種凸函數逼近器來的強大。


### 全局平均池化 Global Average Pooling

傳統 CNN 中，將最後一個卷積層後的數值形成一個向量，然後作為輸入層餵進一個全連接層中，最後再以 softmax 進行分類。全連接層將卷積結構與傳統的神經網路連結起來，以卷積層作為特徵萃取，再進行分類。

這樣的方法非常容易產生 overfitting，在 AlexNet 中利用了 Dropout 技術提高模型泛化能力，並防止 overfitting 的發生。

在此論文中，作者提出了另外一種方式 --- 全局平均池化 ( Global Average Pooling ) --- 取代傳統的全連接層。

Global Average Pooling 捨棄了傳統的全連接層，於最後一個卷積層中輸出數量等同於分類數目的 feature maps，再取每一個 feature maps 的平均值輸出成一個向量 ( 維度亦為分類數目 )，丟進 softmax 中進行分類。這樣的方式有幾個優點 : 

1. 增強了 feature map 與類別之間的關係
2. Global Average Pooling 中沒有參數需要優化，可避免 overfitting
3. Global Average Pooling 將空間資訊結合，對輸入特徵的空間轉換具有更好的強健性。

作者在此段最後說明， Global Average Pooling 就是一個 regulaizer，強制將最後輸出的 feature maps 直接映射到分類機率[^註4]。



### Network In Network Structure

由上面敘述中可以清楚了解到 MLPConv Layer 以及 Global Average Pooling 的作用，而 NIN 的整體結構其實也就是底層由 MLPConv Layer 堆疊起來，頂層則是 Global Average Pooling Layer 以及 Objective Cost Layer 的一個網路結構。

![](https://i.imgur.com/RPoQgTd.png)

上圖顯示了一個具有三層 MLPConv Layers 的 NIN 結構，且每一個 MLPConv Layer 都還有三層 perceptron layer。

但其實 NIN 本身是極具彈性的，我們可以在每一個 MLPConv Layer 後面再接一個 subsampling layer ，如同傳統 CNN 結構 ; 可以調整 MLPConv Layer 的數量 ; 當然也可以調整每一個 MLPConv Layer 內 perceptron layer 的數量。

實驗結果
---

( 此部分主要都為實驗結果的闡述，我只提出幾個論文內有趣的部分，其他的大家有興趣再自行閱讀吧 )

###  Visualization of NIN

![](https://i.imgur.com/9aYZff6.png)

前面討論 Global Average Pooling 時有說到，我們將最後一個 MLPConv Layer 直接輸出與目標數量相同的 feature maps，再將其映射到相對應的機率空間中。藉由這樣的方式，我們可以瞭解到最後一層 MLPConv Layer 的輸出應該要是可以反映出各分類的 feature maps。

上圖便是對於這樣結論的應證。每一種類別的輸入，都會在對應到類別的 feature map 上面有最大的活化效果。如果我們加進了 Bounding Box 則可期望有更好的結果出現。

結論
---

作者們提出了 NIN 這樣的結構來處理分類任務，使用 MLPConv Layer 以及 Global Average Pooling 來取代傳統 CNN 的 Conv Layer 及 Fully Connected Layer，有效的提取更抽象的特徵，也防止了 onerfitting 的發生，讓整個模型更具泛化性。而我們在最後一層 MLPConv Layer 的輸出也確實可以發現到，feature maps 確實的映射到相對應的類別中，更確認了 NIN 在目標檢測中具有一定的效果。

後記
---

雖然說在這篇論文中，對於 $1\times 1$ 卷積層並沒有太大的著墨，但確實將這樣特別的卷積層概念帶給大家，在後面的 GoogLeNet 或是其他研究上都利用這樣的特殊卷積層有效地進行降維及跨 channel 訊息交流的功能。而往後的許多模型也沿用了 Global Average Pooling 的方式，進行最後的分類。這篇論文的確在整個圖像處理的進展中扮演著極重要的地位。

參考資料
---
1. [深度学习（二十六）Network In Network学习笔记](https://blog.csdn.net/hjimce/article/details/50458190)
2. [深度学习: NIN (Network in Network) 网络](https://blog.csdn.net/JNingWei/article/details/79214520)
3. [卷積神經網路(Convolutional neural network, CNN): 1×1卷積計算在做什麼](https://medium.com/@chih.sheng.huang821/%E5%8D%B7%E7%A9%8D%E7%A5%9E%E7%B6%93%E7%B6%B2%E8%B7%AF-convolutional-neural-network-cnn-1-1%E5%8D%B7%E7%A9%8D%E8%A8%88%E7%AE%97%E5%9C%A8%E5%81%9A%E4%BB%80%E9%BA%BC-7d7ebfe34b8)
4. [深度學習方法（十）：卷積神經網絡結構變化——Maxout Networks，Network In Network，Global Average Pooling](https://www.itread01.com/articles/1489258822.html)
5. Github -- [What does the confidence map mean?](https://github.com/ms-sharma/Adversarial-Semisupervised-Semantic-Segmentation/issues/4)

註釋
---

[^註1]: 
在論文 " *Representation Learning: A Review and New Perspectives* " 中提到，「深度」是討論學習策略的一個重要面向，深度結構儘管訓練不易，但仍有著絕佳優勢 : (1) 促進了 feature re-use (2) 深層結構有助於萃取出更抽象的特徵。

[^註2]: 
可參閱 "[Gradient Vanishing Problem --- 以 ReLU / Maxout 取代 Sigmoid actvation function](https://allen108108.github.io/blog/2019/10/05/Gradient%20Vanishing%20Problem%20---%20%E4%BB%A5%20ReLU%20_%20Maxout%20%E5%8F%96%E4%BB%A3%20Sigmoid%20actvation%20function/)" 一文。

[^註3]:
可參考 "[卷積神經網路 (Convolutional Neural , CNN)](https://allen108108.github.io/blog/2019/10/07/%E5%8D%B7%E7%A9%8D%E7%A5%9E%E7%B6%93%E7%B6%B2%E8%B7%AF%20(Convolutional%20Neural%20,%20CNN)/)"一文

[^註4]:
此段原文 "*We can see global average pooling as a structural regularizer that explicitly enforces feature maps to be confidence maps of concepts (categories).* "，confidence maps 簡單一點來說，其實就是將輸出的 feature maps 映射到機率空間中。這樣的方式可以使得分類器不再是黑箱作業。