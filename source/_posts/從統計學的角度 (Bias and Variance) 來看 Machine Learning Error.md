---
title: 從統計學的角度 (Bias and Variance) 來看 Machine Learning Error
date: 2019-10-05 01:09:57
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning
image: https://i.imgur.com/WIlYiAL.png
mathjax: true
---


>* 本文內容節錄自Hung-yi Lee ,  [Machine Learning](http://speech.ee.ntu.edu.tw/~tlkagk/courses_ML17_2.html)(2017) 課程內容 : Where does the error come from?
>* 本文圖片均來自於課程講義內容
>

<!-- more -->

在進行林軒田老師的課程時，有簡略的介紹 error 是來自於現實結果與預測結果的差異，但從李宏毅老師的課程中，我們可以清楚了解到 error 與統計學中 Bias(偏差)、Variance(變異數)的關係，從這些關係中清楚的定義 underfitting & Overfitting，進而可以針對模型的 error 成因來對 model 進行適當的調整及改善。

## Bias & Variance

### Bias
一般在統計學中，我們就是以樣本的統計量來對母體進行推估，而 Bias 便是度量我們樣本統計量與母體的差異程度。


<img width=350 src="https://i.imgur.com/V94Axkb.png" >
<img width=100 src="https://i.imgur.com/TRKqon9.png" >


以上圖為例，正常我們取樣的樣本平均值 $m$ 很大的機率不會等於母體平均值 $\mu$，如果我們對母體做了很多次的抽樣，其期望值 $E(m)=\mu$ 則我們稱這樣的抽樣是 'unbiased' (無偏誤的)。

我們由此可以有一個結論，越好的樣本，其 bias 越低，也就是離目標越近。從機器學習的角度來看，我們對於目標通常會選擇一個模型來進行訓練，針對此模型我們手上會有一整個 model set ( 從林軒田的角度來說就是 hypothesis set )，這一個 model set 的 bias 便是 error 的來源之一。

### Variance
Variance (變異數) 在統計學來說便是一個度量樣本分散程度的統計量


<img width=350 src="https://i.imgur.com/lYLJKn9.png" >
<img width=200 src="https://i.imgur.com/S4NV4vi.png" >


當我們取樣的數量越多，分散程度便會縮小，可以更接近我們的目標，從機器學習的角度來看，當我們的模型選擇越複雜，整個 model set ( Hypothesis set ) 會有越多種各式各樣的選擇，其 variance 也會越高。

總結來說，我們可以用箭靶來描述 Bias & Variance


![](https://i.imgur.com/WIlYiAL.png)
<img width=350 src="https://i.imgur.com/WOArL0H.png" >


## Model V.S. Error V.S. bias and variance

從上面針對 bias 與 variance 的介紹，其實我們可以說機器學習裡面的 error其實就來自於 bias 與 variance。說了這麼多，我們最終還是想知道，到底 model , error , bias and variance 的關係是什麼 ?


### 當我們選擇越複雜的模型，variance 越高
![](https://i.imgur.com/MS8mWxT.png)


### 當我們選擇越複雜的模型，bias 越低
![](https://i.imgur.com/zWgARjS.png)

但是當我們選擇越複雜的模型，整個 variance 產生的 error 上升的速度遠高於 bias 產生的 error 下降速度，這也是為何當我們選擇越複雜的模型時，整個 test error 會飆得非常高的原因

![](https://i.imgur.com/pPcbBNu.png)

由此我們也可以給 underfitting 與 overfitting 一個比較清楚的定義 : 
* Underfitting : Large bias and Small variance
* Overfitting : Small bias and Large variance

而我們在模型的選擇上便是期望在 bias 與 variance 中找到一個平衡點。

## Model Selection

### 當 bias 偏高時，我們可以怎麼做?
* 加入更多 features
* 選擇更複雜的模型

### 當 variance 偏高時，我們可以怎麼做?
* 加入更多資料進行訓練
* Regularization

如果我們只簡單將手中資料切分成 training data 與 testing data，
利用 training data 訓練出來的模型再根據 testing data 所產生的總 error 來進行 model selection 時，testing data 本身的 bias，會使選出的 model 放進 未知的 testing data 測試時，勢必會比我們在已知的 testing data 所計算出來的 error 還要高。

在實務上我們很常會給一個目標誤差值，希望最後得到的 model 可以低於我們給定的目標誤差，但若按照上述的方式，雖然所選出的model在手邊的 testing data上可以達到我們要的目標，但在未知資料上的表現並不見得會低於我們的目標。

![](https://i.imgur.com/K11dPAN.png)

因此我們實務上會將 training data 再切分出一個 validation data ，利用 validation data 來進行 model selection，如此一來，已知的 testing data 與 未知的 testing data 之間就不存在 bias，也就是說，我們利用 validation data 所選出來的 model 應用在 已知的 testing data 上面的效果將會跟未知 testing data 效果一樣。

![](https://i.imgur.com/HViuH6z.png)

> [ Remark ]
> 
> 1. 千萬不要將 model 應用在手邊的 testing data 後又回頭對 model 進行調整，你會將手邊 testing data 的 bias 考慮進去，導致訓練出來的模型應用於未知的 testing data 再度產生落差。
> 
>2. 為了避免 Validation data 與 整個原先的 Training data 有高 bias，我們會使用 N-fold cross validation，來平均因 validation data 的 bias 所造成的 error 差異。
