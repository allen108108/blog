---
title: Gesture Recognition using YOLO v3 tiny
date: 2020-01-18 09:38:00
categories:
- 專案 Project
image: https://i.imgur.com/WiuMJgp.png
mathjax: true
---

Introduction
---

此專案目標是利用 YOLOv3-tiny 進行七組手勢辨識，使用 11 萬張訓練資料以及 4 萬張驗證資料進行 300 epochs 訓練。（ 仍在訓練中，目前使用的是 60 epochs （訓練時間約80小時）訓練後的權重進行測試 ）

<!-- more -->

Datasets
---

電腦視覺領域中，手勢辨識似乎都不是一門顯學，因此在資料集、論文...等等資源上都略顯缺乏，為了快速地可以看看 YOLO 在手勢辨識上的成效，又要兼顧資料本身分布的問題，主要還是尋找 Open Datasets 為主。

找了許多的 Datasets 後，最後使用了論文 “ *Tiny hand gesture recognition without localization via a deep convolutional network* ” 所使用的[資料集](https://sites.google.com/view/handgesturedb/home)，這個資料集中使用了一共 40 個人的手勢圖像資料，並分七種手勢存放各別資料夾。

![](https://i.imgur.com/pmNChkX.jpg)

![](https://i.imgur.com/MyXuUoP.png)


這樣的資料集就資料的分布上來說並不算太好，取景的角度及距離大概如同上圖，只有兩種取景距離，這嚴重地影響了之後模型的泛化能力，使其辨識距離大大受限。

標註圖像 Label
---

### Labelimg

Github : https://github.com/tzutalin/labelImg

這是目前在物件偵測任務上，最廣為人知的 Label tool，如果你搜尋 YOLO 相關實作文章，也大多會遇到作者史工具來進行人工標註。

使用方法非常簡單，將 Labelimg 整個專案 clone 至本地後，在 Labelimg 位址下執行 `labelimg.py` 即可執行 Labelimg 進行標註。

```
python labelimg.py
```

在進行標註前，建議先至專案內`data` 資料夾內 `predefined_classes.txt` 檔案進行 Classes 的修改。

修改完成後我們即可開始進行標註

![](https://i.imgur.com/lDzRaTW.jpg)

每製作完一張圖像的 Label 存檔後，即會在圖片的資料夾中生成一個與圖片檔案名稱一樣的文字檔，裡面就是對此圖像的標註

![](https://i.imgur.com/TIvojzx.png)

輸出文字檔內格式如下 : 

```
Label x_center y_center width height 
```
Label 是 從 0 開始的 Label index，而後面四個數字都是已經 Normalization 過後的數字，這非常重要，因為不同的標註工具產出的格式會非常不一樣，除此之外，不同的 YOLO implementation 所需要的格式也會不同，這都是必須注意的部分。


### DarkLabel

https://darkpgmr.tistory.com/16

從上面的 Labelimg 可以看出來，如果我們有非常大量的未標註資料，利用 Labelimg 會非常花時間，再者，許多時候我們收集資料，會利用連續照片或是利用影片連續擷取影像，這樣的方法可以在短時間內取得非常大量的資料 ( 我們所利用的資料集便是利用這樣的方式來取得資料 )，這樣的狀況下， DarkLabel 便會使標註效率大大提升。

DarkLabel 之所以可以達到 Label 的高效率，最重要的是因為它具有物件追蹤的功能，且可以選擇非常多樣化的輸出格式。

以下是官方 Demo 影片 : 

{%youtube vbydG78Al8s %}

DarkLabel 免安裝直接解壓縮後開啟即可，但僅限於 Windows 系統可用，選擇圖片檔案位置及格式後，就可以立刻進行邊界框的 Label，每一張圖片 Label 完成後不用按儲存，可以儲蓄 label 多張照片後在儲存一次即可。( 不要挑戰從頭 label 到尾才要按儲存，如果中間閃退還是會讓人崩潰 )

![](https://i.imgur.com/WOlwkaf.jpg)

```
frame : 圖像 index
iname : 圖像檔案名稱
label : label index
cx,cy,w,h : 中心點 x 座標，中心點 y 座標，寬度，高度
x1,y1,x2,y2 : 左上角 x 座標，左上角 y 座標，右下角 x 座標，右下角 y 座標
```


如果是連續動作的圖片，基本上第一張圖片 Label 好以後就可以一直按下一張 ( 我習慣按方向鍵的右鍵 )，這樣在同一個 Label 但是多張連續圖像的狀況下可以迅速進行標註。我只要花大概一到兩天的時間就可以標註幾萬張的圖片。

DarkLabel 跟 Labelimg 不同的點在於，它是將所有資料的 Label 資料存在一個檔案中，如果遇到必須要一張圖像一個 Label 檔的 Model 就必須要做後處理，但這不會是什麼大問題，寫個 Code 自動化處理即可。

DarkLabel 還有一個優點在於，假如我們已經 Label 完所有的資料，但遇到格式不同的需求時，不須重新標註資料，打開 DarkLabel 載入原本的 Label 資料及格式，在底下儲存的地方選取自己要的新格式後儲存，即可將幾萬筆資料儲存成新的格式。


訓練 Training
---

在利用 yolo 對自己的資料進行訓練前，必須要先做設定檔 (cfg file) 的調整，優於類別數量的不同，會導致 Model 內部的結構性差異。

以 YOLOv3-tiny 來說，預設通常是 COCO Datasets 的 $80$ 個類別，而其中 filters 的預設數量會是 $(4+1+80)\times 3=255$，以本專案來說，我們只會定義 $7$ 種 classes，因此在設定檔中必須更改 classes 以及 filters 的數量

```
classes = 80   ->  classes = 80
filters = 255  ->  filters = 36
```

本次訓練在 Windows 作業系統下使用 NVDIA GTX 1060 6G 獨立顯卡進行訓練，一個 epoch 大約訓練 1個小時左右，僅訓練約 70 epochs 。



Real-Time inference
---

我利用僅訓練 70 epochs 所儲存的權重進行 Real-time inference，一樣使用 GPU，可以達到 30 FPS ，inference time 每幀大約 0.009 ms。

Lenovo Legion Y530 with NVDIA GTX 1060 6G : 

{%youtube d0x4QLk7FuQ %}

效果似乎不算太差，但由於前面提到資料及分布的問題，導致手的位置太靠鏡頭則會偵測不出來，這是未來在收集資料時必須考量到的部分。

後續工作 TODOs
---

* 能否將 Model 及權重轉換成 TensoFlow Lite 格式，這樣可以在移動設備上進行手勢辨識。
* 如果使用 MobileNet SSD 能否有類似的 Performance ? Inference Time 表現何者較好 ?

目前在移動設備上進行手勢辨識的一些專案及論文中，大多使用 MobileNet SSD 來進行，這其實蠻令人好奇，YOLO-tiny 也不算是很龐大的網路結構，辨識的能力也不算太差，不知移植到移動設備上是否 Performance 會掉很多 ? 這是未來想要來進行看看的工作。