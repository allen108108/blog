---
title: "Semi-supervised Learning"
date: 2019-10-07 15:54:56
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning
image: https://i.imgur.com/hos5ud0.png
mathjax: true
---


>* **本文討論內容請參考: 
Machine Learning (2017) 第十二講 : Semi-supervised**
> 
>*  **本篇所有圖片部分由筆者製作， Machine Learning (2017) 課程內容講義**


<!-- more -->

剛開始學習機器學習中，我們大多看到的是 Supervised learning : $\left\{(x_r,y_r)\right\}_{r=1}^R$，每一個 $x_r$，都有一個 Label $y_r$，而 Semi-Superviced Learning 便是只有一部分資料有標籤，而另外一部分的資料是沒有標籤的，實務上沒有標籤的資料遠多於已經標籤的資料 : $\left\{(x_r,y_r)\right\}_{r=1}^R$ and $\left\{x_u\right\}_{u=R}^{R+U}$， $U\gg R$。

Semi-Superviced Learning 在實務上是非常常見的，收集資料這件事情並不是什麼難事，我們可以輕鬆就收集到足夠多的資料，然而要為這些資料進行標記卻是非常困難的，所以 Semi-Superviced Learning 在此時便能發揮其功能。

Semi-Superviced Learning 又可分為兩種 : 

**1. Transductive learning : 未標記的資料亦為測試資料**
**2. Inductive learning : 未標記的資料不做為測試資料**

一般來說，Transductive learning 的效果會比 Inductive learning 還要好。

## Approach 1 : Generative Model

我們知道，Generative Model (GM) 與 Discriminative Model (DM) 不同的地方在於 GM 會訓練出訓練資料 $\left\{x_r\right\}_{r=1}^{R}$ 對於每一個不同 class $C_1,C_2,\cdots$ 的機率分布模型。

亦即 GM 學習到的會是 $\boldsymbol{P}(C_1),\boldsymbol{P}(C_2)\cdots \mu_1,\mu_2,\cdots,and\ \Sigma$，
而DM 學習到的會是 $\boldsymbol{P}(C_1\mid x),\boldsymbol{P}(C_2\mid x)\cdots$

在 Supervised Learning 上，我們便可以直接利用這樣的分布來決定分類

