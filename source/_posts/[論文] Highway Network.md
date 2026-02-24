---
title: "[論文] Highway Network" 
date: 2019-10-17 00:52:27
categories:
- 論文 Paper
- 卷積神經網路 Convolutional Neural Network
image: https://img-fnc.ebc.net.tw/EbcFnc/news/2018/07/15/1531677908_96181.jpg
mathjax: true
---

概要
---

有越來越多的理論及經驗告訴我們，神經網路的深度是成功的關鍵因素。然而，當神經網路的深度逐漸增加時，整體模型的訓練就會變得越來越困難，想要訓練一個極深層的網路就變成一個很難處理的問題。

這篇論文中，作者們介紹了一種使深層網路也能易於訓練的結構，稱之為 Highway Network，這樣的結構使得信息可以藉著這種 " information highway " 貫穿多層。這種結構主要由 " gate unit " 來調節整個網路的信息流 ( flow of information )。

<!-- more -->

Highway Network 可以使用 Stochastic Gradient Descent ( SGD, 隨機梯度下降 ) 以及多樣化的 activation function 來進行數百層模型的訓練，這項研究開創了深度模型以及高效率的可能性。

簡介
---

許多近期的研究透過深層網路的應用在監督式機器學習 ( supervised machine learning ) 上取得突破，網路深度 (指的是連續運算層的數量) 可能扮演著重要的腳色。舉例來說，這幾年來，利用更深的網路結構以及更小的感受野 (receptive field)使得預測 ImageNet 的準確率已經成長到 95%。

從理論的角度來看，越深層的網路結構，越能表示 (逼近) 任意函數，如同 Bengio 等人所說的，使用深層網路可為複雜的任務同時帶來計算與統計上的高效率。

然而，訓練一個越來越深層的網路，並不如同增加 layers 那麼簡單，深層網路的優化已經被證明隨著 layers 的增加，訓練會更加的困難，這項研究也導致人們對於初始化 ( initialization )、模型的訓練技巧或是在某些層中加入一些暫時性的 loss function[^註1]等面向的研究。

在這篇論文中，作者們提出了一個新的結構使得任意深度的網路優化變的可能。這項研究主要是透過 LSTM 的 gate 機制來完成 ( 但整個看起來比較像是 GRU )，藉由這樣的機制，可以使信息跨越多層卻又不會衰減，作者們稱這樣的路徑為 information highways，而這樣的網路便稱之為 Highway Network。

作者們實驗發現，highway network 可以利用簡單帶有 momentum 的 SGD 對 900 多層的網路進行優化。而以 100 多層的網路結構來看，對比傳統的網路結構， highway network 的優化與深度無關，而傳統網路的優化卻會因為 layers 的增加而越來越困難。而作者們也發現了，在 Romero 2014 年的論文 " *[FitNets: Hints for Thin Deep Nets](https://arxiv.org/abs/1412.6550)* " 中利用一個寬且淺的網路，藉由 tranfer 使一個窄且深的網路能夠被優化的方式可以用 highway network 來取代，不需要 teacher network 便可以達到類似的精確率。

Highway Network
---

若一個共有 $L$ 層的神經網路，其中第 $l$ 層 ( $l\in\{1,2,3,\cdots,L\}$ )的輸入輸出分別為 $\mathbb{x}_l,\mathbb{y}_l$，那麼在這一層我們可以這樣表示 : 

$$
\mathbb{y}_l=H(\mathbb{x}_l,W_{H,l})
$$

為了簡化，我們忽略了模型本身的 bias 以及此層的 index

$$
\mathbb{y}=H(\mathbb{x},W_H)\cdots\cdots(1)
$$

一般來說，$H$ 通常是一個 affine transformation[^註2] ( 在一般的神經網路中，若將輸入輸出、權重及 bias都視為一個矩陣的話，$Y=XW+B$，這樣的形態就是一個 affine transformation ) 後面再接一個非線性的 activation function。

作者們提出的 Highway Network 則是將 gate 機制加入 :

$$
\mathbb{y}=H(\mathbb{x},W_H)\cdot T(\mathbb{x},W_T)+\mathbb{x}\cdot C(\mathbb{x},W_C)\cdots\cdots(2)
$$

$T$ 控制了輸出應該要保留多少比例稱之為 Transform gate，而 $C$ 則是控制了輸入應該要有多少比例被保留下來，稱之為 Carry gate。

相同為了簡化我們令 $C=1-T$

