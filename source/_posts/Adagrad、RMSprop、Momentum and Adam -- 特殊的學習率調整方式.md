---
title: Adagrad、RMSprop、Momentum and Adam -- 特殊的學習率調整方式
date: 2019-10-05 01:37:03
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning
image: https://i.imgur.com/coLjLhr.gif
mathjax: true
---

>* 本文內容節錄自Hung-yi Lee ,  [Machine Learning](http://speech.ee.ntu.edu.tw/~tlkagk/courses_ML17_2.html)(2017) 課程內容 : Gradient Descent、Tips for training DNN
>* 本文圖片均來自於課程講義內容
>

<!-- more -->

在深度學習中，我們進行優化的方式大多使用的是 Gradient Descent，其一般化的形式 

矩陣型態 : $\boldsymbol{W}^{t+1}\leftarrow\boldsymbol{W}^t-\eta\cdot\nabla L(\boldsymbol{W}^t)$

我們也可以單看其中一個分量權重 : $w_i^{t+1}\leftarrow w_i^t-\eta\cdot\frac{\partial L}{\partial w_i}$ 

我們曾經在 [Gradient descent 梯度下降](https://hackmd.io/s/ryQypiDK4) 一文中有討論過，若 $\eta$ ( 學習率 )固定時，太大太小都可能讓我們在優化的過程中遇到困難，最好的方式就是讓 $\eta$ 隨著優化的過程逐漸地減少。[^1]

[^1]: 在 Keras 裡面 ，當我們要進行 model compile 時，需要設置一個 optimizer 參數，系統內提供了許多的優化器可供使用 : RMSprop、SGD、Adagrad、....，這些都是基於 Gradient Descent 之上，但在學習率上面進行不同的設置。

## Adagrad --- 彈性使用 Learning Rate

$\eta$ 應該怎麼設 ? 跟次數成反比的 $\frac{1}{t}$ decay 是最簡單的方式 $\eta^t=\frac{\eta}{\sqrt{t+1}}$ ，但這顯然太過簡單。

Adagrad 所使用的 $\eta^t=\displaystyle{\frac{\eta}{\sqrt{\sum\limits_{n=1}^{t}(\displaystyle{\frac{\partial L}{\partial w_i}}(\boldsymbol{W}^n))^2+\epsilon}}}$ 
( 此處的 $\epsilon$ 旨在不讓分母為 0 的情況產生，一般 $\epsilon=$ 10e-8 )

我們稍微調整一下，Adagrad 的參數優化方式可以這樣寫 :

$w_i^{t+1}\leftarrow w_i^t-\displaystyle{\frac{\eta}{\sigma^t}}g^t$ , whare $g^t=\displaystyle{\frac{\partial L}{\partial w_i}}(\boldsymbol{W^t})$ and $\sigma^t=\sqrt{\sum\limits_{n=1}^{t}(\displaystyle{\frac{\partial L}{\partial w_i}}(\boldsymbol{W}^n))^2+\epsilon}$

<font color="#dd0000">**參數建議 $\eta=0.01$**</font>

白話一點來說，$g^t$ 代表的是第 $t$ 次的梯度更新值，而 $\sigma^2$ 則代表的是第 $t$ 次以前的所有梯度更新值之平方和開根號。 

### Adagrad 的矛盾 ? 

上式中 $g^t$ 清楚地描繪了「當斜率越大，就必須要跨越大步」的這一個事實，但也別忽略了分母的 $\sigma^2$ 卻會造成相反的結論。

要了解這一個狀況是否會產生矛盾，我們要從斜率 (一次微分) 與跨多大步的關係來看 : 

![](https://i.imgur.com/e29ZoCj.png)

假定 Loss function 為一個二次函數，現有一點 $x_0$，從基本數學來看，最好的一步便是 $\mid x_0+\displaystyle{\frac{b}{2a}}\mid$ 。從這裡我們可以看出來，當一次微分值越大，表示 $x_0$ 距離最低點越遙遠，要跨得步伐便越大。

然而事情並沒有想像的這麼簡單，倘若在一個高維度空間下，光看一次微分是無法進行跨維度、跨參數的比較

![](https://i.imgur.com/lFmmFnb.png)

上圖中的 a 與 c 單從一次微分來看，無法進行比較。

從上上一張圖片中，我們其實忽略了 $\mid x_0+\displaystyle{\frac{b}{2a}}\mid=\displaystyle{\frac{\mid 2ax_0+b\mid}{2a}}$ 式中分母 $2a$ 其實就是 Loss function 的二次微分值，在單一維度中，這個常數項或許可以被忽略，但要進行跨參數的比較時，這樣一個數值便不可忽略。

倘若加入這一個分母進行討論，上圖 a 與 c 就可以進行比較了。

![](https://i.imgur.com/gL6dddh.png)

但在 Adagrad 中，為了不增加計算的負擔，我們更進一步的採用一次微分值來對二次微分值進行推估，不僅能達到相同的效果，也不用再一次計算二次微分值。

### Adagrad 的優缺點

#### 優點
1. 當 $t$ 持續增加，$\sigma$ 項會約束梯度，也就是說，Adagrad 可以自動調整 learning rate 直至收斂。
2. 適合處理稀疏梯度

#### 缺點
1. 當後期 $\sigma^t$ 值很大的時候，整個梯度會被約束到趨近於 0 ，導致訓練提前結束。
2. 仍然需要先設置一個全局學習率 $\eta$，且其大小仍然會影響訓練的過程。

## RMSprop --- 處理複雜 error surface

然而，我們現實中常會碰到的 Loss function 並非都是平穩、簡單的，甚至絕大多數我們遇到的 Error surface 都非常複雜。

![](https://i.imgur.com/5N3U3Uy.png)

如上圖，即使在同一個維度上，學習率都有可能必須要能夠快速的反應、變動，因此 Hinton 提出了一個新的優化方式 : RMSprop

RMSprop 在 學習率調整上面多了一個參數 $\alpha$ ，可以在新舊梯度上面做調節

$w_i^{t+1}\leftarrow w_i^t-\displaystyle{\frac{\eta}{\sigma^t}}g^t$ , whare $g^t=\displaystyle{\frac{\partial L}{\partial w_i}}(\boldsymbol{W^t})$ and $\sigma^t=\sqrt{\alpha(\sigma^{t-1})^2+(1-\alpha)(g^t)^2+\epsilon}$

<font color="#dd0000">**參數建議 $\eta=0.001$ , $\alpha=0.9$**</font>

若 $\alpha$ 上調，便對於舊的梯度有更大的佔比，也就是說在整個調節的過程中較傾向相信舊梯度帶給我們的資訊。

### RMSprop 的優缺點

#### 優點
1. 有效改善 Adagrad 提前結束訓練的問題。
2. 適合處理複雜的、non-convex 的 error surface。

#### 缺點
1. 仍然需要先設置一個全局學習率 $\eta$

## Momentum --- 跳脫出 Local minimum 的困境

在 Gradient Descent based algorithm 中，很容易會進入 Local minimum 中而跳脫不出來，雖然說有學者認為 Local minimum 在複雜多維度的 error space 中並不會這麼容易遇到，但 Momentum 或許也能為這個問題找出一個合適的處理方式。

![](https://i.imgur.com/V5wDXNv.png)


Momentum 是利用物理學中動量的概念來進行梯度更新 ( $\lambda$ 為動量因子 )

$v^0=0$
$w_i^{t+1}\leftarrow w_i^t+v^t$ , where $v^t=\lambda\cdot v^{t-1}-\eta\cdot g^t$ , and $g^t=\displaystyle{\frac{\partial L}{\partial w_i}}(\boldsymbol{W^t})$

<font color="#dd0000">**參數建議 $\lambda=0.9$**</font>



![](https://i.imgur.com/pY7ZrkX.png =400x)

(註 : 此圖中的 $\theta^t$ 即本文中 $\boldsymbol{W}^t$)


這樣的梯度更新包含了前次梯度的量值，也是在某種程度上面保留了原本的動能，如果遇到 local minimum 便有機會可以跳脫出來。

### Momentum 的優缺點

#### 優點

1. 當梯度更新時，$\lambda\cdot v^t$ 這項有助於減緩更新，可以抑制震盪，加快收斂。
2. 在初期，我們可以藉由較大的 $\lambda$ 來對整個訓練加速
3. 中後期由於梯度逐漸下降，因為我們有 $\lambda\cdot v^t$ 這項，可以使得擺動幅度加大，有助於跳脫出 Local minimum

#### 缺點

1. $\lambda$ 、$\eta$ 固定無法隨時調整 



## Adam --- 常用的 optimizer

Adaptive Moment Estimation ( Adam ) 其實就是加入了動量概念的 RMSprop，且在更新梯度過程中考慮了偏差校正 ( bias-correction )

$m^0=v^0=0$

$w_i^{t+1}\leftarrow w_i^t-\eta\cdot\displaystyle{\frac{\hat{m}^t}{\sqrt{\hat{v}^t}+\epsilon}}$ 

where $m^{t+1}=\beta_1\cdot m^{t}+(1-\beta_1)\cdot g^t$ , and $v^{t+1}=\beta_2\cdot v^{t}+(1-\beta_2)\cdot (g^t)^2$

and $\hat{m}^t=\displaystyle{\frac{m^t}{1-\beta_1}}$ , $\hat{v}^t=\displaystyle{\frac{v^t}{1-\beta_2}}$ , $g^t=\displaystyle{\frac{\partial L}{\partial w_i}}(\boldsymbol{W^t})$

<font color="#dd0000">**參數建議 $\beta=0.9$** , $\beta_2=0.999$</font>

上面的式子看起來有點恐怖，但其實仔細跟 RMSprop 比較一下，不管從更新方向或是更新步伐都帶入了 RMSprop 新舊權衡的概念，而其中參數 $\beta_1$ 及 $\beta_2$ 可以視為每一次更新後方向及不乏上的衰減率。

比較值得注意的是參數更新的部分是藉由 $\hat{m}^t$ / $\hat{v}^t$ 而非 $m^t$ / $v^t$ 來進行更新，$\hat{m}^t$ / $\hat{v}^t$ 可以視為是對 $m^t$ / $v^t$ 的偏差校正。

### Adam 的優缺點

#### 優點

1. 結合了 Adagrad、RMSprop 及 Momentum 的優點
2. 對內存的需求小
3. 對所有不同的參數都有新舊之間的權衡調節
4. 各種狀況均適用，是目前較為推薦的優化方式。




---

#### 參考內容

1. Hung-yi Lee ,  [Machine Learning](http://speech.ee.ntu.edu.tw/~tlkagk/courses_ML17_2.html)(2017) : Gradient Descent、Tips for training DNN
2. [深度学习最全优化方法总结比较（SGD，Adagrad，Adadelta，Adam，Adamax，Nadam）](https://zhuanlan.zhihu.com/p/22252270)
3. [深度学习——优化器算法Optimizer详解（BGD、SGD、MBGD、Momentum、NAG、Adagrad、Adadelta、RMSprop、Adam）](https://www.cnblogs.com/guoyaohua/p/8542554.html)
4. [深度学习笔记：优化方法总结(BGD,SGD,Momentum,AdaGrad,RMSProp,Adam)](https://blog.csdn.net/u014595019/article/details/52989301)

