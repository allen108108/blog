---
title: "ShuffleNet V2 於 MNIST 上之實作"
date: 2019-10-08 01:10:58
categories:
- 實作 Implementation
image: https://i.imgur.com/Urwinee.png
mathjax: true
---

前陣子在微博的某一個公眾號看到了一篇文章 : [ShufflenetV2_高效网络的4条实用准则](https://zhuanlan.zhihu.com/p/42288448)

文章主要是針對於有限計算能力的限制下，如何設計高效率的模型來達到最好的精準度，文章中一句話清楚的敘述現在許多論文的研究方向 : 「輕量級架構設計與速度 -- 精度的權衡」。

<!-- more -->

而文章內容主要在介紹 2018 年發表的論文 : *ShuffleNet V2: Practical Guidelines for Efficient CNN Architecture Design*，論文中針對 2017 年發表的 ShuffleNet 進行改良升級，在論文中顯示出 ShuffleNet V2 在相同的複雜度下，準確度可以比 ShuffleNet 和 MobileNetv2 更高。



論文中針對計算能力及速度利用了幾個指標來衡量 --- FLOPs 及 MAC (Memory Access Cost) --- 在論文中綜合評量發現 ShuffleNet V2 的表現優於 MobileNet、SHuffleNet、DenseNet 以及 Xception，本文不多加詳述，有興趣者可以參考論文內容。



![](https://i.imgur.com/RKtWUEw.png)



ShuffleNet V2 架構
---

根據論文作者結合理論與實驗得出了高效率模型設計的四項準則 : 
1. 相同的 channel 下，最小化記憶體訪問成本 MAC
2. 太多的 Group Convolution[^註1] 會增加 MAC
3. 模型內部太多零碎操作[^註2]容易降低平行運算的能力
4. Element-Wise operation[^註3] 不可忽略


在這樣的概念下，論文作者分析了 ShuffleNet 的缺失並加以改進而設計了 ShuffleNet V2 ，兩者的對比如下 : 

![](https://i.imgur.com/Urwinee.png)

( a ) the basic ShuffleNet unit
( b ) the ShuffleNet unit for spatial down sampling (2X)
(GConv : group convolution.)
( c ) the basic ShuffleNet V2 unit
( d ) the ShuffleNet V2 unit for spatial down sampling (2X) 
(DWConv : depthwise convolution.)


ShuffleNet V2 於 keras 上的實現
---

在 Github 上已經有人將 ShuffleNet V2 實現在 keras 上，詳情可參考 : 

https://github.com/opconty/keras-shufflenetV2

先前於 " [卷積神經網路 Convolutional Neural Network ( CNN ) 與 全連接神經網路 Fully Connected Feedforward Network 於 MNIST 上之實作](http://bit.ly/2oZH5f6) " 一文中我們比較了 CNN 與 Fully Connected Feedforward 的辨識能力比較，這次我仍然使用 MNIST 為資料集，配合上 ShuffleNet V2 進行 100 epochs 的訓練。



比較讓人意外的是，最後在 test data 上的成果竟然不比一個單純 CNN model 來的好。

CNN : 
![](https://i.imgur.com/55tqS66.png)
ShuffleNer V2 : 
![](https://i.imgur.com/a9CkuYL.png)

剛好最近在逛 reddit ML 版，便想順便問了一下這樣的情況是否正常 
所得到的回覆其實有點出乎我意料

![](https://i.imgur.com/hYlPI5c.jpg)

大致上的回覆都差不多，均指出 MNIST 這樣的資料集太過簡單，可以用來檢測自己的模型 how bad ，但若要拿來檢驗 how good 可能就並不適用，或許使用 CIFAR-10 或 CIFAR-100 可以比較明確比較租模型的優劣。

註釋
---

[^註1]: Group Convolution 最早於 AlexNet 中被提出，處理模型在雙 GPU 上的訓練。

[^註2]: 這裡所指的零碎操作指的是再多分支架構下，每一個分支底下的卷積層或池化層。

[^註3]: Element-Wise operation 指的是 ReLU、AddTensor、AddBias...等等。