---
title: "為什麼需要反向傳播 ? Why Backpropagation ?"
date: 2020-06-01 13:56:12
categories:
- 深度學習 Deep Learning
image: https://i.imgur.com/sBMXOoA.png
mathjax: true
---


反向傳播是現今深度學習中非常重要的一個演算法，利用反向傳播，我們可以用有效率的方式找到損失函數對於權重的梯度，進而利用梯度下降法 ( Gradient Descent ) 來對每一個權重進行優化。梯度下降法在本文不另贅述，有興趣的話可以參考筆者很早寫的一篇簡單介紹 " [Gradient descent 梯度下降](http://bit.ly/32EPtzi) "，本文將會簡單介紹整個反向傳播的運作原理，以及到底為什麼我們會需要反向傳播 ?

<!-- more -->

反向傳播 Backpropagation
---

反向傳播最主要的概念，就是將誤差值往回傳遞資訊，使權重可以利用這樣的資訊進行梯度下降法來更新權重，進一步的降低誤差。為什麼要利用這樣的方式來更新權重 ? 在這一個部分筆者暫時先不進行解釋，我們先來了解所謂的反向傳播應該怎麼傳播，最後，利用一個簡單的例子來手推一次反向傳播過程。

### 符號定義


![](https://i.imgur.com/wllk0uc.png)

給定一個 $L$ 層神經網路如上圖所示，輸入層為 $n$ 維，輸出層 $m$ 維，共有 $L$ 層，且任一 $l$ 層中都有 $n_{l}$ 個神經元。為了方便接下來的推導，先定義一個特定的損失函數 : 

$$
E=\dfrac{1}{2}\sum_{k=1}^m(t_{k}-o_{k})^2=\dfrac{1}{2}\sum_{k=1}^mE_k
$$

其中，輸出向量維度為 $p$，$t_i$ 指的是目標向量第 $i$ 維之值，$o_i$ 則是輸出向量第 $i$ 維之值。此外，我們可以針對第 $l$ 層中第 $j$ 個神經元 (上圖紅色神經元) 給出以下符號定義。


![](https://i.imgur.com/pXVrJRl.png)


前一層中第 $i$ 個神經元與之連結之權重 $w^l_{ij}$，則此神經元之輸入為

$$
net_j^l=\sum_{i=1}^{n^{l-1}}w_{ij}^lf(net_i^{l-1})
$$

輸出則為 $o_j^l=f(net_{l})$，其中 $f$ 為神經網路中的 activation function ( 本文假設各層 activation function 均相同 )，此函數的作用可以增加神經網路整體的非線性能力。

### 反向傳播演算法

從梯度下降的角度進行最佳化，那我們就要想辦法取得每一個

$$
\dfrac{\partial E}{\partial w_{ij}^l}
$$

先看最後一層，這樣的偏微分並不難求得，根據微積分連鎖率 (Chian Rule) : 

$$
\dfrac{\partial E}{\partial w_{ij}^L}=\dfrac{\partial E}{\partial net_{j}^L}\dfrac{\partial net_{j}^L}{\partial w_{ij}^L}\\=\dfrac{\partial E}{\partial o_j^L}\dfrac{\partial o_j^L}{\partial net_{j}^L}\dfrac{\partial net_{j}^L}{\partial w_{ij}^L}\\
=\dfrac{\partial}{\partial o_j^L}\Big(\dfrac{1}{2}\sum_{k=1}^m(t_{k}-o_{k})^2\Big)\dfrac{\partial}{\partial net_{j}^L}\Big(f(net_j^{L})\Big)\dfrac{\partial}{\partial w_{ij}^L}\Big(\sum_{k=1}^{n^{L-1}}w_{kj}^Lf(net_k^{L-1})\Big)\\
=2\cdot\dfrac{1}{2}(t_j-o_j)\cdot f'(net_j^{L})\cdot f(net_i^{L-1})\text{     .............. (1)}
$$

如果是中間層的話呢 ?

$$
\dfrac{\partial E}{\partial w_{ij}^l}=\dfrac{\partial E}{\partial net_{j}^l}\dfrac{\partial net_{j}^l}{\partial w_{ij}^l}\\
\overset{let}{=}\delta_j^l\dfrac{\partial net_{j}^l}{\partial w_{ij}^l}\\
=\delta_j^l\cdot f(net_i^{l-1}){     ........ (2)}
$$

其中

$$
\delta_j^l=\dfrac{\partial E}{\partial net_{j}^l}\\
=\sum_{k=1}^{n^{l+1}}\dfrac{\partial E}{\partial net_{j}^{l+1}}\dfrac{\partial net_{j}^{l+1}}{\partial o_j^{l}}\dfrac{\partial o_j^{l}}{\partial net_{j}^l}\\
=\sum_{k=1}^{n^{l+1}}\delta_k^{l+1}\cdot w_{jk}^{l+1}\cdot f'(net_j^l)\text{     .............. (3)}
$$


由 (1) (2) 式中確認了整個梯度的計算方式，只要知道每一個 $net_{i}^l$ ， $w_{ij}^l$ 及 $\delta_k^{l+1}$ 便可找出誤差對每一個權重的偏微分。再由 (3) 式可以知道，任一個 $\delta_j^l$ 均可由 $\delta^{l+1},\delta^{l+2},\cdots ,\delta^{L}$ 計算出來。因此我們將整個反向傳播法 Backpropagation 分成兩個步驟進行 : 

* 前向傳播 Forward Pass
輸入向量 $x_1,x_2,\cdots,x_n$ 計算出每一層中每一個神經元的 $net_i^l$ 以及相對應的權重 $w_{ij}^l$ 


* 反向傳播 Backward Pass
前向傳播後所計算出來的 $E$ 計算出最後一層的 $\delta^L$ 後開始往前退算出所有的 $\delta$

經由前後向傳播得到的值便可以計算出整個梯度，再來利用梯度下降法來更新梯度。

$$
w_{ij}^l \longleftarrow w_{ij}^l-\eta\cdot \dfrac{\partial E}{\partial w_{ij}^l}
$$


誤差傳播
---

再許多的反向傳播部落格文章中，甚至 Sepp Hochreiter 提出的 LSTM 論文 " [*LONG SHORT-TERM MEMORY*](https://www.bioinf.jku.at/publications/older/2604.pdf) " 中都有提及反向傳播法的概念可以視為是一個「誤差傳播」的方式。但是究竟誤差是怎麼傳播的呢 ? 

要知道傳播的方式，我們先定義 $(t_j-o_j)=\delta_j^L$，那麼 : 

$$
\dfrac{\partial E}{\partial w_{ij}^L}=\delta_j^L\cdot f'(net_j^{L})\cdot f(net_i^{L-1})
$$

$$
\dfrac{\partial E}{\partial w_{ij}^{L-1}}=\delta_j^{L-1}\cdot f(net_i^{L-2})=\sum_{k=1}^{n^{L}}\delta_k^{L}\cdot w_{jk}^{L}\cdot f'(net_j^{L-1})\cdot f(net_i^{L-2})
$$

$$
\dfrac{\partial E}{\partial w_{ij}^{L-2}}=\delta_j^{L-2}\cdot f(net_i^{L-3})=\sum_{k=1}^{n^{L-1}}\delta_k^{L-1}\cdot w_{jk}^{L-1}\cdot f'(net_j^{L-2})\cdot f(net_i^{L-3})
$$

$$
\vdots
$$

從上面的式子來看，都必須先計算出後面一層的 $\delta$ 乘上相對應的權重才能得到誤差函數對每一層權重的偏微分，如果我們把 $\delta$ 放進神經網路圖中，就可以看出其「傳遞」的模式了


![](https://i.imgur.com/vYSMDN2.png)

從這樣的傳遞方式，也更可以理解反向傳播法其實就是「誤差反向傳播法」，藉由誤差乘以權重的傳遞，來計算出梯度。


為什麼要用反向傳播法 ?
---

 最直覺的一個問題是，**為什麼不直接計算梯度** 就好 ? 如果我們能寫出整個誤差函數，要寫出誤差對每一層權重的偏導函數，當然不是問題，但是如果今天的整個神經網路長成這樣的話呢 ?

![](https://i.imgur.com/GsecI9n.png)
(圖片來源 : [Andrej Karpathy's Twitter](https://twitter.com/karpathy/status/597631909930242048?lang=en))

我想，瘋了才會想要把整個誤差函數寫出來吧。

再者，當我們想要針對每一個權重進行偏微分計算的時候，在計算過程中，會有許多部分會被重複計算到，如果今天整個網路結構如上圖般複雜，那麼這樣造成的計算冗餘將會十分驚人。

對於電腦而言，反向傳播法之所以能夠至今屹立不搖，說穿了，就是讓電腦更有效率的利用微積分的鏈鎖率來進行梯度的計算。下面我們用一個簡單的例子來實際進行反向傳播的計算，或許能夠更深刻理解其效率。


One Simple Example ...
---

我們假定有一個兩層的神經網路，輸入維度為 3 ，輸出維度為 2 ，並為了簡化討論過程，筆者使用一個簡單的線性函數 $f(x)=x$ 作為 Activation Function。

現有一筆資料 $x = (x_1,x_2,x_3) = (1,9,3)$，其真實標註為 $y = (-183, 160)$，一開始神經網路隨機初始化所有連結權重，我們便可以先進行一次前向傳播，計算出誤差訊號 $\delta_1^2$ 及 $\delta_1^2$。

![](https://i.imgur.com/40iLvCv.png)

之後，我們可以開始傳播這個誤差訊號，並且利用前向傳播的資訊加上誤差訊號的傳播，計算出第二層的權重偏微分。

![](https://i.imgur.com/sBMXOoA.png)

接著，我們利用前向傳播的權重傳遞下一個誤差訊號至第一層


![](https://i.imgur.com/HYx1cxc.png)

當然也就可以藉此計算出第一層的權重偏微分

![](https://i.imgur.com/3aKbZhX.png)

這個例子，筆者僅推導一小部分的偏微分以及誤差訊號，但是其他的部分都是用相同的方式進行計算，有興趣的讀者可以自己手動計算看看。


後記
---

在學習深度學習的過程中，必然熟悉 『梯度下降法』 與 『反向傳播法』這兩個詞彙，但大多數人一開始大多僅止於概念上的理解，但對於實際上的運作以及真正的意義上卻不見得那麼深刻的體會，當然，筆者就是屬於這樣的人。

經過一段時間的自學以及實際開發專案後，總會希望自己的學習更加踏實，便開始思考，要怎麼對於這些基本概念更加熟悉，便有了這篇文章，雖然從第一次聽見 『反向傳播』這個名詞直到完成這篇文章，中間隔了非常久的時間，但是，這段時間中的經驗、思考以及淬鍊，讓我在審視這些基本觀念時，變得更加清晰。

或許，當我們前進到一個階段時，不妨回頭看看，這中間聽過的、看過的，自己究竟了解多少，筆者認為，這樣的不斷審視自己，更能了解自己學習的盲點。




Reference
---

1. [如何直观地解释 backpropagation 算法？](https://www.zhihu.com/question/27239198?rf=24827633)
2. [反向传播算法](https://zhuanlan.zhihu.com/p/40761721)
3. [Principles of training multi-layer neural network using backpropagation](http://galaxy.agh.edu.pl/~vlsi/AI/backp_t_en/backprop.html)
4. [Backpropagation Derivation - Delta Rule](https://blog.yani.io/deltarule/) 
5. Raul Rojas. (2013). "[*Neural Networks : A Systematic Introduction*](https://page.mi.fu-berlin.de/rojas/neural/neuron.pdf)". Springer Science & Business Media.
6. Hsuan-Tien Lin. (2017)."*[Machine Learning Techniques : 	neural network](https://www.youtube.com/watch?v=fFSi5I3OLtk&feature=youtu.be)*" from National Taiwan University .
7. Fei-Fei Li, Justin Johnson & Serena Yeung. (2017). "*[CS231n: Convolutional Neural Networks for Visual Recognition.](http://cs231n.stanford.edu/2017/)*" Spring, 2017, from Stanford University.


