---
title: "Jetson Nano 初體驗 (一) -- WaveShare JetBot"
date: 2020-04-06 08:40:29
categories:
- 創客 Maker
image: https://i.imgur.com/0xqlFni.png
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

一直都想玩玩看開發板，但由於自己實在不懂 Linux 系統，再加上硬體安裝也不熟，深怕一個不小心就被我給搞壞，所以一直抱持著「可遠觀而不可褻玩焉」的心態在面對 Jason Nano。

適逢最近工作上面筆者想要有一些突破，希望能夠有一個平台可以更方便作 Model 的推論，也想要親自驗證自己看過的論文或是一些新方法，便趁此機會購入一台 Jetson Nano & WaveShare JetBot。( 謎之音 :　這也是新挑戰的開始．．．．． )



從東西到手，到完全將環境建好，一共花了我大約一週多的時間，期間的坑真的是靠血淚去填平的。先不說一堆安裝上面會出現的問題，一度我還以為自己手殘把 OLED 給毀了，還好後來也算是救了回來。總結這一段時間來的心得。

這個系列筆者會分成兩篇文章，第一篇，主要介紹純粹 JetBot 的安裝及環境設置。而第二篇就單純介紹 Jetson Nano 本身的應用，這樣的順序編排主要是因為 JetBot 的環境已經被 Nvidia 或 JetBot 開發商完整建構，使用者在進行環境安裝的過程會簡單非常多。
 
Jetson Nano & JetBot
---

Jason Nano 是 Nvidia 官方出產的開發板，與樹莓派 (Raspberry Pi) 最大的不同之處就是其搭載了 GPU ，可以在上面進行深度學習的各種應用，而不用侷限在硬體效能上，而 JetBot 則是基於 Jetson Nano 發展出的人工智慧機器人套件。

