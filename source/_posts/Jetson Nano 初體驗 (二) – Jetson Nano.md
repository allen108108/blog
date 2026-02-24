---
title: "Jetson Nano 初體驗 (二) -- Jetson Nano"
date: 2020-04-06 08:41:35
categories:
- 創客 Maker
image: https://i.imgur.com/KzdaeEL.png
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

前一篇文章 " [*Jetson Nano 初體驗 (一) -- WaveShare JetBot*](https://allen108108.github.io/blog/2020/04/06/Jetson%20Nano%20%E5%88%9D%E9%AB%94%E9%A9%97%20%28%E4%B8%80%29%20%E2%80%93%20WaveShare%20JetBot/) " 提到怎麼組合並建構一個 JetBot ，但不見得每一個人在利用 Jetson Nano 開發都必須要利用到 JetBot，單單一片 Jetson Nano 開發板就可以部屬深度學習的模型 : 人臉辨識、物件追蹤、分類模型...等等，都不需要兩個輪子趴趴走，因此，僅使用 Jetson Nano 怎麼開發專案，也是必須要去學習且了解的方向。 



和 JetBot 在整個建構環境上的不同之處在於

* 不需要繁瑣且謹慎的硬體安裝過程，但
* 軟體安裝上面耗時又費工

軟體上面的安裝，可以依據使用者開發的內容來決定依賴項的安裝，麻煩，但也顯示了純粹用 Jetson Nano 進行開發的彈性會相對來說大。

環境及硬體安裝
---

