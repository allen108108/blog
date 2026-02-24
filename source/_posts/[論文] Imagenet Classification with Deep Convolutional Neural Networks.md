---
title: "[論文] Imagenet Classification with Deep Convolutional Neural Networks" 
date: 2019-07-28 22:15:58
categories:
- 論文 Paper
- 卷積神經網路 Convolutional Neural Network
image: https://i.imgur.com/J3KrotA.png
mathjax: true
---

前言
---
現今的物體辨識技術基本上都是使用機器學習來進行。為了增強整個辨識的表現，我們必須收集更多的資料、使用更強大的模型以及更好的技巧來防止 overfittng 的發生。直至目前為止，我們所使用的 label Data 仍是相對較小的 data set ( 約莫幾萬張圖像的數量級 )，而現今的技術，這樣大小的資料在簡單的分類任務上已經可以做的非常好。

<!-- more -->

然而，現實生活中的物體總有著極大的可變性，要進行更精準的辨識，我們就必須收集更多的資料。雖說先前大家就已經認知到小數據集會造成一些缺點，但真正要能夠有數以百萬計的 label data 也是直到最近才變的可能。像是 LabelMe (數十萬張 fully segmented 圖像) 或是 ImageNet ( 22000的類別，包含了1500萬張高解析度圖像 )  都是大數量級的圖像資料集。

為了處理這麼大數量的圖像資料，Convolution Neural Network ( CNN ) 可以藉由改變深度及廣度來控制模型的複雜度，並且 CNN 對於圖像的本質有著強而有力的假設 ( 穩定的統計概念以及局部圖像的相依性 )，因此相較於一般的 feedforward NN 來說有更少的參數及連接使得整個 model 更容易訓練且表現與不會比 feedforward NN 還要差。

儘管 CNN 有著上述的優點，但要廣泛應用在高解析度的圖像上仍然需要很大的計算成本。所幸近年來 GPU 的發展搭配著高度優化的 2D 卷積技術才得以使 CNN 應用到這樣大型數據集上，且可避免嚴重的 overfittng。

此篇論文具體的成果如下 : 
* 在 ILSVRC ( ImageNet Large-Scale Visual Recognition Challenge ) 上使用了截至目前為止最大的 NN 之一，且有著至今為止的最好成果。
* 使用高度優化的 GPU 並行運算來處理 2D 卷積運算並且訓練整個 CNN model。
* 此網路架構有著與以往不同的特性，使得整體性能提高並減少訓練時間
* 利用一些有效的技巧來避免掉 overfitting
* 此網路架構一共有五層卷積層及三層全連接層，這樣的深度架構極為重要，刪除任何一層都會使整體性能變差。

Data Set
---

ImgeNet 上的圖像全部都由網路上收集而來，且利用 Amazon's Mechanical Turk crowd-soursing tool 進行人工標記。ImageNet 每年都會舉辦 ILSVRC ，利用 ImageNet 內的1000類別，每個類別約使用1000張圖像 ( 總計約120萬筆訓練資料，5萬筆驗證資料及15萬筆測試資料 ) 進行競賽。

由於 ImageNet 包含各種解析度的圖像，而作者的 NN 要求輸入必須為固定尺寸。因此先將 ImageNet 圖像的短邊縮成 256，再從圖像中間取 $256\times 256$ 的部分作為輸入。除了對圖像 $zero-mean$ 處理外，並沒有進行其他的預處理，也就是說我們直接以原始 RGB 圖像 (的中心)進行 model training。




架構
---

