---
title: Gradient Vanishing Problem --- 以 ReLU / Maxout 取代 Sigmoid actvation function
date: 2019-10-05 01:48:30
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning
image: https://i.imgur.com/mlzgQy1.png
mathjax: true
---


>* 本文內容參考自Hung-yi Lee ,  [Machine Learning](http://speech.ee.ntu.edu.tw/~tlkagk/courses_ML17_2.html)(2017) 課程內容 : Tips for Training DNN
>* 本文圖片部分來自於課程講義內容
>

<!-- more -->

## 梯度消失 Gradient Vanish

「類似」於 Sigmoid function 的激勵函數，普遍帶有梯度消失 ( Gradient Vanish ) 的隱憂，那究竟什麼是梯度消失?

$Sigmoid\ function=\theta(s)=\displaystyle{\frac{1}{1+e^{-s}}}$

此函數圖形為

![](https://i.imgur.com/mlzgQy1.png =450x)
(圖片取自 :　[Wikipedia --- Sigmoid function](https://zh.wikipedia.org/wiki/S%E5%87%BD%E6%95%B0))

從圖上可知，其圖形切線斜率 ( 導數 ) 不會超過0.25，如此情況當我們在進行 Gradient Descent 的過程中，隨著迭代次數的增加，參數的更新會越來越緩慢 而整個 train 不起來。[^註1]



## Rectified Linear Unit ( ReLU )

從上述對 Gradient Vanish 的觀察，以 Sigmoid 作為 actvation function 雖然是一個平滑便於求導數的函數且能壓縮資料到0-1之間，但是卻有梯度消失的問題，也因此衍生出了 Rectified Linear Unit ( ReLU ) 這樣的 activation function。

$ReLU(x)=max(0,x)$

![](https://i.imgur.com/XVCR5Wp.png)
(圖片取自 : [ReLU : Not a Differentiable Function: Why used in Gradient Based Optimization? and Other Generalizations of ReLU.](https://medium.com/@kanchansarkar/relu-not-a-differentiable-function-why-used-in-gradient-based-optimization-7fef3a4cecec) )

ReLU 的優點除了可以有效「減輕」梯度消失外[^註2]，計算成本也大幅下降，也有相關學者提出 ReLU 也很接近生物神經元的激活模型[^註3]，這種種優點也使 ReLU 在深度學習中被廣泛應用，成為 activation function 的優先選擇之一。

ReLU 函數的稀疏性 ( Sparsity )，使得整個神經網路變得更輕巧、更多樣性，但卻不會使梯度變得越來越小。

![](https://i.imgur.com/di2NKzI.png)

如同本文註釋[^註2]中提到的，ReLU 並非完全沒有缺點，為了改善大量神經元壞死的狀況，便有人提出了改進版本的 ReLU --- Leaky ReLU & Parametric Relu。

![](https://i.imgur.com/OhIc8Xq.png)

值得一提的是，這些改進版本並不一定會表現得比 ReLU 還要好，當 $\alpha$ 值過小時，仍然會有梯度消失的情況出現。再來，這兩者改進版本跟 Sigmoid 不同都是無上下界函數，如果遇到很深的神經網路，碰到極多的權重及連續乘積後，縱使 Leaky (Parametric) ReLU 導數為1或小於1的數，仍有可能造成梯度爆炸的狀況出現。

## Maxout 

![](https://i.imgur.com/eKJmGeq.png)

Maxout 是一種可自行學習的 activation function，跟 ReLU 類似，但比 ReLU 更有彈性，我們只需設置要比較的 neurons 個數，Maxout 便能訓練出多樣化的 activation function。

我們其實可以說， ReLU 或是任何一種 Convex activation function 都是 Maxout 的特例。

![](https://i.imgur.com/BOZPTjl.png)

![](https://i.imgur.com/0is8BGa.png)
(取自論文 Ian J. Goodfellow, David Warde-Farley, Mehdi Mirza, Aaron Courville, Yoshua Bengio (2013) . *Maxout Network*)

Maxout 概念也衍生出了 Convolution Neural Network ( CNN )架構中的 Pooling Layer ( 池化層 )。

總結來說，基本上，梯度消失的原因出現在 Backpropagation 中的連乘項，這當中導致梯度消失的原因不會只有 activation function 導數是否小於1，梯度消失算是一個非常綜合性的問題，即使改善 activation function 也只是稍微減緩某一個造成梯度消失的原因。就目前現有的 activation function 來說，ReLU 仍是一個不錯的優先選擇。


註釋
---

[^註1]:
假如現在有一個 L 層 Deep Neural network ，activation function 為 sigmoid，
第 $l$ 層中，從前一層第 $i$ 個neuron到此層第 $j$ 個 neuron 之權重為 $w_{ij}^l$
第 $l$ 層第 $j$ 個neuron 輸入為 $s_j^l=\sum\limits_{i=0}^{d^{l-1}}w_{ij}^lx_i^{l-1}$，輸出為 $x_j^l=\theta(s_j^l)$
![](https://i.imgur.com/T4tZkaW.png =450x)
根據反向傳播法 Backpropagation
針對某分量權重來看權重的更新 
$$
w_{ij}^l\leftarrow w_{ij}^l-\eta\cdot\displaystyle{\frac{\partial L}{\partial w_{ij}^l}}$$
其中 
$$
\displaystyle{\frac{\partial L}{\partial w_{ij}^l}}=\displaystyle{\frac{\partial L}{\partial s_j^l}\cdot\frac{\partial s_j^l}{\partial w_{ij}^l}}=\displaystyle{\frac{\partial L}{\partial s_j^l}}\cdot x_i^{l-1}=\sum\limits_{k=1}^{d^{l+1}}\displaystyle{\frac{\partial L}{\partial s_k^{l+1}}\cdot\frac{\partial s_k^{l+1}}{\partial x_j^l}\cdot\frac{\partial x_j^l}{\partial s_j^l}}=\sum\limits_{k=1}^{d^{l+1}}\displaystyle{\frac{\partial L}{\partial s_k^{l+1}}\cdot\frac{\partial s_k^{l+1}}{\partial x_j^l}}\cdot\theta'(s_j^l)
$$
從上式我們可以知道，當我們層數很多的時候 ( $l$極大 ) ，$\displaystyle{\frac{\partial L}{\partial w_{ij}^l}}$ 這一項會產生非常多的 $\theta'$ 相乘，而每一個 $\theta'<1$ ，最後這個項就會幾乎趨近於 0 ，導致權重的更新十分緩慢。

[^註2]:
ReLU 並非完全沒有梯度消失的問題，輸入值若為負數，輸出便為0，導致某些神經元不會被 activate，這是優點也是缺點，雖然能讓整個神經網路訓練速度、計算成本都大幅降低，也能讓整個神經網路更多樣性，但也會造成層數過多時，有很高比例的神經元將會沒有運作，造成跟梯度消失類似的效果。
參考 : [深度学习解密：我的梯度怎么消失了？](https://www.jqr.com/article/000284)

[^註3]:
參考 [ReLu(Rectified Linear Units)激活函数](https://www.cnblogs.com/neopenx/p/4453161.html)