我們可以從 [Nvidia 官網](https://www.nvidia.com/zh-tw/autonomous-machines/embedded-systems/jetbot-ai-robot-kit/)上看到，JetBot 種類繁多，分屬 Nvidia 不同合作夥伴所開發，當然，在功能、外型、供電系統上都有著不同程度上的差異，台灣目前可以購入的多為中國廠商所製作的，其中又以微雪電子 ( WaveShare ) 開發的 JetBot 最常見。本文便是以 WaveShare JetBot AI Tool Kits 為基礎所撰寫的入門文章。

![](https://i.imgur.com/ZRfWDMq.png)

硬體安裝
---

![](https://i.imgur.com/LjVHI2v.png)

上圖為 WaveShare JetBot 的整個零組件 : 

1. Jetson Nano ( 不含在 JetBot 中，需另購 )
2. Micro SD card ( 不含在 JetBot 中，需另購，建議64G以上容量 )
3. JetBot 車殼及底板
4. 鏡頭支架
5. 壓克力板
6. JetBot 擴展板
7. IMX218-160 鏡頭，八百萬畫素，160度廣角
8. 無線網卡
9. 直流馬達 x 2
10. 車輪
11. 萬向輪
12. 18650 電池 x 3
13. 12.6V 電源線
14. 無線遙控手把
15. 十字起
16. 排線
17. 螺絲材料包
18. 板手
19. 風扇 ( 本文另裝風扇 )

基本上的一些安裝，可以參考說明書以及下列影片進行安裝，本文不另贅述。

{%youtube jGna0oGKdV4 %}

這裡指出幾個要小心的部分

* ### 無線網卡接線

天線要連接無線網卡的細線請由下圖中機殼的中間偏後方孔洞穿出 ( 筆者拿到的 Waveshare JetBot 機殼孔洞為圓形 )，經筆者試驗過，線由這個地方穿出可以確保連接到無線網路不會不夠長，而且線也不會露在外面感覺很雜亂。


<img width=500 src="https://i.imgur.com/FWbXUTm.png" >




完成如下圖所示


<img width=500 src="https://i.imgur.com/65oUVUK.jpg" >



* ### 6 pin 排線

當硬體全部安裝完成後，這個排線先不要接上，原因是要先進行 Jetson Nano 的系統設置，確定 Jetson Nano 系統沒有問題後再接回測試硬體的連接。

當要接上 Jetson Nano 時，請務必記得排線的位置必須正確

![](https://i.imgur.com/ruFqhbM.jpg)


* ### Noctua NF-A4x10 5V PWM

![](https://i.imgur.com/8ZU0PKn.png)


這是一款頗多人推薦的小型風扇，其優點莫過於超靜音，即使貼近廳都聽不太到風扇運轉的聲音。Noctua NF-A4 有出幾個不同的版本 : 

* A4x10 與 A4x20 : 在於風扇厚度的差異，筆者對於整體外觀還是有點要求的，裝上A4x20整個風扇的比例實在不太對，因此筆者使用的是 A4x20 的版本，這僅差在外觀的不同，對於系統本身均適用。
* 5V 與 12V : 由於整個 Jetson Nano 的供電系統屬於 5V 電壓，因此必須確定買的是 5V 版本的風扇，不然將無法使用。

Noctua NF-A4 建議在 Jetson Nano 上自攻 m3 螺絲孔徑，這對筆者來說太麻煩而且還要注意攻牙後的鋁屑若未清理乾淨可能有短路的風險，因此筆者僅利用一般的 PVC 電線綁帶作固定，除了簡單方便外，還可以有整線的功能。

![](https://i.imgur.com/rB5EAu7.jpg)



環境安裝
---

這一個部分大致上就是分作幾個步驟

1. 將準備好的 Micro SD Card 格式化
2. 將系統燒寫進 Micro SD Card 中
3. 開機確定系統運作無異狀

而其中第一、第二步驟，均在 Desktop 上進行，請勿務必小心，如果真的不小心把自己的硬碟格式化或是覆蓋，那我也只能送給你一個慘字。

### 格式化 SD Card 

無論新舊，都建議將 SD Card 先進行格式化，以免出現問題，至於格式化，在 Windows 系統中只要 SD Card 的硬碟槽上按滑鼠右鍵即可選擇格式化。( 有些文章會有建議的格式化軟體，大家也可以參考看看 )

### 燒寫系統

所謂的燒寫系統，其實就是把一整個硬碟環境複製到任一個硬碟中的概念，為了避免不同設備、環境設置的不同，可能會導致 JetBot 在使用上的各種問題，因此利用這樣燒寫的方式，把官方可用的環境原封不動地放進你的 SD Card 中。這樣一來可以省下許多麻煩的工作，僅需要下載、燒寫就可以使用。

首先，先下載 Nvidia 官方的 [JetBot image](https://drive.google.com/open?id=1G5nw0o3Q6E08xZM99ZfzQAe7-qAXxzHN) 檔案以及燒寫工具 [Win32DiskImager](https://freewarehome.tw/pc/win32-disk-imager/) 或 [Etcher](https://www.balena.io/etcher/)。(本文使用 Win32DiskImager)


<img width=500 src="https://i.imgur.com/xjHMV1H.png" >




上圖兩個部份選擇好後，就可以按下 `write` 進行燒寫。( 請務必確定磁碟機位置沒有錯誤。)這需要一段時間，當燒寫完成後，便可將 SD Card 轉移至 Jetson Nano 來確認系統可以正常運作。

Jetson Nano 的記憶卡插槽十分隱密，位置在風扇、散熱片的正下方底下，插入後暫時**先以 Jetson Nano 本身的供電方式 ( Jetson Nano 適用之電源線)，並且連接滑鼠、鍵盤以及螢幕進行開機**。

![](https://i.imgur.com/lKalzUX.jpg)

因為是整個系統的移植，開機之後就是可以立即使用的 Ubuntu 系統電腦，使用者名稱及密碼均預設為 `jetbot`，且已經安裝所有必需的依賴軟體。開機如果一切順利，我們最重要的事情就是讓其可以連上網路供我們遠端操作，連上網的方式也跟一般電腦相同，選擇 WIFI 輸入密碼後之後就會開機自動連線。

接下來，利用 Ubuntu 的關機程序進行(螢幕右上方)關機，並且移除 Jetson Nano 的電源以及鍵鼠、螢幕。

最緊張的時刻來了，我們將 6 pin 排線接上，並且將 Jetson Nano 接上 JetBot 本身的供電裝置 ( WaveShare 支援邊充電邊使用，因此我們可以直接接上 JetBot 擴展板的電源線進行後續動作 )。

上面的所有安裝都完成後，打開擴展板上的開關應該可以順利開機 ( 若開機， Jetson Nano 左後方的電源燈會亮 )。

沒有接上螢幕，所有的資訊會來自於擴展板後面的 PiOLED 面板

![](https://i.imgur.com/IphsQZa.jpg)
( 剛裝好的 PiOLED 顯示應該會跟我的不太一樣，這很正常，因為剛裝好的裡面用的是預設的系統 )

我們只關注在 IP 位址上面就好，請記住這個 IP 位址，這代表目前 JetBot 
上網的 IP 位址，有了這個位址，我們就可以從其他電腦使用 Jupyter Notebook 來進行遠端控制。

找一台電腦，在瀏覽器網址輸入 `<IP_Address>:8888` 即會進入 Jupyter Notebook ，密碼一樣為 `jetbot`。先不管裡面有什麼內容，先按下左上方的 `+` 準備進入終端機來進行指令操作

![](https://i.imgur.com/RXvO9HA.png)

按下下圖紅框圖示進入終端機

![](https://i.imgur.com/LyXXlxL.png)


進入終端機後，我們要作的就是更新 JetBot 專案，這個專案就是 JetBot 所有運作的關鍵。然而有個部分要注意，在許多的文章說明中都指出，你可以由下列擇一下載

* Nvidia 官方的 JetBot Repository 
( https://github.com/NVIDIA-AI-IOT/jetbot )
* WaveShare 的 JetBot Repository 
( https://github.com/waveshare/jetbot )

但經由筆者實測，若是 WaveShare JetBot 則必須要使用 WaveShare JetBot Repository 才可以運作，否則會造成 PiOLED 正常但是發動機無法作動的情況。

因此在本文推薦使用 WaveShare 版本來更新，請依序輸入下列指令 : 

```
$ git clone https://github.com/waveshare/jetbot
$ cd jetbot
$ sudo python3 setup.py install

$ cd
$ sudo apt-get install rsync
$ rsync -av jetbot/notebooks/ Notebooks/
```

微雪版本並非最新的版本，較 Nvidia 官方的為舊，但是其優點在於 PiOLED 中可以顯示電量，在使用 JetBot 時便可以清楚什麼時候必須要充電，避免因為電量不足而關機的狀況，這就也是會呈現出上面 PiOLED 圖片的顯示。

### 風扇開啟

前面我們只進行了風扇的安裝，但仍未對系統開啟風扇，所以你會發現風扇一直都沒有運傳。開啟風扇很簡單，只要依照下列步驟將風扇加入 service 之中，使其可以於開機時便開始運作

```
$ git clone https://github.com/Pyrestone/jetson-fan-ctl.git
$ cd jetson-fan-ctl
$ ./install.sh
```


測試
---

>**在測試之前，建議先確定車子所在位置是足夠空曠且有足夠活動空間，不然執行下>去車子亂跑可能會「發生車禍」，可以將車輪先拆除或是將車體架高來進行測試。**

使用 Jupyter Notebook 遠端連進 JetBot 時，可以從 `jetbot/notebooks` 裡面看到許多不同範例的資料夾

* basic_motion 
* collision_avoidance
* object_following
* road_follwing
* teleoperation

可以試著執行看看每一個 `ipynb` 檔，按照上面的順序進行，應該是可以順利執行這些動作。這裡就僅僅是執行程式，筆者也就不花費太多時間說明。


### 鏡頭紫邊

在測試的時候，或許會發現，執行鏡頭時畫面會出現嚴重的色差、紫邊，一度筆者覺得自己也太過幸運拿到機王，但查閱資料後發現，這似乎是 Jetson Nano 在使用  CSI 鏡頭時很容易會有的狀況。

![](https://i.imgur.com/T2CfVkV.png)

如果我們的 Model 在訓練的時候有作 Data Augmentation，這樣的色差或許可以對於我們在 inference 上面不會有非常大的影響，但這非常的難以控制，因此在作電腦視覺的專案時，盡可能的還是得避免 input image 的色差狀況。

所幸，這個問題可以被解決。

依序執行

```
$ wget https://www.waveshare.com/w/upload/e/eb/Camera_overrides.tar.gz
$ tar zxvf Camera_overrides.tar.gz
$ sudo cp camera_overrides.isp / var / nvidia / nvcam / settings / 
$ sudo chmod 664 /var/nvidia/nvcam/settings/camera_overrides.isp 
$ sudo chown root：root /var/nvidia/nvcam/settings/camera_overrides.isp
```

執行完後我們就可以來看一下結果。

![](https://i.imgur.com/9yd4Dlj.png)


後記
---

這一整趟下來，也是花了將近兩周的時間在處理。不過完成環境的建構後，就可以開始想想看可以進行什麼樣子的專案。

目前有兩個比較大的 Project 想要進行 : 

1. **利用 JetBot 進行 SLAM 任務**
2. **人臉辨識配置紅外線體溫偵測**

另外 **利用手勢遙控 JetBot** 感覺也是個有趣的方向可以玩玩呢 ! 其實有些專案並不需要 JetBot 環境就可以作，如果我們今天只需要單純的 Jetson Nano 又該怎麼建構環境呢 ? 這就是下一篇要來提的了。

