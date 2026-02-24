---
title: "Regularization 方法 : Weight Decay , Early Stopping and Dropout"
date: 2019-10-07 02:59:16
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning
image: https://i.imgur.com/jWfhiJl.png
mathjax: true
---

>* 本文內容節錄自Hung-yi Lee ,  [Machine Learning](http://speech.ee.ntu.edu.tw/~tlkagk/courses_ML17_2.html)(2017) 課程內容 : Tips for training DNN
>* 本文圖片均來自於課程講義內容
>
<!-- more -->
## Weight-Decay

Weight-Decay 其實就是我們最常說的 L1 / L2 regularization，但是為何加入了 regularizer 就會造成 weight decay ?

我們拿 L2 regularization 為例 

$w_i^{t+1}\longleftarrow w_i^t-\eta\cdot \displaystyle{\frac{\partial L'}{\partial w_i}}$，where $L'(\boldsymbol{W})=L(\boldsymbol{W})+\lambda\cdot\sum\limits_{n=1}^{N}(w_n^t)^2$

$\Longrightarrow w_i^{t+1}\longleftarrow w_i^t-\eta\cdot(\displaystyle{\frac{\partial L}{\partial w_i}}+2\lambda w_i^t)$

$\Longrightarrow w_i^{t+1}\longleftarrow (1-2\eta\lambda)w_i^t-\eta\cdot\displaystyle{\frac{\partial L}{\partial w_i}}$

從右邊第一項我們可以知道，進行參數更新時我們必然每一次都會將參數先乘上 $(1-2\eta\lambda)$ 這一項，這會導致參數會遞減並往0靠近 ( 因為 $1-2\eta\lambda<1$ )，這就是 weight decay 的精神所在。

### Weight-Decay 的缺點

Weight decay 在實務上並不常用，原因是他的整體效果其實並沒有辦法比類似 Early-Stopping 這樣的 regularization 來的優秀，再加上必須另外添加 regularizer ，又不像 SVM 將 regularizer 直接寫進演算法內部。



## Early Stopping


![](https://i.imgur.com/5UVk0lX.png)

在講到 overfitting 的時候我們很常會看到這樣的圖，從這樣的曲線圖我們可以知道，在訓練的過程中，最佳的時間點就是在 $d_{vc}^*$ 這裡停止訓練，就可以得到一個最佳的 model。

然而我們怎麼知道 $d_{vc}^*$ 會在哪個時間點或是哪個 iteration 次數出現 ? 這就可以利用 Validation set 來做驗證。



## Dropout

這是一個在 Deep Learning 中獨特的 regularization 方式，在每一層中隨機停掉一定 %數的 neurons 來進行訓練

![](https://i.imgur.com/jWfhiJl.png)

我們隨機停掉 neurons 換一個角度來說就是有很多多樣化的 neural network ，而將這些多樣化的 model 最後再統整起來 (Ensemble) 取得平均。

![](https://i.imgur.com/b0affPm.png)

<font color="#8e8e8e">但這裡有一個數學上的問題存在，這些 neural networks 利用 $p\%$ 訓練所得到的 average weights 最後要拿到 test data 中，就必須要將 training 所得 weights 乘以 $1-p\%$ 才會是我們比較希望得到的結果。

會有這樣的問題，直觀的來看，當我們進行 $p\%$ dropout 得到的權重，直接拿到沒有 dropout 的 test data 中，有可能會有 $\displaystyle{\frac{1}{p\%}}$ 倍數上的差異

![](https://i.imgur.com/Er4Wr15.png)

因此我們必須要將這些考量進去才能在 test data 上有合理的結果出現。</font>







( 不過寫這段是僅供參考，現在的 toolkit 已經將這些轉換都寫進 dropout code 中 )