和 JetBot 一樣，我們必須將 Jetson Nano 的[官方環境](https://developer.nvidia.com/jetson-nano-sd-card-image)燒進 Micro SD 卡中，這個環境裏面有 CUDA, CuDNN, OpenCV .... 等預先安裝好的套件及依賴，筆者建議，要拿來燒寫系統的 Micro SD 卡至少要 64 G，最好選用 128 G。

硬體部分，筆者在前一篇有提到，使用的是 Noctua NF-A4 風扇，鏡頭也沿用 JetBot 的 CSI 鏡頭，環境安裝過程以及風扇運作詳情請見[前一篇文章](https://allen108108.github.io/blog/2020/04/06/Jetson%20Nano%20%E5%88%9D%E9%AB%94%E9%A9%97%20%28%E4%B8%80%29%20%E2%80%93%20WaveShare%20JetBot/)。

上述工作安裝完成後，連接電腦螢幕及鍵鼠設備開機進行一些 Ubuntu 的基本設定，設定其實都很簡單，不過為了遠端連線方便，請記得將登入方式設定為開機自動登入。



遠端連線
---

JetBot 環境中可以直接遠端連線，但若單純使用 Jetson Nano ，遠端連線就必須要自行設置，在這裡我們使用 VNC Viewer 來進行遠端連線的工具，在還未進行遠端連線之前，仍然需要先連接電腦螢幕及鍵鼠設備以利操作。


### Jetson Nano 端
首先，在 Jetson Nano 的部份我們要先進行一些安裝及設置

```
# 安裝 vino 遠端連線
$ sudo apt-get install vino

# 安裝 dconf-editor 修改桌面設定
$ sudo apt-get install dconf-editor

# 安裝 nano 編輯器
$ sudo apt-get install nano -y

$ dconf-editor 
# 在 /org/gnome/desktop/remote-access 內將 require-encryption 加密需求設為關閉

# 使用 nano 編輯器修改 Gnome 桌面設定 
$ sudo nano /usr/share/glib-2.0/schemas/org.gnome.Vino.gschema.xml
```

進入 nano 編輯器後，向下滑到最後，在 `</schema>` 前加入 : 

```
    <key name='enabled' type='b'>
      <summary>Enable remote access to the desktop</summary>
      <description>
        If true, allows remote access to the desktop via the RFB
        protocol. Users on remote machines may then connect to the
        desktop using a VNC viewer.
      </description>
      <default>false</default>
    </key>
```

完成後，按下 `Ctrl+ x`  離開， 離開前會確認是否存檔，按下 `y` 後，按下 `Enter` 確認檔名離開即可。

```
# 編譯 Gnome Schema
$ sudo glib-compile-schemas /usr/share/glib-2.0/schemas
```

進入 Startup Application 設定將 Vino 設定為啟動程式

![](https://i.imgur.com/8uiDXDn.png)

選擇 `Add` 後， `Name` 與 `Comment` 可以自設，`Command` 部分輸入 `/usr/lib/vino/vino-server`，按下 `Add` 儲存設定。

![](https://i.imgur.com/xeg5otO.jpg)

隨後，進入設定內的共享桌面中 : 

* 選取 `Allow other users to view your desktop` 
* 選取 `Allow other users to control your desktop`
* 取消 `You must confirm each access to this machine`
* 選取 `Require the user to enter this password` 以設定共享密碼

最後在終端機輸入 `sudo reboot` 重新開機後即完成 Jetson Nano 端的所有設定。

### Desktop 端

電腦端相對簡單，下載並安裝 [VNC Viewer](https://www.realvnc.com/en/connect/download/viewer/) 後，開啟應用程式，在網址列輸入 Jetson Nano 的連線 IP 位置即可進行遠端桌面操作。

開啟後，會發現解析度非常低，我們可以利用遠端在 Jetson Nano 的終端機輸入 `sudo xrandr --fb 1920x1080` 來設定遠端桌面的解析度。





軟體安裝
---

正如前面所說，雖然系統已經預載了一些套件，但大多數的工具都必須重新下載並安裝，因此要有心理準備，整個安裝時間會非常耗時


### 環境變數

首先我們要先進行環境變數的設定，雖然預載了 CUDA 以及 CuDNN，但仍未設置環境變數，所以我們處理這個部分。

```
$ sudo nano ~/.bashrc
```
至最後面加入以下內容：
```
export CUDA_HOME=/usr/local/cuda-10.0
export LD_LIBRARY_PATH=/usr/local/cuda-10.0/lib64:$LD_LIBRARY_PATH
export PATH=/usr/local/cuda-10.0/bin:$PATH
```
完成後，使其生效：
```
source ~/.bashrc
```
最後我們可以輸入 `nvcc -V` 進行測試。

### 設定 swap 交換磁區

Jetson Nano 的記憶體僅有 4G，因此我們必須設定 swap 來避免 Jetson Nano 因為記憶體過載而出現問題，在這邊我們 swap 僅設定 4G，但建議最好設定 8G。

```
$ sudo fallocate -l 4G /swapfile

$ sudo chmod 600 /swapfile

$ ls -lh /swapfile

$ sudo mkswap /swapfile

$ sudo swapon /swapfile

$ sudo swapon –show

$ sudo cp /etc/fstab /etc/fstab.bak

$ echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

```


### 安裝必要套件

許多的套件或工具，最後下載下來後都還需要作編譯、建構環境等後續工作，因此我們要先對這些編譯、建構工具等，先進行安裝，以利後續的安裝作業。

這個部份還會有順序的問題，建議依照下面的順序進行安裝。

```
$ sudo apt-get update

$ sudo apt-get upgrade

$ sudo apt-get install git cmake

$ sudo apt-get install libfreetype6-dev pkg-config -y

$ sudo apt-get install zlib1g-dev zip libjpeg8-dev libhdf5-dev -y

$ sudo apt-get install libssl-dev libffi-dev python3-dev -y

$ sudo apt-get install libhdf5-serial-dev hdf5-tools

$ sudo apt-get install libblas-dev liblapack-dev

$ sudo apt-get install libatlas-base-dev gfortran

$ sudo apt-get install build-essential libgtk-3-dev libboost-all-dev -y

```
安裝 `pip` 套件

```
$ wget https://bootstrap.pypa.io/get-pip.py

$ sudo apt install python3-testresources

$ sudo python3 get-pip.py

$ rm get-pip.py
```

安裝虛擬環境工具，這邊筆者使用 `mkvirtualenv` 來建構虛擬環境，相較於 `virtualenv`，可以將虛擬環境集中管理，也便於專案的管理分配，此外，要進入虛擬環境也可以使用 `workon` 指令快速進入環境。

```
$ sudo pip install virtualenv virtualenvwrapper

$ nano ~/.bashrc
```

在 nano 編輯器的最後加入下列文字

```
# virtualenv and virtualenvwrapper
export WORKON_HOME=$HOME/.virtualenvs
export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
source /usr/local/bin/virtualenvwrapper.sh
```
建立一個 `deep_learning` 虛擬環境

```
$ source ~/.bashrc

$ mkvirtualenv deep_learning -p python3

$ workon deep_learning
```

接下來，就是安裝一些機器學習、深度學習的常用套件


```
$ pip install numpy

$ pip install scipy

$ pip install cython

$ pip install matplotlib

$ pip install scikit-build

$ pip install imutils

$ pip install pillow

$ pip install pandas

$ pip install jupyter notebook
```

下載 Keras 以及 TensorFlow，筆者認為在 TensorFlow 1.X 版本中， 1.12 版本仍然是比較穩定的版本，所以筆者在此是安裝 TensorFlow 1.12.0，但按照各種專案的不同，可能會需要不同版本的 TensorFlow，建議大家可以先至下列網址尋找自己需求的版本進行下載

* https://developer.download.nvidia.com/compute/redist/jp/v411/tensorflow-gpu/
* https://developer.download.nvidia.com/compute/redist/jp/v42/tensorflow-gpu/
* https://developer.download.nvidia.com/compute/redist/jp/v43/tensorflow-gpu/



```
$ pip install keras

$ pip install --extra-index-url https://developer.download.nvidia.com/compute/redist/jp/v411 tensorflow-gpu==1.12.0+nv19.1
```


### OpenCV

#### 方法一

其實 Jetson Nano 已經預載了 OpenCV，最簡單的方式便是將系統的 OpenCV 直接連結到虛擬環境中。
```
# 先確定 Jetson Nano OpenCV 的預載位址
$ sudo find / -name "cv2*"
```

![](https://i.imgur.com/ZYlfxMq.png)


```
# 進行虛擬環境 OpenCV 的連結
$ cd ~/envs/AI/lib/python3.6/site-packages/
$ ln -s /usr/lib/python3.6/dist-packages/cv2.cpython-36m-aarch64-linux-gnu.so
```

但筆者利用這樣的方法，發現 OpenCV 似乎沒有用到 CUDA 。


#### 方法二

第二種方法，其實也就是重新編譯 OpenCV，網路上已經有許多高手將 Jetson Nano 的 OpenCV 編譯安裝都寫成腳本，直接執行就可以進行安裝。

最有名的有 : 

* [JetsonHacks - OpenCV 4 + CUDA on Jetson Nano](https://www.jetsonhacks.com/2019/11/22/opencv-4-cuda-on-jetson-nano/)
* [JK Jung's blog -Installing OpenCV 3.4.6 on Jetson Nano](https://jkjung-avt.github.io/opencv-on-nano/)

上列文章的方式都讓 OpenCV 的編譯變得簡單許多。


### Dlib

在 Ubuntu 上對 Dlib 的安裝簡單也順利許多，而且可以順利使用到 GPU 進行處理，只要按照下列進行下載安裝即可。

```
$ mkdir temp; cd temp

$ git clone https://github.com/davisking/dlib.git

$ cd dlib

$ mkdir build; cd build

$ cmake .. -DDLIB_USE_CUDA=1 -DUSE_AVX_INSTRUCTIONS=1

$ cmake --build .

$ cd ..

$ python setup.py install

$ sudo ldconfig
```

我們可以進入 python 來進行上面幾個套件的安裝確認

```python=
import tensorflow as tf
import keras
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import cv2
import dlib

print(tf.test.is_gpu_available())
print(cv2.__version__)
print(dlib.DLIB_USE_CUDA)
```

### Jetson Stats

最後，建議安裝 [Jetson Stats](https://github.com/rbonghi/jetson_stats)，這個套件可以藉由簡單的指令 `jtop` ，來達成整個環境的配置狀態、CPU / GPU 使用狀況、風扇運轉狀態、...等等。

安裝也非常的簡單

```
sudo -H pip install -U jetson-stats
```

障礙排除
---

在建構單純 Jetson Nano 環境的過程當中，其實出現一些狀況，並不是這麼順利，因此這一段主要將我遇到的問題作一個簡單的紀錄，以便於未來可以參閱 

### 鏡頭紫邊

鏡頭紫邊的狀況仍然需要解決，在上一篇文章中已經提出解決方式，這裡也不再贅述。

### JetBot Oled 無法運行

雖然這裡我們是單純使用 Jetson Nano 進行環境安裝，但是我還是有 JetBot 套件，所以仍然希望可以使用到 JetBot 上面的 OLED 面板顯示，這也方便我進行 IP 位址的查詢進行遠端連線。

沒有 JetBot 環境，該怎麼使用 OLED 呢 ? 首先要作的當然是將 6pin 排線連接 Jetson Nano 及 JetBot 擴展板 ( 相同的，請注意排線安裝位置 )。

連接完成後按照下列步驟進行

```
$ git clone https://github.com/JetsonHacksNano/installPiOLED.git

$ cd installPiOLED

$ ./installPiOLED.sh

$ ./createService.sh
```

最後一個步驟其實是將顯示 OLED 的工作加入 service 中，確保每一次開機都能開啟 OLED。

按照上面的 Repo 設計，OLED 會顯示四行

* 有線網路 IP
* GPU 使用計量條
* 記憶體用量
* SD 卡用量

然而筆者希望要有無線網路的 IP 位址，因此希望將顯示方式作調整，這裡我們使用 Nvidia JetBot 的 OLED 顯示方式來替代，因此先將 Nvidia 的 [OLED script](https://github.com/NVIDIA-AI-IOT/jetbot/blob/master/jetbot/apps/stats.py) 下載到 `/installPiOLED/pioled/` 內，為避免與原本資料夾中的 `stats.py` 重複，我們將此檔名更改為`oled.py`。

由於我們前面已經建立了 service，因此我們從已建立的 service 中進行更改。已建立的 service 位址在 :  `/etc/systemd/system/pioled_stats.service`

打開 `pioled_stats.service` 檔案將

```
ExecStart=/bin/sh -c "python3 -m pioled.stats"
```

更改為
```
ExecStart=/bin/sh -c "python3 /home/<user_name>/installPiOLED/pioled/oled.py"
```

重新開機後就可以發現 OLED 的顯示會跟 Nvidia 官方 JetBot OLED 的顯示相同。


### opencv 無法獲得畫面

這個問題是我嘗試很久的時間，卻一直無法解決的狀況。我們在 Desktop 中要利用 `cap = cv2.VideoCapture(0)` 取得鏡頭的畫面非常的易如反掌。

然而，筆者在 Jetson Nano 卻一直無法順利進行，當我在 Jupyter Notebook 中試圖使用 `cap = cv2.VideoCapture(0)` 會一直出現錯誤

```
[ WARN:0] global /home/nvidia/host/build_opencv/nv_opencv/modules/videoio/src/cap_gstreamer.cpp (1757) handleMessage OpenCV | GStreamer warning: Embedded video playback halted; module v4l2src0 reported: Internal data stream error.
[ WARN:0] global /home/nvidia/host/build_opencv/nv_opencv/modules/videoio/src/cap_gstreamer.cpp (886) open OpenCV | GStreamer warning: unable to start pipeline
[ WARN:0] global /home/nvidia/host/build_opencv/nv_opencv/modules/videoio/src/cap_gstreamer.cpp (480) isPipelinePlaying OpenCV | GStreamer warning: GStreamer: pipeline have not been created
```
然後有時候會得到綠色的畫面，總而言之就是無法由鏡頭得到畫面。

原本認為是否鏡頭沒有被 Jetson Nano 偵測到，但從終端機輸入下列指令卻可以順利得到畫面

```
gst-launch-1.0 nvarguscamerasrc sensor_mode=0 ! 'video/x-raw(memory:NVMM),width=3820, height=2464, framerate=21/1, format=NV12' ! nvvidconv flip-method=0 ! 'video/x-raw,width=960, height=616' ! nvvidconv ! nvegltransform ! nveglglessink -e
```
也可以利用 `	
ls -l /dev/video*` 來確認系統的確有抓到相機

![](https://i.imgur.com/9CVOhs7.jpg)


查詢了許多 Script 或是論壇主題，通常都是建議 OpenCV 利用 Gstreamer 來取得畫面

```python=
gst_pipeline = ("nvarguscamerasrc ! "
                "video/x-raw(memory:NVMM), "
                "width=3820, height=2464, "
                "format=(string)NV12, framerate=21/1 ! "
                "nvvidconv flip-method=0 ! "
                "video/x-raw, width=1920, height=1080, format=(string)BGRx ! "
                "videoconvert ! "
                "video/x-raw, format=(string)BGR ! appsink"
                )
                
cap = cv2.VideoCapture(gst_pipeline, cv2.CAP_GSTREAMER)                
```
這樣的方法的確可行，但為什麼無法使用舊方式的原因我仍然沒有找到，或許目前僅能暫時以這種方式來進行 OpenCV 專案的開發了。

後記
---

Jetson Nano 的整體環境設置的確比 JetBot 來的複雜且耗時，不過不管是 Jetson Nano 還是 JetBot，動手實際上操作個幾次，也其實沒有想像中困難，待這些前置工作都完成後，就可以開始準備專案的實現及操作。

這些套件、環境安裝完成後，其實也佔去接近 30G 的 SD 卡容量，這也是前面筆者提到最好使用 128G SD 卡的原因了，若有許多專案要進行管理，剩下 20-30 G 的容量也真的是不太足夠。

參考資料
---

1. [Jetson Nano系列教程1：烧写系统镜像](http://www.waveshare.net/study/article-881-1.html)
2. [NVida Jetson Nano 初體驗（一）安裝與測試](https://chtseng.wordpress.com/2019/05/01/nvida-jetson-nano-%E5%88%9D%E9%AB%94%E9%A9%97%EF%BC%9A%E5%AE%89%E8%A3%9D%E8%88%87%E6%B8%AC%E8%A9%A6/)
3. [[第一次用Jetson Nano 就上手]硬體介紹、開機步驟、遠端連線（繁體）](https://www.rs-online.com/designspark/jetson-nano-1-cn)
4. [Getting started with the NVIDIA Jetson Nano - pyimagesearch](https://www.pyimagesearch.com/2019/05/06/getting-started-with-the-nvidia-jetson-nano/)
5. [OpenCV 4 + CUDA on Jetson Nano - JetsonHacks](https://www.jetsonhacks.com/2019/11/22/opencv-4-cuda-on-jetson-nano/)
6. [Jetson Nano + Raspberry Pi Camera - JetsonHacks](https://www.jetsonhacks.com/2019/04/02/jetson-nano-raspberry-pi-camera/)
7. [Installing OpenCV 3.4.6 on Jetson Nano - JK Jung's blog](https://jkjung-avt.github.io/opencv-on-nano/)
8. [how to run a jetbot right after bootup? #55 - NVIDIA-AI-IOT/jetbot](https://github.com/NVIDIA-AI-IOT/jetbot/issues/55)
9. [GStreamer: Error when accessing onboard camera - NVIDIA Developer Forum](https://forums.developer.nvidia.com/t/gstreamer-error-when-accessing-onboard-camera/80446)
10. [How to slove opencv green screen? - NVIDIA Developer Forum](https://forums.developer.nvidia.com/t/gstreamer-error-when-accessing-onboard-camera/80446)