$$
\mathbb{y}=H(\mathbb{x},W_H)\cdot T(\mathbb{x},W_T)+x\cdot (1-T(\mathbb{x},W_T))\cdots\cdots(3)
$$

必須要確定所有維度都必須相等才能使上式成立。

可以清楚看到，加上 gate 機制的網路結構會比最初的 ( 1 ) 式更具靈活性

$$
\mathbb{y}=\begin{cases}\mathbb{x},&\mbox{if }T(\mathbb{x},W_H)=0\\H(\mathbb{x},W_H),&\mbox{if }T(\mathbb{x},W_H)=1 \end{cases}
$$
其 Jacobian[^註3] 為
$$
\frac{d\mathbb{y}}{d\mathbb{x}}=\begin{cases}I,&\mbox{if }T(\mathbb{x},W_H)=0\\H'(\mathbb{x},W_H),&\mbox{if }T(\mathbb{x},W_H)=1 \end{cases}
$$

根據上面兩個式子可以了解到，根據 Transform gate 的輸出，Highway block 可以在原始的網路架構 $\big( \mathbb{y}=H(\mathbb{x},W_H)\big)$ 與單純只將輸入傳至下一層 $\big(  \mathbb{y}=\mathbb{x}\big)$ 這兩種極端行為之間進行平滑的轉換。

一個 Highway Network 由多個 Highway block 組成，可以藉由上面的式子計算每一個 block 輸出

$$
y_i=H_i(\mathbb{x})* T_i(\mathbb{x})+x_i*(1-T_i(\mathbb{x}))
$$


### Constructing Highway Networks

前面有談到，為了使 (3) 式成立，必須要確保輸入輸出及轉換維度上面都必須要相等，但在建構網路時，有時候還是會遇到輸入輸出維度不同的時候，論文中提出兩個方法來做改善 : 

* 利用適當的 Subsampling 來進行維度調整
* 利用 padding 方式來做維度轉換

本論文則採用第二種方式來做維度上的調整。

此外，Convolutional Hiway Network 架構上也類似 Fully connected Network，但概念上使用權重共享以及局部感受野的方式，因此若要調整維度也都以第二種方式進行處理。

### Training Deep Highway Networks

對一般的深層網路來說，除非是特殊的初始化方法，否則使用 SGD 訓練容易在一開始的時候就呈現停滯狀態進而使得前後向傳播傳遞的訊號 variance 維持初始狀態。這樣的初始化方式取決於 $H$ 函數的型態。[^註4]

作者給出此論文中的 $T(\mathbb{x})$ 之定義

$$
T(\mathbb{x})=\sigma(W_T^T\mathbb{x}+\mathbb{b}_T)\\
\text{where }W_T \text{ is the weight matrix and }\mathbb{b}_T\text{ is the bias vector for }T 
$$

從定義可以知道，$T(\mathbb{x})\in (0,1)$ 且獨立於 $H(\mathbb{x},W_H)$，如此一來，我們便可以獨立地進行初始化，避免因 $H(\mathbb{x},W_H)$ 造成的 SGD 的停滯狀況。

Gers 1999年的論文 " *Learning to forget: Continual prediction with LSTM.* " 中針對最初的 LSTM 版本加入了 Forget gate，並且藉由設置初始 bias 為負值來讓 LSTM 在訓練早期可以以 " 記住 " 為主要的任務。而在 Highway Network 也一樣希望在早期的訓練整個系統傾向於 " carry " 住當下的資訊，針對初始 bias 有負值的設置。

( 稍微解釋一下，Gate 的設計中，如果將 bias 設為負值，相當於我們讓 gate 不容易被打開，在 Highway Network 中，為了讓早期的輸入都要被 carry 起來，便希望將 Transform gate 關閉，使 $\mathbb{y}=H(\mathbb{x},W_H)\cdot T(\mathbb{x},W_T)+x\cdot (1-T(\mathbb{x},W_T))$ 的第二項可以被完全保留下來 )

在作者的實驗中發現這樣的初始化方法可以於不同初始分布的 $W_T$ 及各種不同 $H$ 的 activation function  的深層網路結構進行有效學習。( 這句話實在有點繞口，大概就是指這樣的初始化方法適合各種不同初始值的深度學習結構吧 )

這項結果是很重要的，因為一般來說不太可能針對各種不同的 $H$ 都找到有效的初始化方法。

Experiments
---

( 依照慣例省略，有興趣者請直接參閱論文 )


Aanlysis
---

