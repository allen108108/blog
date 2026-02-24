---
title: Tensorflow-GPU的安裝及確認 ( Win 10 )
date: 2019-10-07 15:03:32
categories:
- 軟體 Software
image: https://www.tensorflow.org/images/tf_logo_social.png
mathjax: true
---

> 安裝前請先確認是否安裝Anaconda，若仍無安裝可參閱”[Anaconda的安裝](http://bit.ly/2ANwdnd)”

---

<!-- more -->

年前跟老婆申請了一台筆電，就是為了方便日後進行DL運算使用，過年後拿到電腦後，便著手來進行tensorfloe-GPU的安裝，參考了前人的一些步驟流程，仍然遇到一些安裝上面的問題，雖說過了一個多月了，還是想把它紀錄下來，以後若要重新安裝也可以回頭來看看怎麼處理遇到的問題。

## 安裝流程


<img width=350 src="https://i.imgur.com/kVce4bf.png" >


### 確定顯卡型號

由於目前預算上考量，也沒辦法選擇最頂級的獨顯筆電，因此當初在考量筆電上面，便以搭載NVIDIA GeForce GTX1060 6G 獨顯，記憶體能擴充至32GB的筆電為考量。在進行後面各項安裝前，獨顯的型號必須優先考慮，因為CUDA、CudNN及Tensorflow-GPU的各版本間並沒有廣泛的互相支援[^註1] ，必須要按照對應版本安裝才不會有問題。 



### CUDA

首先先至[NVIDIA CUDA](https://developer.nvidia.com/cuda-downloads)官網下載
![](https://i.imgur.com/FSWWwJV.png)


> 此處選取系統平台進行安裝的是CUDA最新版本，此版本並未支援您的顯卡，則必須到右下角的 " [Legacy Releases](https://developer.nvidia.com/cuda-toolkit-archive)" 中尋找對應版本進行安裝

> 根據許多前輩們指出，進行network安裝很容易出現問題，因此也建議大家安裝時候進行local安裝
 

這樣看似簡單的步驟，我仍然在安裝過程中遇到一點麻煩的狀況[^註2]。


### CudNN
進入[CudNN](https://developer.nvidia.com/rdp/form/cudnn-download-survey/)頁面進行下載前，必須先加入NVIDIA Developer Program
    
![](https://i.imgur.com/aCwQC6B.png)
    
加入會員並且登入後，即可選擇相對應的版本下載，下載完成後建立一個新的資料夾(此處範例 C:\pythonwork)，並將下載的壓縮檔解壓縮至此。並確認資料夾內cudnn64_7.dll路徑 ( 此初應為 C:\pythonwork\cuda\bin )
![](https://i.imgur.com/Qy3XUXi.png)
    
至「編輯系統環境變數」>「環境變數」，進行環境變數的變更

![](https://i.imgur.com/otD1OZj.png)

![](https://i.imgur.com/8G18Zvz.png)
    
選取Path進行編輯，在行末加入 " :\pythonwork\cuda\bin; " (引號不須加入，但裡面的分號要留著)後按下確認。



### 安裝虛擬環境
先至命令提示字元中切換成我們要的資料夾( C:\pythonwork)後輸入:
`conda create --name tensorflow-GPU pyhton=3.5 anaconda`
    
一陣子後畫面會問你`Proceed ([y]/n)?`按下y後完成虛擬環境的安裝
    
安裝完成後我們還要有一個執行此虛擬環境的指令
`activate tensorflow-GPU`
執行後，在C:\pythonwork的前面會出現 (tensorflow-GPU) 即代表已成功啟用虛擬環境。
    


### 安裝tensorflow GPU版本
此步驟依然要選擇你安裝的CUDA、CudNN應該配合tensorflow的版本來進行安裝，一般要安裝最新的版本，直接打以下的指令即可
`pip install tensorflow-gpu`
但若要指定版本進行安裝則要輸入以下指令
`pip install tensorflow-gpu==1.12`
    
    
### 安裝keras
`pip install keras`


當我們完成以上步驟，原則上應該都已經將所有工具安裝完成了，但是為了避免再使用的時候才驚覺安裝出現問題，我會建議先做一輪檢查。

1. 先確認各套件均安裝完成且沒有版本衝突
在jupyter notebook中先嘗試匯入所有套件

```
import tensorflow as tf
import keras
import matplotlib.pyplot as plt
```

> 當初我也是信誓旦旦覺得一定沒問題，但是到了這個步驟卻怎麼樣都會出現
> `Failed to load the native TensorFlow runtime`這樣的traceback，後來找了許久也算是解決了這個莫名的問題[^註3]。



2. 確認tensorflow版本
在jupyter notebook中輸入並執行以下code

`tf.__version__`

即可出現版本

3. 最後確認tensorflow是否正確使用GPU
輸入以下code進行測試

```
# Creates a graph.
a = tf.constant([1.0, 2.0, 3.0, 4.0, 5.0, 6.0], shape=[2, 3], name='a')
b = tf.constant([1.0, 2.0, 3.0, 4.0, 5.0, 6.0], shape=[3, 2], name='b')
c = tf.matmul(a, b)
# Creates a session with log_device_placement set to True.
sess = tf.Session(config=tf.ConfigProto(log_device_placement=True))
# Runs the op.
print(sess.run(c))
```

在jupyter notebook上跑完之後僅會出現答案，
此時要轉到命令提示字元上看會發現如下畫面
![](https://i.imgur.com/hoPoUXD.png)

即可確定tensorflow的確是使用GPU進行運算


進行到此，總算是真的完成了整個tensorflow-GPU版本的安裝，也恭喜完成了一項艱鉅的任務啊啊啊啊啊啊啊啊XDDDDDD

註釋
---
[^註1]: 
[TensorFlow與CUDA、CudNN之版本對照表](https://www.tensorflow.org/install/source_windows)
[Nvidia GPU與CUDA之版本對照表](https://developer.nvidia.com/cuda-gpus)
(2019.03.26後記)
在上面的連結中，官方建議的版本搭配為 : TensorFlow 1.12 + CUDA 9 + CudNN 7在上述各種測試當中，也似乎都可以運作，沒有問題
但是實際再跑CNN Model時卻出現 Traceback `Error : Failed to get convolution algorithm.This is probably because cuDNN failed to initialize, so try looking to see if a warning log message was printed above.`
許多建議都是將 TensorFlow 降版到 1.8 ，但這實在很奇怪，我自己也希望使用較新版本，因此尋找許久終於看到[一線曙光](https://github.com/tensorflow/tensorflow/issues/24828)。看來  CudNN 7.0上述的搭配上面會出現無法兼容的狀況，而且不是特例。許多人建議將CudNN升級至 7.4版本便可解決。(這的確解決了我的問題XD)

[^註2]: 
在安裝時一直會出現錯誤訊息如下 :
![](https://i.imgur.com/nVBFZCk.png)
我嘗試過許多種方式 (關掉內顯、重新安裝讀顯驅動... )，這樣的訊息仍然會依直跳出，最後我的解決方法是無視。而最後也證實這樣的方式沒有使用上的問題。

[^註3]: 
[導入Keras、TensorFlow 時出現：Failed to load the native TensorFlow runtime.](https://www.twblogs.net/a/5b826f982b717766a1e83f97?fbclid=IwAR0dPmN6q3myE_osUZvdww1Uzt7EppegKMYp8qarYxKLKZmGjY0KIqZq22U)