![](https://i.imgur.com/8d3GgWD.png)

但在 Semi-Supervised Learning 上我們必須利用未標籤的資料來輔助才能得到比較好的分類模型

![](https://i.imgur.com/2AInKRi.png)

### 如何利用未標籤資料進行生成模型推估 ?

我們使用迭代的方式進行模型的推估，也就是說利用一筆一筆的未標籤資料來調整模型



![](https://i.imgur.com/x2P9nWs.png =500x)



理論上這樣的模型最後會收斂，但 Initialization 的設定會影響最後結果。

這樣的演算法的理論其實用到的概念很簡單 : 

***我們先對有標記的資料計算出各自的 Maximum likelihood，再利用未標籤的資料來最大化 Maximum likelihood***


![](https://i.imgur.com/UBzwvIL.png)




## Approach 2 : Self-training 

Self-training 首先利用已標籤的資料進行訓練，再使用訓練出來的 model 對未標籤資料進行標記。然後我們再取一部分的未標籤資料加到 training data 中重新訓練一個 model，進行上述的迭代訓練來取得一個好的 model。



![](https://i.imgur.com/hos5ud0.png =500x)



這是一個很直覺的方式，但這個方法要能夠取得一個夠好的 model 必須奠基在<font color="#dd0000">**「各類別之間有明確的分野」**</font>，也就是說，每一個分類之間不存在模糊的地帶，有一個明顯的 gap。

而且，從 Self-training 的步驟來看，有幾個狀況並不適用 Self-training : 

1. Regression 問題
2. Soft label 的 Neural Network

這兩個不適用於 Self-training 的理由都一樣，就是我們無法在這樣的演算法中迭代得到更好的 model。當 label 是實數，我們將加了 label 的未標籤資料放進 training data 中並不會改變 $f^*$。

Soft-label 要進行 gradient descent 也會造成梯度不會改變的問題。

![](https://i.imgur.com/466qa2g.png)
****

要解決 Neural Network 的問題，可以使用 Entropy-based reluarization 來處理。

因為 Neural Network 輸出會是機率分布，我們可以加入 entropy 的 constrain 來規範，這直覺上來說是合理的，因為我們本來就希望資料本身要夠集中才有參考價值。



![](https://i.imgur.com/YwEaB3I.png =450x)



這個演算法其實也延伸出了 Semi-Supervised SVM 。

一般 Supervised SVM 的概念就是最大化 margin 且最小化 error 的分類方式，而 Semi-Supervised SVM 窮舉出所有未標籤資料的可能 label ，針對每一種可能性進行 SVM ，找出最小 error 狀況。




![](https://i.imgur.com/4AHWn2h.png =450x)



而論文 *Transductive Inference for Text Classication using Support Vector Machines* 提出了近似的估計方式解決當未標籤資料過多，無法窮舉所有情況的解決方式。

## Approach 3 : Graph Approach

為了瞭解圖論在這邊的應用，我們先給出一些定義 : 

首先，我們將所有的資料 $x_1,x_2,\cdots,x_R,\cdots,x_{R+U}$ 視為空間中的點
* $Similarity\ of\ x_i,x_j=s(x_i,x_j)=e^{-\gamma\|x_i-x_j\|^2}$ ( Gaussian Radial Basis Function )
* $Add\ edge\ with\ K-Nearest Neighbor$ ( $or\ e-Nearest Neighbor$ )
* $Edge\ weight\ of\ x_i,x_j \propto s(x_i,x_j)$
* $Smothness\ of\ the\ labels\ on\ the\ graph=S=\displaystyle{\frac{1}{2}}\sum\limits_{i,j}w_{i,j}(y_i-y_j)^2$
( $Smaller\ S\ means\ Smother$ )
* $\boldsymbol{Y}=\begin{bmatrix}\cdots y_i\cdots y_j\cdots\end{bmatrix}_{1\times (R+U)}^T$
* $\boldsymbol{W}=\begin{bmatrix}w_{i,j}\end{bmatrix}_{(R+U)\times (R+U)}$
* $\boldsymbol{D}\ is\ a\ diagonal\ matrix\ d_{i,i}=\sum\limits_{k=1}^{R+U} w_{i,k}$
* $Laplacian\ matrix=\boldsymbol{L}=\boldsymbol{D}-\boldsymbol{W}$



![](https://i.imgur.com/xo8QwjH.png =300x)



從上面的定義我們可以推得 :  $S=\boldsymbol{Y}^T\boldsymbol{L}\boldsymbol{Y}$

我們清楚了圖論的一些概念後，從圖論的方式來進行 Semi-Supervised Learning 跟 Self-training 奠基的假設完全不同。Graph Approach 必須基於一個 Smoothness 假設上才能確保這樣的方式可以做出一個夠好的 model。

那什麼是 Smoothness 假設呢 ?

簡單來說，$x_1,x_2$ 如果在一個稠密的區域裡面夠接近，那麼他們的 label 必然會相同。 ( 換句話說，$x_1,x_2$ 可藉由一個高密度的路徑相互連接 )



![](https://i.imgur.com/dm1t4z4.png =400x)




最後我們可以利用上面的概念，重新將資料間的關係做一個定義，丟到 neural network 中進行訓練，而待訓練的當然就是 edge weight。

然而我們一樣為了避免 overfitting 的問題，還有整個 graph 上對 smoothness 的要求，我們在 Loss function 上仍會加上一個 regularizer $=S$。



![](https://i.imgur.com/Jidug86.png =500x)



## Approach 4 : Better Representation

( 待 Unsupervised Learning 進行詳細說明 )

簡而言之就是我們觀察到的現象通常都比較複雜，試圖去找出潛在因素，則可以讓訓練變得比較簡單。