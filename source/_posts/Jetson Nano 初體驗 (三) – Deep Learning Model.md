---
title: "Jetson Nano 初體驗 (三) -- Deep Learning Model"
date: 2020-04-16 18:50:55
categories:
- 創客 Maker
image: https://i.imgur.com/iCkTj5Q.jpg
mathjax: true
---




系列文章
---

* [*Jetson Nano 初體驗 (一) -- WaveShare JetBot*](https://allen108108.github.io/blog/2020/04/06/Jetson%20Nano%20%E5%88%9D%E9%AB%94%E9%A9%97%20%28%E4%B8%80%29%20%E2%80%93%20WaveShare%20JetBot/)
* [*Jetson Nano 初體驗 (二) -- Jetson Nano*](https://allen108108.github.io/blog/2020/04/06/Jetson%20Nano%20%E5%88%9D%E9%AB%94%E9%A9%97%20%28%E4%BA%8C%29%20%E2%80%93%20Jetson%20Nano/)
* [*Jetson Nano 初體驗 (三) -- Deep Learning Model*](https://allen108108.github.io/blog/2020/04/16/Jetson%20Nano%20%E5%88%9D%E9%AB%94%E9%A9%97%20%28%E4%B8%89%29%20%E2%80%93%20Deep%20Learning%20Model/)

<!-- more -->

前言
---

前面的工作已經為我們的專案開發打好了基礎，接下來就可以開始進行專案開發。筆者試圖將之前自己作的幾個開發專案移植到 Jetson Nano ，看看是否可行。除此之外，筆者也藉由 Jetson Nano 搭配 MLX 90615 紅外線測溫模組來對專案進行延伸開發。



手勢辨識 Gesture Recognition
---

手勢辨識專案之前筆者已經寫過一篇專題介紹 " [Gesture Recognition using YOLO v3 tiny](https://allen108108.github.io/blog/2020/01/18/Gesture%20Recognition%20using%20YOLO%20v3%20tiny/) " ，但移植到 Jetson Nano 的模型是基於 Darknet 的 YOLOv2 tiny ，訓練資料及時間上都與前文之版本不太相同，因此表現上也有所差異，但這並非本文所要討論的內容，因此不多加解釋。

{%youtube UoOW0amTH9w %}

硬體上與筆電仍有差異，因此效能上會有落差，這是可以預期的，可以藉由 Gstream 調整畫面大小來提升 FPS。但要達到 Nvidia 官方宣稱的 25 FPS 我認為有其難度，這方面還需要再研究。

人臉辨識 Face Recognition
---

這個專案沿用了 "[人臉辨識系統 Face Recognition 開發紀錄 ( OpenCV / Dlib )](https://allen108108.github.io/blog/2020/04/16/%E4%BA%BA%E8%87%89%E8%BE%A8%E8%AD%98%E7%B3%BB%E7%B5%B1%20Face%20Recognition%20%E9%96%8B%E7%99%BC%E7%B4%80%E9%8C%84%20%20%28%20OpenCV%20_%20Dlib%20%29/)" 並且利用 MLX 90615 來量測體溫。

![](https://i.imgur.com/GmxuUp4.jpg)

MLX 90615 是一個紅外線溫度感測模組，支援 I2C 接口，因此額外購入了 其附上的 I2C 接線筆者想直接使用，因此額外購入了 Base Hat for Raspberry Pi Zero 將 MLX 90615 與 Jetson Nano 可以直接串接。

![](https://i.imgur.com/FFiuXhb.jpg)


紅外線偵測必須跟鏡頭同步偵測，因此筆者將 MLX90615 的紅外線探測頭固定在鏡頭之上。

![](https://i.imgur.com/iCkTj5Q.jpg)

![](https://i.imgur.com/WwC4PEh.jpg)

僅將接線鏡頭安裝，仍未能使用，必須要有相關模組的 Library 才能使用。MLX90615 官方僅提供 Arduino Lib，筆者嘗試使用 MLX 90614 Lib 也無法正常運作，最後找到了一個 MLX 90615 可用的 [Python Lib](https://github.com/2n3906/python-sensor-drivers/blob/master/mlx90615.py)

惟要注意的地方是，必須將 `i2c_bus` 與 `i2c_address` 設定正確才可正常運作。

{%youtube 6uDmfeJqv9k %}

從上面的實作上面看來，Inference time 不算太高，接下來還可以嘗試將 Facenet 替換掉 Dlib 進行人臉辨識，看看是否能有更高的效率。