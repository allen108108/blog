---
title: "[論文] YOLOv3 : An Incremental Improvement" 
date: 2020-02-15 08:45:35
categories:
- 論文 Paper
- 物件偵測 Object Detection
image: https://i.imgur.com/PDAaAGA.png
mathjax: true
---


這篇雖然說看似一篇論文，但整體讀下來比較像是一篇筆記 (作者也說這篇論文它定位為技術文件)，因此這一篇論文閱讀筆記我就以略讀的方式來撰寫，想留一點時間來做一下不同版本 YOLO 之間的比較。

<!-- more -->

摘要 Abstract
---

這篇論文主要針對 YOLOv2 做了一些小改良，讓整個模型變得更好。然而 YOLOv3 相對於 YOLOv2 來說，整個模型變得相對龐大，但是彌補了不少 YOLOv2 的問題，而且速度上仍然表現得相當不錯。


簡介 Introduce
---

( 略 )

更新 The Deal
---

### 邊界框的預測 Bounding Box Prediction



<img width=500 src="https://i.imgur.com/tsAKJ5N.png" >




上圖其實就是 YOLOv2 的邊界框預測方式，詳細可參閱 " [[論文] YOLO9000 : Better, Faster, Stronger](https://) " 一文。在訓練中，一樣使用平方和來計算誤差，如果真實座標值為 $\hat{t}_*$ 那麼其梯度值為 $\hat{t}_*-t_*$。 藉由上面的計算公式，我們可以輕易地推導出 $\hat{t}_*$。

$$
\hat{t}_x=\sigma^{-1}(x_{label}-c_x)\\
\hat{t}_y=\sigma^{-1}(y_{label}-c_y)\\
\hat{t}_w=\ln(\frac{w_{label}}{p_w})\\
\hat{t}_h=\ln(\frac{h_{label}}{p_h})\\
$$

在 YOLOv3 中，利用 logistic regression 來預測物件分數 ( object score )，也就是 YOLOv1 論文中說的信心指數 ( confidence ) : 

* 與真實邊界框之 $IOU$ 為最大值，且高於閾值 $0.5$ 之 anchor box 其 $object\ score =1$
* 與真實邊界框之 $IOU$ 非最大值，但仍高於閾值 $0.5$ 之 anchor box 則忽略其預測值。

與 Faster-RCNN 不同，YOLOv3 僅對每一個真實物件分配一個 anchor box，若沒有分配到 anchor box 的真實物件，便不會有座標誤差，僅會具有 object score 誤差。



### 分類預測 Class Prediction

在分類預測的部分，YOLOv3 使用的是 logistic 分類器，而不是之前使用的 softmax。主要是因為 softmax 在使用上有一個前提是類別間必須要是互斥的，也就是說每一個邊界框只能被預測出一個類別，但是在一些資料集中，類別之間並不一定滿足這樣的前提條件 ( 可能同時滿足「人」及「女人」的類別 ) 。

在 YOLOv3 的訓練中，便使用了 Binary Cross Entropy ( BCE, 二元交叉熵 ) 來進行類別預測。

### 多尺度預測 Prediction Across Scales

YOLOv3 中會預測三種不同尺度的邊界框。
整個系統會使用類似 Feature Pyramid Network ( FPN ) 的概念來對這些尺度中進行特徵萃取。在整個特徵萃取的系統中，YOLOv3 加進了一些卷積層，最後輸出一個 3D 張量，包含了邊界框預測、物件分數以及類別預測。

以 COCO 為訓練資料進行的實驗中，最後輸出的張量為 $N\times N\times [3\times (4+1+80)]$

