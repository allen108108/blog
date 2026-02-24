---
title: Tensorflow 2.0 - GPU的安裝及確認 ( Win 10 )
date: 2019-10-07 15:20:25
categories:
- 軟體 Software
image: https://www.tensorflow.org/images/tf_logo_social.png
mathjax: true
---

> 安裝前請先確認是否安裝Anaconda，若仍無安裝可參閱”[Anaconda的安裝](http://bit.ly/2ANwdnd)”

---

<!-- more -->

為了怕各種套件之間的衝突，或是安裝上面出現問題，建議在進行重大系統安裝的時候均建立新環境來進行安裝。

### Step 1. 建立一個新環境

我們首先在 Anaconda Prompt 建立一個名為 tf2.0 的新環境

```
conda create -n your_env_name python=X.X anaconda
```
我們最終要安裝的是 TensorFlow 2.0，因此至少要是 python3 的版本，行末加上 "anaconda" 是在新環境安裝時會順便裝上這個環境本身的 Anaconda Prompt、Jupyter notebook 以及 Spyder，我自己習慣或多安裝這些，這樣就不用在每一次使用的時候還要切換環境，要用哪個環境就點哪個環境的工具即可。

安裝過程會出現`Proceed ([y]/n)?`，請輸入 y 繼續安裝

建立完成以後，接著還要啟動環境才算完成，啟動環境要輸入以下命令
```
activate tf2.0
```

啟動完成之後，在你所在路徑前方會出現 (tf2.0) 則表示你已經啟動這個環境，並且已在這環境底下了。

### Step 2. 安裝 TensorFlow 2.0

在 tf2.0 環境下輸入
```
pip install tensorflow-gpu==2.0.0beta1
```
如果單純只要安裝 CPU 版本，就將上述命令中 `-gpu` 刪除即可，此外，目前的最新版本已來到 TensorFlow 2.0 RC，因此如果要安裝 RC 版本便改為輸入

```
pip install tensorflow-gpu==2.0.0-rc0 
```

RC 與 Beta 版本的不同在於，RC 版已經是正式發行版前的候選版本，在這段期間，已經不會像 Beta 版本不斷加入新功能，基本上 RC 版整個架構都已經固定，重點會擺在除錯上。

### Step 3. 安裝 cuDNN 與 CUDA

這裡我們安裝 cuDNN 與 CUDA 與先前的方式不同，經測試過後是可以使用的

在命令行輸入

```
conda install cudnn=7.6.0
conda install cudatoolkit=10.0.130
```
TensorFlow 2.0 目前支援的是 cuDNN 7.6、CUDA 10.0 版本，因此在安裝時必須指定版本進行安裝。

這裡要注意的一點是必須先安裝 cudnn，由於 conda 會將關聯依賴做更新，因此當我們安裝 cudatoolkit 的同時也會將 cuDNN 7.6 內依賴的部分進行降級。


### Step 4. 測試

```python=
import tensorflow as tf
import keras 
```

如果沒出現報錯，應該就表示已經成功安裝。
我們可以來看一下現在的版本

```python=
tf.__version__
```
然後確定一下是否有使用到 GPU

```python=
tf.test.is_gpu_available()
```

若顯示 `True` 則表示 GPU 版本是沒有問題的。