---
title: "[論文] 速讀論文 Accurate Hand Keypoint Localization on Mobile Devices" 
date: 2020-01-11 14:55:57
categories:
- 論文 Paper
image: https://i.imgur.com/v2IeruH.jpg 
mathjax: true
---

論文中提出了一個網路結構，可以將一般圖片 ( 未經過處理 ) 輸入後，直接進行手部關鍵點的定位。此方法利用改良過的 VNect[^註1] 來對輸入圖像計算其 Heatmap，再利用此 Heatmap 來進行關鍵點的定位。

<!-- more -->

這樣的方法在一些 Open dataset 上取得了優異的成果，甚至可優於現今 SOTA 的方法，這也使得這樣的結構可以成為移動設備上用來定位關鍵點的重要部件。


![](https://i.imgur.com/yRGgQ1I.jpg)



VNect
---

利用 RGB Camera 進行 3D 姿勢辨識一直以來都是一個極具挑戰性的問題，在 VNect 中利用了兩個部份來進行即時的 3D 姿勢預測 :
* CNN 回歸預測 ( CNN Pose Regression ) : 以 CNN 方法來將生成 2D Heatmap 以及 3D Location map。[^註2][^註3]
* 骨架擬合 ( Kinematic Skeleton Fitting ) : 利用 Model-based kinematic skeleton fitting 來修正 CNN 迴歸分析後的關鍵點座標，來確保運動的一致性。

網路結構如下 : 

![](https://i.imgur.com/SHrtVls.png)

將 ResNet50 在 4f 層的輸出接上上述結構，最後輸出 Heatmap 以及 X,Y,Z 的 Location maps。



改良版 VNect
---

在此論文中，目標是在移動設備上可以進行即時的關鍵點預測，因此，必須將 VNect 的結構進行修改，讓其更輕量卻不失準確度。

修改的方向如下 : 

1. 將 ResNet 改為輕量級的 MobileNet v2
2. 原本的 Convolution Layer 改為 Depthwise Convolution Layer[^註4]

![](https://i.imgur.com/v2IeruH.jpg)

這也就是此論文主要使用的網路結構，這樣的結構生成出手部關鍵點 Heatmap $H(p)$，之後會利用 MLE 的方式來找出關鍵點座標 $\bar{k}$

$$
\bar{k}=\arg\max_{p\in P}H(p)
$$






註釋
---

[^註1]: 
Dushyant Mehta et al. *"Vnect: Real-time 3d human pose estimation with a single rgb camera"*. In: ACM Trans. on Graphics (TOG) 36.4 (2017).

[^註2]: 
為什麼在現今的姿勢預測中，不直接針對關鍵點進行回歸預測即可 ? 除了省去 Bounding Box 的預測計算成本外，使用 Heatmap 來進行關鍵點預測還有許多的優點，詳細說明可參閱知乎 : 《*[关键点检测中，为什么要生成高斯图，而不是直接与groundtruth比较？](https://www.zhihu.com/question/293815527)* 》

[^註3]: 
CSDN : [论文阅读《*Flowing ConvNets for Human Pose Estimation in Videos*》](https://blog.csdn.net/qq_36165459/article/details/78320781)

[^註4]: 
Convolution 與 Depthwise convolution 的差異可以參閱 : [深度學習-MobileNet (Depthwise separable convolution)](https://medium.com/@chih.sheng.huang821/%E6%B7%B1%E5%BA%A6%E5%AD%B8%E7%BF%92-mobilenet-depthwise-separable-convolution-f1ed016b3467)