![](https://i.imgur.com/J3KrotA.png)


上面我們曾經提到有一些特殊性質可以使訓練時間減少並且性能提高，以下作者依照重要性排列介紹 : 

### Activation Function -- ReLU

一般來說，標準的方式是在每一個 neuron 上利用 Sigmoid function 或是 tanh function 作為 activation function。當我們使用 Gradient Descent 進行權重優化時，很容易發生梯度消失的狀況，而且這種飽和非線性函數[^註1]的訓練時間會比 ReLU 這種非飽和非線性函數來的久。



![](https://i.imgur.com/iej50IW.png)


上圖是一個四層神經網路 Training error rate --- Epochs 的關係圖，其中虛線是使用 tanh 作為 activation function，而實線則是利用 ReLU 作為 activation function 的結果。

由上圖來看，當我們要達到 0.25 training error rate 時，使用 ReLU 所需的 epochs 遠少於使用 tanh 所需要的 epochs。可見如果我們要用一個大的數據集來訓練一個 model，使用 ReLU 會來的比較好。

### Multiple GPUs ( 多 GPU 平行運算 )

作者使用兩顆 GPU ( GTX580 3G ) 進行並行運算。採用的方式是將所有的 Neurons 分為一半各放到兩個 GPU上做運算，除此之外還搭配一點小技巧 : GPU 之間的只在某幾層才會有進行通信 ( 第3、6、7、8層 )。這樣的配置其實是從 Cross-Validation 試驗出來的，經由 CV 的測試，我們終究可以調整出一套計算成本是我們可以負荷的通信模式。

作者這套通信模式可以使 top-1 及 top-5 error rate 分別降低 1.7% , 1.2% ( 與 GPU 之間各自獨立運作，互不通信的情況相比 )。

### Local Response Normalization ( 局部響應歸一化 / 近鄰歸一化 )

前面講到 ReLU 的特性，可以不用將輸入 normalization 來防止飽和，但作者在這裡還是引進了一種 normalization 方法，稱為 " Local Response Normalization "。

引進這樣的方法主要是為了整個模型可以更貼近現實，利用生物學上的側抑制 ( Lateral inhibition) [^註2] 行為概念來實現 Neural Network 中神經元的局部抑制。

在這篇論文中，作者給出了 LRN 的具體計算公式

![](https://i.imgur.com/tCkcLZj.png)

而其中的 $k,n,\alpha$ 與 $\beta$ 都是超參數，可以利用 Validation Set 來找出來。最後作者在某些特定的 Layer ( 第1、2層 ) 中設置 LRN。



<font color="#dd0000">**[ 補充 ]**</font>
2015年 Karen Simonyan, Andrew Zisserman 的論文 " *Very Deep Convolutional Networks for Large-Scale Image Recognition* " 中針對 LRN 有著不同的看法 : 

" *We note that none of our networks (except for one) contain Local Response Normalisation (LRN) normalisation (Krizhevsky et al., 2012): as will be shown in Sect. 4, such normalisation does not improve the performance on the ILSVRC dataset, but leads to increased memory consumption and computation time.* "

他們認為，加入 LRN 並沒有使整體表現更好，反而增加記憶體消耗與計算時間的延長。

### Overlapping Pooling 重疊池化

一般的 CNN 架構中，Pooling kernel 針對上一層 feature map 做滑窗池化的過程中，kernel 是不會重疊的 ( $2\times 2$ kernel，stride=2 )。但在此架構中，作者令 kernel 為 $3\times 3$，stride=2。
這樣一來，在池化的過程中必然會有重疊的部分，而這樣的方式確實也降低不少 error rate，並且更能避免 overfitting 的發生。


### 整體架構


![](https://i.imgur.com/gyZMo89.png)
(圖片來源 : [alexnet筆記（ImageNet Classification with Deep Convolutional Neural Networks）](https://www.twblogs.net/a/5b7c1def2b71770a43d96a77) )


* #### 輸入
    我們使用 $224\times 224\times 3$ 的圖像作為輸入的固定尺寸。
* #### 第一層 ( 卷積+池化層+LRN )
    此層使用96個 $11\times 11\times 3$ kernel 進行 stride=4 的卷積運算，隨後進行 LRN 及 Max Pooling ( $3\times 3$ kernel , stride=2 ) ，形成 $27\times 27\times 48$ ( per GPU ) 的feature maps。
* #### 第二層 ( 卷積+池化層+LRN )
    此層使用256個 $5\times 5\times 48$ kernel 進行 stride=1 , padding=2 的卷積運算，  隨後進行 LRN 及Max Pooling ( $3\times 3$ kernel , stride=2 )，形成 $13\times 13\times 128$ ( per GPU ) 的feature maps。
* #### 第三層 ( 卷積層 )
    此層使用384個 $3\times 3\times 128$ kernel 進行 stride=1 , padding=1 的卷積運算，形成 $13\times 13\times 192$ ( per GPU ) 的feature maps。
* #### 第四層 ( 卷積層 )
    同上層構造
* #### 第五層 ( 卷積層 )
    此層使用256個 $3\times 3\times 128$ kernel 進行 stride=1 , padding=1 的卷積運算，隨後進行 LRN 及 Max Pooling ( $3\times 3$ kernel , stride=2 )，形成 $6\times 6\times 128$ ( per GPU ) 的feature maps。
* #### 第六層 ( FC層 )
    將輸入 flatten 成為一維向量對此層的 4096 neurons 進行全連接
* #### 第七層 ( FC層 )
    對此層的 4096 neurons 進行全連接
* #### 第八層 ( FC層 )
    對此層 1000 neurons 進行全連接
* #### 輸出 ( Softmax )
    將此1000個輸出值經過 Softmax 轉換為機率值作為最後輸出。

每一層都會先經過 ReLU 轉換後 ( 再經由LRN、池化層 ) 才輸出，且其中第2、4、5層只與同一 GPU 上的單位做連結，而第3、6、7、8層則是進行 GPU 的通信

## 降低 Overfitting

上述的架構，會產生將近6000萬個參數，儘管最後只限定在1000分類上，這會附加一個 $2^{10}=10$ bit 的 Constrain，但仍無法避免 Overfitting 的發生。因此，作者在這裡面加了兩種克服 Overfitting 的方法 : 

### Data Augmentation 資料增強

在圖像辨識上，很常會用來減少 overfitting 的方法是使用 label-preserving transformations ( 亦即，圖像的 label 不變，但對原始圖像進行旋轉、縮放、翻轉...等方式製造出新的資料 )。在此篇論文中，作者使用了兩種資料增強的方式 : 

1. **將圖像進行變換以及水平翻轉** : 將原本的 $256\times 256$ 圖像，隨機提取 $224\times 224$ 區域作為新的訓練資料，利用這樣的方式，我們的訓練集會增加 2048 倍，且能大幅降低 overfitting 的狀況。針對測試集，我們會選取四角及中心的 5 個  $224\times 224$ 區域以及其翻轉共10個圖像進行預測，將 Softmax 輸出的 10個值進行平均。

2. **改變 RGB Channel 的強度** : 將原始圖片像素值 (RGB 三維向量) 先進行一次 Principle Component Analysis ( PCA , 主成分分析 )得到特徵向量及特徵值，根據這組特徵向量及特徵值計算出一組隨機值，加進原始像素值中。
$$
\begin{bmatrix}I_{xy}^R,I_{xy}^G,I_{xy}^B\end{bmatrix}+\begin{bmatrix}p_1,p_2,p_3\end{bmatrix}\begin{bmatrix}\alpha_1\lambda_1,\alpha_2\lambda_2,\alpha_3\lambda_3\end{bmatrix}^T
$$
這裡的 $p_i$ , $\lambda_i$ 分別為三維像素值的 $3\times 3$ Co-Variance Matrix ( 協方矩陣 ) 之特徵向量與特徵值，而 $\alpha_i$ 即為隨機值。這樣的方式其實表現了光照顏色及強度改變時的 label-preserving 現象。[^註3]


### Dropout

將每種不同的模型預測進行結合，通常可以得到一個很強大的模型，但是每一個模型都需要花上許多訓練的時間，計算成本非常高，因此衍生出 Dropout 這種方式。

Dropout 可以依照我們設定的機率值 ( 這裡作者採 0.5 為機率值 )，隨機使 Neurons 停止運作，不再進行前向 / 後向傳播。這樣的方式相當於每一次的輸入，都會產生一個不同的 Neural Network。而且這樣會強制使神經元不會依賴其他神經元的存在來學習，使整個網路可以學習出更好的特徵。

但因為我們隨機停掉一些神經元，因此在後面的測試階段我們必須進行幾何平均才能貼近現實情況。[^註4]



## Details of Learning 學習過程

下列是作者對於此模型訓練的參數設定 : 

* Optimization Algorithm : Stochastic Gradient Descent ( SGD ) with momentum ( 動量因子 $\lambda=0.9$ )
* Batch size = 128
* Regularizer : L1 regularization with $\lambda=0.0005$
* Learning Rate : 0.01 ( 當 validation error rate 沒有繼續下降時，LR 會變為0.1倍 )[^註5]
* 權重初始值 : 利用 Gaussian Distribution ( Mean=0 , STD=0.01 )這樣的分布將所有權重進行初始化。
* 偏置初始值 : 第 2、4、5 層以及所有的 FC 層Neurons 偏置初始值設為 1，其他偏置都為 0

利用這樣的設定在兩個 GTX 580 3GB GPU 上訓練 120萬筆訓練資料約90個循環一共花費了 5 -- 6 天。


## 結果

( 此部分主要都為實驗結果的闡述，我只提出幾個論文內有趣的部分，其他的大家有興趣再自行閱讀吧 )



![](https://i.imgur.com/8CsSWFD.png)



這是在第一層中學習到的 96 個 $11\times 11\times 3$ 個 kernels。上半部是 GPU-1 學習到的48個 kernels，而下半部則是 GPU-2 學習到的 Kernels。

有趣的一點是，整個網路學習到了頻率、方向以及顏色，但由於我們架構中限制連接的結果，使得兩個 GPU 會學習到不同的重點，GPU-1 學習到的 kernels 是不帶顏色的，但 GPU-2 的 kernels 卻針對顏色來學習。而且這樣的結果不論我們如何初始化權重都依然會發生。


## 探討

這篇論文作者認為在一個大量且具高度挑戰性的資料上，使用大型深度 CNN 模型處理 Supervise Problem 絕對可以有非常好的表現，但深度降低都會使性能降低，深度絕對是非常重要的架構條件之一。

作者為了簡化實現，不採用任何的 Unsupervised pre-training，儘管運算能力已經足夠強大到可以這麼做且也期望它可以對整個模型更有幫助。但簡化後我們仍能確定結果是好的，效能也提高了。

雖然好像整個模型變的更大，訓練時間拉得更長，但仍不及人類的視覺系統。最終作者還是希望能有更大更深的模型可以處理影片時間序列的結構。時間序列結構會有很多有用的訊息在其中，而這些訊息在靜態影像中是不這麼明顯的。


參考資料
---

1. Krizhevsky, A., Sutskever, I., & Hinton, G.E. (2012). ImageNet Classification with Deep Convolutional Neural Networks. Commun. ACM, 60, 84-90. (論文[下載](https://papers.nips.cc/paper/4824-imagenet-classification-with-deep-convolutional-neural-networks.pdf))
2. [【深度学习技术】LRN 局部响应归一化](https://blog.csdn.net/hduxiejun/article/details/70570086)
3. [alexnet筆記（ImageNet Classification with Deep Convolutional Neural Networks）](https://www.twblogs.net/a/5b7c1def2b71770a43d96a77)
4. StackExchange -- [What does the term saturating nonlinearities mean?](https://stats.stackexchange.com/questions/174295/what-does-the-term-saturating-nonlinearities-mean)

註釋
---

[^註1]
Definition of (non) satuating function:
* $f$ is non-saturating function $\iff\underset{z\to +\infty}{\lim}f(z)=+\infty$ or $\underset{z\to -\infty}{\lim}f(z)=+\infty$
* $f$ is saturating function $\iff$ $f$ is non-saturating function


[^註2] 
側抑制 ( Lateral inhibition) : 簡單來說就是相近的神經元彼此之間發生的抑制作用﹐即在某個神經元受到刺激而產生興奮時﹐再刺激相近的神經元﹐則後者所發生的興奮對前者產生的抑制作用。 --- 引自 [側抑制](https://linfengwei.pixnet.net/blog/post/10515788-lateral-inhibition(%E5%81%B4%E6%8A%91%E5%88%B6)) 一文


[^註3]
這樣的方式，我們又稱為 " Fancy PCA ",可參閱 " [Fancy PCA (Data Augmentation) with Scikit-Image](https://deshanadesai.github.io/notes/Fancy-PCA-with-Scikit-Image) " 一文，內有完整 code 實現。


[^註4]
可參閱 " [Regularization 方法 : Weight Decay , Early Stopping and Dropout](/ZlFtvH-0S7ORuXe3-v1hJA) " 一文。

[^註5]
在 Keras 中我們可以利用 `keras.callbacks.ReduceLROnPlateau` 來實現這樣的 LR 自動化過程。