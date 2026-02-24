---
title: Batch Size
date: 2019-10-05 01:43:38
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning
image: https://i.imgur.com/eWIcRB5.png
mathjax: true
---

我們進行 Gradient Descent 時，使用 Stochastic Gradient Descent 的確會讓整個訓練的速度變得比較快速，而且擬合的狀況並不會太差，但在實務上我們更常使用的方式是利用 mini batch 來進行訓練。

<!-- more -->


![](https://i.imgur.com/eWIcRB5.png)

如上圖，我們在使用 keras 進行 model fit 時便會要求輸入 `batch_size` 以及 `epochs` 兩個參數來確認我們要用什麼樣子的資料進行練，又要訓練多少輪次。

就上圖的例子來看，當我們更新一次參數後，會隨機選取一個 batch 計算這個 batch 的 total loss，不再是像以往一樣計算整個 dataset 的 total loss，接著再利用計算出來的 batch total loss 進行一次參數更新，然後再利用另外一個隨機選取的 batch 計算 loss......重複至所有 batch 都輪完一遍，這便是一個 epoch。

```
mini_batch= 1  .......  Stochastic Gradient Descent
mini_batch= number of data  ........  Gradient Descent
```

然而，絕大多數人會有一個疑問是，不管我們取 mini_batch 等於多少，計算 loss 的資料數量不是還是一樣嗎 ? 

![](https://i.imgur.com/p4RptZD.png)


的確，計算的資料總量的確相同，但在實務上操作的速度就是會不一樣，主要發生這種差異的原因是在於我們 <font color="#dd0000">**利用 GPU 進行平行矩陣運算**</font>

利用 GPU 進行平行運算，較大的 batch size 便會有較快的運算速度。

![](https://i.imgur.com/BTPs6j3.png)


既然這樣，那我們為何不就直接把 mini batch 數量設為所有資料的數量就好了 ?

因為要考量到 GPU 平行運算的能力，GPU也是會有上限的，如果我們資料量一大，這樣做就有可能會讓整個訓練 train 不起來。