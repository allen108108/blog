---
title: "Lookahead optimizer 於 Dogs vs. Cats 貓狗辨識上之實作"
date: 2019-10-08 01:18:05
categories:
- 實作 Implementation
image: https://i.imgur.com/O0s57DT.png
mathjax: true
---

拖稿拖了一段時間，最近才開始著手寫這一篇文章，不過這個時間來寫也是挺好的，上一周的機器學習馬拉松期末考也正好實作了一番，趁著記憶猶新來把整個演算法以及實作過程做一次總整理。

<!-- more -->

Optimizer
---

優化器 (optimizer) 作為一個超參數的角色，在深度學習的訓練過程中，具有主導整個訓練效能的地位，找到一個好的優化器，便可以讓 model 的表現更好，預測準確度更高。

然而在過去幾年之間，Adam 在優化器中獨占鰲頭，幾乎是優化器選擇的預設選項。然而 Adam　並非這麼的無懈可擊，它的收斂性在近幾年來備受質疑，在某些狀況下甚至不及 SGD ( Stochastic Gradient Descent ) 來的優秀。 

然而在今年七月 (2019.07)， Department of Computer Science, University of Toronto 發表了一篇名為 " *Lookahead Optimizer: k steps forward, 1 step back* " 的論文，介紹了一種新的優化方法 Lookahead Optimizer，試圖彌補 Adam 的不足。

下一個部份我們會介紹整個優化演算的概念，這邊有一個值得注意的是這篇論文的作者群。

除了第四作者 Jimmy Ba 是當初 Adam 論文 " *Adam: A Method for Stochastic Optimization* " 的作者之一外，Hinton 也以第三作者的身分參與了此論文的研究，因此這篇論文的重要性不言可喻。

Lookahead Optimizer: k steps forward, 1 step back
---

現今絕大多數成功的深度學習都是利用 Stochastic Gradient Descent ( SGD ) 來進行學習的。而近年來想要比 SGD 有更好表現的演算法大概分為兩種類型 : 

1. 適應性學習率的學習方式 : 如 AdaGrad 及 Adam
2. 以物理為概念加速的學習方式 : 如 Polyak heavyball 及 Nesterov momentum

上述兩種方式都會考量過去的梯度來進行快速的收斂，但相對的要付出很多的超參數調整成本。

而此論文作者提出了一種新的、獨立於上面兩個方向的優化方式 -- Lookahead -- 透過兩組權重，slow weight 與 fast weight，的迭代更新來進行優化。首先，我們會先指定一個 optimizer ( SGD or Adam ) 用來對於 fast weight 進行更新，進行 fast weight 的 $k$ 次更新後，利用最後一次更新後的方向來更新 slow weight 的方向，如此反覆更新兩組權重，直至最後收斂。



![](https://i.imgur.com/wcE3fEd.png)



上圖為整個 Lookahead Optimizer 的演算過程 : 

Step 1 : 先設定 $\phi_0$ 的初始值，以及選定 Loss function
Step 2 : 確定 $k$值、slow weight 的更新步伐 $\alpha$ 以及 optimizer $A$
Step 3 : $\theta_{1,0}\leftarrow\phi_0$ 或 $\theta_{t,0}\leftarrow\phi_{t-1}$ 
Step 4 : 利用 optimizer $A$ 由 $\theta_{1,0}$ 更新到 $\theta_{1,k}$ 或由 $\theta_{t,0}$ 更新到 $\theta_{t,k}$ 
Step 5 : $\phi_1\leftarrow\phi_0+\alpha(\theta_{1,k}-\phi_0)$ 或 $\phi_t\leftarrow\phi_{t-1}+\alpha(\theta_{t,k}-\phi_{t-1})$
重複 Step 3 - Step 5 直至收斂。



![](https://i.imgur.com/O0s57DT.png =400x)



上圖展示了 ResNet-32 在 CIFAR-100 中訓練後期的情況，在後期由於梯度已經非常小，因此從右上的圖中可以發現 SGD 更新速度極為緩慢 (fast weight)，還在慢慢探索更優化的解。但從紫色的路徑 (slow weight) 來看， Lookahead 會比較積極的往更優解的方向前去。此時 slow weight 會比 fast weight 更接近最優解。

Lookahead Optimizer for Keras
---

已經有人將 Lookahead 實現在 Keras 上 ( https://github.com/bojone/keras_lookahead )，作者在介紹 Lookahead 在 keras 上的實現時有特別說到這是一種嵌(侵)入式的實現方式。

在自己的項目裡面放進作者 github 裡面的 lookahead.py 檔案後，在自己的 code 內 import 即可

```python=
from lookahead import Lookahead
```

當模型構建好以後，我們先指定一個 optimizer (這裡用的是 SGD)，之後再嵌入 `Lookahead()` 即可

```python=
model.compile(optimizer=optimizers.SGD(lr=0.0001,momentum=0.9) , 
              loss='binary_crossentropy' , 
              metrics=['accuracy'])
lookahead = Lookahead(k=5, alpha=0.5) # Initialize Lookahead
lookahead.inject(model) # add into model
```


於貓狗辨識上實作 Lookahead
---

這是機器學習百日馬拉松期末的考題，Data 內容與 Kaggle 上的 Dogs vs. Cats 略有不同，不過這不是本文討論的重點，因此前面有關 Data Preprocessing 的部分就先跳過。


這裡我直接調用 VGG19 model，但接上我們自己的分類器，除此之外，所有的權重我們都重新 train 過一次。

![](https://i.imgur.com/P0TQbem.png)

我們可以在 test 上面取得不錯的分數

![](https://i.imgur.com/F6G8HGa.png)

而這樣的 model 在 kaggle 的 public score 可以達到 0.99802
( 這是重新 train 過的成績，之前甚至有達到 0.99882 的成績，排名27 )