![](https://i.imgur.com/PDAaAGA.png)

上圖為 YOLOv3 的整體架構，在整個基本的特徵萃取結構中用到了兩次的 upsampling，並且使用 ResNet 中 Shortcut 的概念將早期的特徵圖與 upsampling 的特徵圖進行結合。這樣可以將初期的粗粒度特徵與後期的細粒度特徵進行結合，讓整個特徵萃取可以抓到更全面的特徵。

後面再加入一些卷積層來處理組合後的特徵圖，最後輸出預測的張量。

從上述的過程可以發現，第三個尺度的預測直接受益於前面兩個尺度的預測及整個計算過程，以至於在最終這個預測可以提取到更精細的特徵。

在 Anchor Box 方面，一樣使用 k-means 來確定出 Anchor Box 的尺寸，在YOLOv3 上利用 COCO 資料及所選出的 Anchor Boxes 尺寸為 

$$
(10\times 13),(16\times 30),(33\times 23),(30\times 61),(62\times 45),\\(59\times 119),(116\times 90),(156\times 198),(373\times 326)
$$


### 特徵萃取器 Feature Extractor

整個特徵萃取器的骨幹 (backbone) 其實就是由 YOLOv2 所提出的 Darknet-19 與 ResNet 的混合方式，稱之為 Darknet-53。


<img width=500 src="https://i.imgur.com/58lrJli.png" >




Darknet-53 相較於其他架構 ( Darknet-19, ResNet-101, ResNet-152 ) 來說，準確度高而且速度上大為提升。Darknet-53 在運算上使用了更高效率的浮點運算，這表示整個結構在 GPU 上可以進行更高效率的評估。


### 訓練 Training

( 略 )

作法 How We Do
---

![](https://i.imgur.com/qv1xVvc.png)

上表顯示，以一般的 $AP$ 指標來看，YOLOv3 的表現與 SSD 的變體 DSSD513 並駕齊驅。若以 $AP_{50}$ ($IOU$ 閾值為 $0.5$) 來看，則 YOLOv3 甚至表現接近 RetinaNet，但上表沒有說的事情是不管是 RetinaNet 還是 DSSD ，YOLOv3 的速度都快上 3 倍以上。然而以 $AP_{75}$ 來說，YOLOv3 表現反而下滑。

且在小物件的偵測上面，YOLOv3 的表現提升不少。

這樣的訊息透露幾件事情 ：

1. YOLOv3 擅於偵測出「合適」，但無法偵測出非常精準的邊界框。
2. YOLOv3 小物件偵測能力提升，但中大型物件的偵測反而相對較差。(原因不明 ？)
3. 若將速度考量進來，YOLOv3 整體來說表現非常出色。


![](https://i.imgur.com/zQkkUJB.png)



無用的嘗試 Things We Tried That Didn't Work
---

都說了這是一篇技術文件，作者也將這段時間嘗試的一些徒勞無功的做法記錄下來。


### Anchor Box $x,y$ 偏移預測

將 $x,y$ 的偏移量假設為 $w$ 或 $h$ 的倍數，這樣的嘗試導致模型的不穩定，而且表現不是很理想。

### 取代 logistic 以線性預測 $x,y$

將 activation function 從本來的 logistic 改為 linear，一樣導致 $mAP$ 下滑。

### Focal Loss 的使用

這個嘗試比較有趣，Focal Loss 出現在論文 "Focal Loss for Dense Object Detection" 中，上面提到的 RetinaNet 也是這篇論文中的一個中間產物，但重點還是放在 Focal Loss 上。簡單來說，Focal Loss 就是用來解決物件偵測中，背景比例極高的狀況。

YOLO 其實利用獨立的 object score 以及 class prediction 解決了 Focal Loss 要解決的問題，然而使用 Focal Loss 後 YOLOv3 卻有兩個百分點的 $mAP$ 下降，為什麼會這樣，作者也不太確定。( 詳細討論可以閱讀一下 " [知乎 － 为什么 YOLOv3 用了 Focal Loss 后 mAP 反而掉了？](https://www.zhihu.com/question/293369755) " 一文 )


### 使用雙 $IOU$ 閾值，並分配正負樣本

在 Faster-RCNN 中使用了兩個閾值來進行訓練。當預測的邊界框 $IOU$ 大於 $0.7$ 則將此視為正樣本，介於 $0.3-0.7$ 之間則忽略，小於 $0.3$ 便視為負樣本。如將 YOLOv3 利用類似的方法進行訓練，則無法得到好的結果。


意義 What This All Means
---

YOLOv3 是一個快速且準確的物件偵測系統，即使在閾值 $0.5-0.95$ 之間表現得差強人意，但在閾值 $0.5$ 的狀況下，它表現得非常好。

在 COCO 論文中追求高閾值的表現，對 YOLO 作者來說覺得並不是太有意義。事實上，人類無法用肉眼精準看出 $IOU$ 值在 $0.3-0.5$ 之間的差異，既然如此，只要有一定的精準度，定位精度並不需要追求這麼高標準。

反倒是作者提出了另外一個問題 ( 這也是現今 AI 技術一直為人詬病的部分  )：「 我們有了這些優良的偵測系統，我們該拿它來做什麼事情 ？」

作者希望，應用電腦視覺技術的人們，可以將其用在好的、對的事情上面，我們有責任去考量到我們的工作可能對社會造成的危害。

> We have a responsibility to at least consider the harm our work might be doing and think of ways to mit- igate it. We owe the world that much.


反駁 Rebuttal
---

我覺得比較有趣的還是針對 COCO mAP 的部分。

到底一直追求定位高精準度合理嗎？從人眼對於物件偵測的過程中，不是更應該著重在分類的準確度上嗎 ？

論文中提出了下圖，對於這兩個物件偵測器來說，都是非常優秀的，但是下方的偵測器卻著實不符合人類對於偵測的直覺印象。

![](https://i.imgur.com/0Xli88H.jpg)



參考資料 Reference
---

1. [StackExchange-Yolo v3 loss function](https://stats.stackexchange.com/questions/373266/yolo-v3-loss-function)
2. [YOLO v3: You Only Look Once (version 3)](https://zhuanlan.zhihu.com/p/93273011)
3. [A Closer Look at YOLOv3](https://www.cyberailab.com/home/a-closer-look-at-yolov3)
 