![](https://i.imgur.com/PxArooN.png)

作者們利用上圖展示出利用 MNIST 與 CIFAR-100 分別訓練出來的兩個最佳 50 層 fully connected highway networks[^註5] 的內部運作。 (Transform gat bias 分別初始為 -2 及 -4)

* 兩個 Networks 均為 50 層 (y軸)
* 第一層主要的作用就是將 input 維度轉換成 50 維 (x軸)
* 第一行展現了訓練過程後 Transform gate bias 在每一層的狀態
* 第二行顯示了訓練過程後利用一萬筆資料計算出來的每一層 Transform gate output 平均輸出。
* 第三、四行顯示了訓練過程後利用單一筆隨機抽選出來的資料的每一層 Transform gate output 以及 每一個維度的輸出狀態

從第一行來看，Transform gate bias 會因為深度而遞增 ( 表示 Transform gate 隨身度越來越容易被開啟 )，但第二三行卻顯示 Transform gate output 活躍狀態隨著深度而遞減。作者們給出的解釋是，在淺層的部分，雖然 bias 較低，但卻非關閉整個 Transform gate，而是使其更具選擇的能力。這樣的狀況在 CIFAR-100 訓練出來的 model 中可以很明顯地看見這樣的趨勢，在 MNIST 訓練的 model 沒這麼明顯，但也略有一點趨勢存在。

而第四行則是完整體現了 "Highway" 的概念，大多數的輸出變化都在淺層 ( MNIST 大約在前10層，而 CIFAR-100 約發生在前 30 層 ) 的時候發生，深層幾乎就是直接跨越多層直接傳達到更深層。

結論
---

學習如何利用神經網路傳遞信息有助於藉由增進 credit assignment[^註6] 以及簡化訓練來擴展其應用層面。但要訓練一個極深層的網路仍然非常困難。尤其是在沒有辦法顯著增加其網路規模的前提下。

Highway Network 提供了一種新型的網路結構，只需使用 SGD 便可以解決一般深層網路訓練困難的問題，甚至結構高達數百層都不會對訓練造成阻礙。

Highway Network 不僅為利用無限制深度來解決複雜問題的研究提供了可能性，且可使用各種沒有有效初始化方法的 activation function 來適應各種不同的任務。



註釋
---

[^註1]: 
參考論文 " [*Going Deeper with Convolutions*](https://hackmd.io/@allen108108/HkNuF7E7S) "，在 VGG Net 中會對某些層加入一些輔助分類器 (auxiliary classifier)

[^註2]: 
仿射變換 Affine Transformation 指的就是原空間經過一個線性變換再加上一個平移量所構成新空間的變換。具有共線不變性 (直線經過仿射變換後仍然是一條直線 ) 及比例不變性 ( 經過仿射變換後兩點距離不變 ) 兩特點。詳細可參閱 " [線代啟示錄 : 仿射變換](https://ccjou.wordpress.com/2011/03/24/%E4%BB%BF%E5%B0%84%E8%AE%8A%E6%8F%9B/) " 以及 " [知乎 : 如何通俗地讲解「仿射变换」这个概念？](https://www.zhihu.com/question/20666664) "

[^註3]: 
簡單來說就是其值為向量的函數之梯度。詳細請參閱 " [線代啟示錄 : Jacobian 矩陣與行列式](https://ccjou.wordpress.com/2012/11/26/jacobian-%E7%9F%A9%E9%99%A3%E8%88%87%E8%A1%8C%E5%88%97%E5%BC%8F/) "、" [【math】梯度（gradient）、雅克比矩阵（Jacobian）、海森矩阵（Hessian）](https://blog.csdn.net/DSbatigol/article/details/12558891) "

[^註4]: 
其實這一小段我不是太能理解他在說什麼，附上原文供參 : " *For plain deep networks, training with SGD stalls at the beginning unless a speciﬁc weight initialization scheme is used such that the variance of the signals during forward and backward propagation is preserved initially (Glorot & Bengio, 2010; He et al., 2015). This initialization depends on the exact functional form of H.* "

[^註5]: 
怎麼找出這樣的最佳模型 ? 論文提到是對超參數進行 random grid 後再比較最低 training error 而找出來的。

[^註6]: 
Credit assignment 大多指的是在 reinforcement learning 中， reward 會有延遲的現象，稱之為 Credit assignment problem。舉例來說，利用 reinforcement learning 訓練下棋的過程中，每一步棋幾乎都不會有 reward，直到棋局結束確定勝負才會得到 reward，在中間的每一步棋，究竟是哪一步會得到最後的 reward ? 又或者每一步棋會有多少比例影響著最後的 reward ? 這些都是 Credit assignment problem。