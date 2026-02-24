---
title: Win10 系統下進行 XGBoost CPU / GPU 的安裝
date: 2019-10-08 02:24:23
categories:
- 軟體 Software
image: https://h2o.ai/wp-content/uploads/2018/02/xgboost-narrow.png
mathjax: true
---

XGboost 一直是 Kaggle 個項目中前段班最愛用的演算法之一，之前就一直想要安裝，但總聽聞 XGboost 在 Windows 系統下安裝十分麻煩，在尚未要用到之前，我也就一直擺著沒有去安裝，直到最近開始接觸，便想著該來鼓起勇氣安裝 XGboost 了....

<!-- more -->

## XGBoost CPU 版本

原本思考要重新設置一個環境來安裝，避免與 tensorflow 之類的造成衝突 (我也不知道會不會?)，但礙於還是希望能在同一個環境下進行操作，未來在演算法的挑選上面也比較不會這麼麻煩，於是便還是決定在 tensorflow GPU 的環境下安裝，所幸安裝過程十分順利，目前看起來也都沒有造成任何內部衝突的狀況發生，以下便開始介紹我的安裝流程 。


### STEP 1 : 下載 XGboost 套件

XGboost 套件似乎沒有包在 Anaconda 內，因此我們無法直接從 Anaconda Prompt 下指令 `pip install xgboost` 直接安裝，必須從第三方下載套件再進行安裝。

我所使用的套件是從 [美國加州大學(UCI) Laboratory for Fluorescence Dynamics](https://www.lfd.uci.edu/~gohlke/pythonlibs/#xgboost) 中下載的，其中還是有許多版本必須選擇，我們可以到 Anaconda Prompt 執行 python 先進行版本確認

![](https://i.imgur.com/U3mWycH.png)

確認好版本後點選即可下載

### STEP 2 : 以系統管理員身分重新執行 Anaconda Prompt，並進行安裝

在安裝時可能會因為權限問題而導致安裝失敗，因此我們必須要利用系統管理員的身分來重新執行 Anaconda Prompt 來進行安裝。

![](https://i.imgur.com/FtSsSXS.jpg)

執行後我們必須在與剛剛下載的檔案同樣路徑下進行安裝，因此執行 Anaconda Prompt 後利用 `cd 下載檔案所在路徑` 指令 ( 本例將檔案下載至 C:\pythonwork 資料夾中 )

![](https://i.imgur.com/JMgm5Zs.jpg)

轉換完成即可進行安裝 `pip install xgboost-0.90-cp36-cp36m-win_amd64.whl`

<font color="#dd0000">**[ Remark ] 必須要有副檔名，否則安裝會出現錯誤**</font>

### STEP 3 : 安裝完成後進行簡單測試


安裝完畢後，執行 python ，`import xgboost` 確認會不會出現錯誤

![](https://i.imgur.com/rJsRGN2.jpg)

若成功 import 便可算安裝成功。

由於我是在 tensorflow 環境下進行安裝，因此我也有多測試看看 tensorflow、keras  能否順利運作。



---

## XGBoost GPU 版本

照著上面的流程裝完以後自得意滿，後來發現裝的是 CPU 版本，便開始著手研究 GPU 版本的安裝，怎知，這才是真正魔鬼地獄的開始......

花了將近一整天的時間研究各種安裝方式，終於把所有問題處理好，以下提供安裝流程及所遇到的狀況排除方式給需要的讀者參考

### STEP 1 :下載xgboost原始碼

首先要從 https://github.com/dmlc/xgboost 將整個原始碼下載下來解壓縮

![](https://i.imgur.com/lgVg7Ls.png)
(圖片取自 : https://www.itread01.com/content/1543742352.html)

[ Remark ]

 
當我照著上述教程進行原始碼下載，到後面的安裝會出現錯誤，錯誤的原因似乎是指出有檔案找不到 ??

   ![](https://i.imgur.com/YMIyz8j.png)
     
的確，按照上面教程的方式，整個 dmlc-core 的資料夾內的資料全部都沒有下載下來，我只好將 dmlc-core 整包單獨下載下來在解壓縮到源碼文件裡面的 dmlc-core 資料夾內。
     

### STEP 2 : 將已編譯好的 xgboost.dll 文件下載後放入源碼中

一般來說，正常的安裝是需要自行編譯的，但網路上總有大神們能幫我們把這些前置作業處理好。我們只需從 http://ssl.picnet.com.au/xgboost/ 選擇 GPU enabled 裡的 xgboost.dll 下載下來放入源碼文件 xgboost\python-package\xgboost 路徑下。

### STEP 3 : 執行安裝

執行 Anaconda Prompt，並將路徑設在 xgboost\python-package 下，執行 `python setup.py install` 進行安裝

### STEP 4 : 進行 XGBoost GPU 的測試

Xgboost GPU 不像 TensorFlow GPU 一樣可以很清楚的看的出來現在是不是由 GPU 在進行運算，一般建議的方式是利用源碼文件中的 `benchmark.py` 檔案進行測試。( 在xgboost\tests 目錄下執行 ` python benchmark_tree.py --tree_method gpu_hist`  )

基本上就是，只要可以執行 `gpu_hist` 這樣的參數設計，就表示已經可以使用 XGBoost GPU 了。

[ Remark ]

然而這樣簡單的步驟還是出現了問題，在我們下載的源碼中，並沒有 `benchmark.py` 這個檔案，取而代之的是 `benchmark_linear.py` 及 `benchmark_tree.py` ，記得請使用後者進行測試。

另外我也找到另外一個測試的 code

```python=
import xgboost as xgb
from sklearn.datasets import load_boston

boston = load_boston()

# XGBoost API example
params = {'tree_method': 'gpu_hist', 'max_depth': 3, 'learning_rate': 0.1}
dtrain = xgb.DMatrix(boston.data, boston.target)
xgb.train(params, dtrain, evals=[(dtrain, "train")])

# sklearn API example
gbm = xgb.XGBRegressor(silent=False, n_estimators=10, tree_method='gpu_hist')
gbm.fit(boston.data, boston.target, eval_set=[(boston.data, boston.target)])
```
若可以執行也應該可以確定 XGBoost GPU 運作是沒有問題的。


