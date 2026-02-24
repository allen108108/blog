---
title: "手部關鍵點 (Hand Keypoints) 的預測及其應用"
date: 2020-06-06 10:46:39
categories:
- 專案 Project
image: https://i.imgur.com/OUbhPM3.jpg
mathjax: true
---


前言
---

手部關鍵點的預測，在整個深度學習以及電腦視覺領域中並不算是一個非常受歡迎、研究廣泛的領域，但是在許多虛擬操作上，預測手部關鍵點這件事情便顯得至關重要。要怎麼利用手勢來做操作 ? 或許我們可以利用現有的 Object Detection 技術來針對手勢定義相對應的操作，這樣的方式既直覺也有相對來說成熟的模型可以使用。有興趣的可以閱讀筆者之前的文章 " *[Gesture Recognition using YOLO v3 tiny](https://allen108108.github.io/blog/2020/01/18/Gesture%20Recognition%20using%20YOLO%20v3%20tiny/)* "。

<!-- more -->

![](https://i.imgur.com/jOv86Wr.jpg)

這樣的方法有著極大的問題，不管今天要增加新的手勢或想改用動態手勢來進行操作，都必須重新進行訓練，更甚至得設計新的網路架構才能達到我們想要的功能，這對於實際在應用上有著非常大的限制，因為每做一次更動都得消耗大量的時間及人力成本。

然而，如果我們可以訓練出一個模型準確且即時的預測出手部關鍵點的位置，那麼我們便可以利用這樣的結果來定義動、靜態手勢，儲存在設定檔中，日後要增加或更改手勢也僅需要更改設定檔即可，可以省去非常多的成本也有利於後續的應用。


Method
---

現在針對手部關鍵點的研究方向大多分為兩種，一種是類似 Object Detection 利用 Anchor Box 先進行手部位置的預測，之後再進行關鍵點的預測。另外一種則是 Anchor Free 的方法，對輸入圖像進行關鍵點的熱圖預測。

### Anchor Based


Anchor base model 中比較著名的方法便是 Google 所開發的 Mediapipe ，在手部關鍵點辨識中，首先利用一個手掌偵測器 Blaze Palm 進行手掌的偵測，僅偵測手掌而非整個手部的原因在於，手掌之於手部動作來說是相對固定的，就連 anchor 上的設定都不需要太多種長寬的組合，僅需要正方形 Anchor 即可，這樣的設計對於整個系統來說是非常節省運算成本的一種作法。

![](https://i.imgur.com/XvTv9CU.png)

手掌偵測完後，再針對 Bounding Box 的長寬做某個倍數的放大來做為整個手部的 Bounding Box。

至於最重要的手部關鍵點則是使用回歸的方式進行預測，也就是讓模型直接預測出 21 個點座標，42 個數字。

![](https://i.imgur.com/SxB50Lz.png)


### Anchor Free

之前筆者曾經寫過一篇論文速讀文章 " *[[論文] 速讀論文 Accurate Hand Keypoint Localization on Mobile Devices](https://allen108108.github.io/blog/2020/01/11/[%E8%AB%96%E6%96%87]%20%E9%80%9F%E8%AE%80%E8%AB%96%E6%96%87%20Accurate%20Hand%20Keypoint%20Localization%20on%20Mobile%20Devices/)* " ，這篇論文介紹的就是 Anchor Free 的關鍵點預測，Anchor Free based 的方法大多都是利用 CNN 加上回歸來對每一個關鍵點輸出 Heatmap。

![](https://i.imgur.com/Ap3Oo8Y.png)

這種方法在現今關鍵點的預測上應該可以算是主流的方式，著名的 OpenPose 辨是基於這樣的概念加上 Bootstraping 來預測身體及手腳等部位的關鍵點，在 Marcelo Ortega 的部落格文章 " *[Training a Hand Detector like the OpenPose one in Tensorflow](https://medium.com/@apofeniaco/training-a-hand-detector-like-the-openpose-one-in-tensorflow-45c5177d6679)* " 中也提到了類似的作法。

![](https://i.imgur.com/wkncp3Y.png)


Datasets
---

筆者在此專案中，使用的是由德國佛萊堡大學 (University of Freiburg) 開發的 FreiHAND Dataset ( arxiv : https://arxiv.org/pdf/1909.04349.pdf )，這個資料集中利用下方的設備進行手部圖片的收集，其中包含了八個校準過後的 RGB 相機，而綠幕背景可以方便後續進行背景的替換。

![](https://i.imgur.com/tLCOw9A.png)

該資料集中包含了綠幕背景原始手部圖片共 32560 張，及利用這些原始圖片進行背景填補還有色調變化的延伸照片共 97680 張。


![](https://i.imgur.com/ocoPQXh.jpg)

有鑒於此資料集在收集上面的一些限制 ( 如 : 手部位置固定、色彩對比差異不大...等 )，因此筆者利用這個資料集做了一些資料增強 (Data Augmentation) ，將原本的資料平移、翻轉以及 HSV 上面做一些調整

![](https://i.imgur.com/la4l2d0.jpg)

以上資料進行比例上的分配後做為訓練資料進行訓練。

Model Structure
---

筆者不對整張圖像先做 Hand Detector，希望利用整張圖像直接進行手部關鍵點的預測，如果我們可以在這樣的條件下準確的預測關鍵點，那我們就可以自行選擇加上或不加上另外一個模型偵測手部位置。

在模型的架構上，使用 ResNet50 (`include top = False`) 作為模型骨幹，加上一層 $1\times 1$ 的卷積層，一方面用來融合各 channel 的資訊，另一方面則是降低維度。

```python=
base =ResNet50(weights=None, include_top=False, input_shape=input_shape)
model = Sequential()
model.add(base)
model.add(Conv2D(filters=64, kernel_size=1, activation='relu'))
```

實際比較過後，加了這一層 $1\times 1$ 的卷積層 ，模型尺寸從原先的 600 MB 降至 270 MB ，而且推論的能力也提高不少。


```python=
model.compile(loss='mean_squared_error', optimizer='adam', metrics=['accuracy'])
```

由於整個關鍵點的預測是利用回歸方式來進行，所以模型會輸出 42 個數字，即 21 個點的座標，然後直接與標註的 21 點座標進行 MSE 作為損失函數。

這樣的模型結構是否是最佳的結構，筆者仍在嘗試各種可能的結構，但這樣的簡單結構就可以有每種程度上的準確度存在。


Training and Inference
---

訓練過程中，筆者並沒有使用到什麼 Regularizer 的技巧，原因在於這樣的一個任務中，訓練幾乎都是 underfitting 的，出現 overfitting 的狀況並不多，一來是因為手部關鍵點的任務相對於一些分類、或是臉部特徵點的預測都來的困難許多，再者，筆者目前使用的模型都還算是簡單的結構，對於這樣的任務來說都還不夠 robust，當然也就暫時不用使用 Regularizer 技巧來限制模型的能力了。

在 Learning Rate 上面，筆者設計當 Validation Loss 沒有顯著下降時，LR 會自動減半，直至 `1e-7` 為止便不再下降，這樣的LR 動態設計可以使 LR 在不同的訓練狀況下適當作出調整。

硬體方面，使用 GTX 2080 訓練約一天的時間，Training Loss 與 Validation Loss 均收斂在 $9$ 左右，而 Accuracy 則大約落在 $0.73$。就姿態估計的模型來看，這樣的模型表現的確不是非常好，在未來的工作上仍需要持續修正模型架構來得到更好的表現。

筆者利用 Webcam 進行 Real-Time Inference 可以得到下列的結果 : 

{%youtube Cju8uPcCHBs %}

在這樣的推論結果，筆者對於目前這個模型有幾個結論 : 

1. Inference Time 不差，但由於模型太大，載入模型需花費相對長的時間。
2. 張手的預測結果不錯，但當手部有較多遮擋的姿態時，預測效果便會下降不少。
3. 手部的位置亦會影響推論結果，在目前的這個模型中，可以發現尤其當手部位於 Frame 的左側邊緣時，整個關鍵點的預測會出現問題，另外，手部於畫面占比太大或太小也會降低預測正確性。
4. 關鍵點的預測會因為背景環境而受到不少干擾，如果手部在較為簡單、顏色較為單純的背景下，預測結果也會相對來的好。


Result
---

從上面的結果，我們大概可以從兩個面向來討論 :『資料集』 與 『模型』。

資料集的部分，我們使用的是公開的資料集，因此在真正的使用上會導致模型容易受到資料集本上的分布影響其推論能力，舉例來說 ， FreiHAND Dataset 的手部與整個畫面的占比幾乎是固定，因此在真正的推論中，太近或太遠都會使預測準確度下降，除此之外，FreiHAND 的解析度也是一個大問題，筆者認為，利用低解析度的資料集訓練出來的模型，在高解析度圖像上的推論準確程度會造成一定程度的下降。

從模型的角度來看，先前本文有提到，手部關鍵點的預測本來困難度就非常高，因此目前就現有的 keras 預訓練模型，都有可能無法達到接近於 Mediapipe 的推論結果，因此在整個模型的結構上，該怎麼兼顧模型尺寸以及推論能力，這是需要花一些功夫去找出來的。

Application
---

假如，我們可以準確地找出手部關鍵點的位置可以做哪些事情呢 ? 在這個段落，筆者嘗試在 Mediapipe 的模型基礎上做出三種不同的應用給大家參考 :

### 1. 手勢辨識

下圖是筆者模擬 Mediapipe 部落格文章中提到的，利用關節角度來進行手勢的定義。當然方式有很多種，筆者也嘗試用關鍵點的相對位置來進行手勢定義，也是可行的唷 !

![](https://i.imgur.com/OUbhPM3.jpg)

### 2. 動態手勢

除了靜態的手勢外，關鍵點也可以用來進行動態手勢的辨識，以往動態手勢的模型通常都會使用帶有 LSTM 的 CNN Based 模型下去做訓練，跟靜態手勢一樣，如果我們要定義新的手勢，整個模型就必須重新訓練一次。

![](https://i.imgur.com/PMtT6WL.gif)


### 3. 手勢操作

現在科技的進步，我們多少都會幻想自己能跟鋼鐵人 Tony Stark 一樣來進行虛擬的手勢操作，事實上，經由 AR 或 VR 眼鏡我們已經可以做到類似的動作，當然，我也就可以利用手部關鍵點來控制滑鼠的鼠標以及進行滑鼠左鍵的操作 ( 點擊、拖曳...等動作 )

![](https://i.imgur.com/9Rsfytl.png)

筆者將掌心定義成為鼠標位置，而利用食姆指的捏合模擬滑鼠左鍵的動作，當食姆指碰在一起的時候，左上角的 `State` 便會顯示 `Press` 的字樣，表示現在的動作就等同於按下滑鼠左鍵，而同時會將掌心的座標記錄成 `Start point` 起點，當鼠標移動到期望的地方，放開食姆指，便會顯示 `End point` 的座標，這也可以知道這段過程中鼠標移動的距離為何，當然也可以記錄成拖曳的軌跡。

![](https://i.imgur.com/DyG8v49.gif)



Extended Question : Dataset Quality ?
---

從前面的討論，或是過去專案設計的經驗，模型對於資料集的依賴程度相當高，當我們在量化評估模型網路結構時，很多時候不自覺的會建立在資料集本身沒有問題的前提下進行。但是，當我們現在要進行的是一個特別的專案，必須使用到 customize 的資料集時，我們可以怎麼衡量這個資料集的 Quality ? 如果我們必須要自己收集資料，怎麼樣可以確定這個資料的分布接近於我們的需求 ? 

換另外一個面向來討論這個問題，如果當我們訓練出來的模型其推論能力不如預期，我要如何確定，問題是出在資料集還是模型本身 ? 

這個問題筆者詢問過相關的同好，也似乎沒有一個肯定的答案，或許，許多自行設計的專案中，表現不佳的根本問題就是出在資料而不是模型本身 ?



Reference
---

1. [On-Device, Real-Time Hand Tracking with MediaPipe.](https://ai.googleblog.com/2019/08/on-device-real-time-hand-tracking-with.html)
2. [[論文] 速讀論文 Accurate Hand Keypoint Localization on Mobile Devices](https://allen108108.github.io/blog/2020/01/11/[%E8%AB%96%E6%96%87]%20%E9%80%9F%E8%AE%80%E8%AB%96%E6%96%87%20Accurate%20Hand%20Keypoint%20Localization%20on%20Mobile%20Devices/)
3. [Training a Hand Detector like the OpenPose one in Tensorflow](https://medium.com/@apofeniaco/training-a-hand-detector-like-the-openpose-one-in-tensorflow-45c5177d6679)
4. [susantabiswas/facial-keypoint-regression](https://github.com/susantabiswas/facial-keypoint-regression)