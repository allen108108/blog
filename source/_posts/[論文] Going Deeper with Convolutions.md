---
title: "[論文] Going Deeper with Convolutions" 
date: 2019-10-07 15:32:00
categories:
- 論文 Paper
- 卷積神經網路 Convolutional Neural Network
image: https://i.imgur.com/4p5jlmG.png
mathjax: true
---

概述
---
在此論文發表前的三年中，物件分類 / 偵測技術不僅隨著更強大的硬體、更大量的資料及更大的模型而有所進步，最主要的還是因為有著更多新的想法、演算法以及更好的網路架構所導致的。

在這篇論文中提出了一個新的網路架構 GoogLeNet 用了比 AlexNet 還要更少的參數 ( 約是 AlexNet 參數量的 1/12 ) 在 ILSVRC 2014 競賽上取得更好的成績，準確率更高。這告訴我們，現在的趨勢並非著重在更大更深的網路架構，而是如 R-CNN 一樣，結合深層網路結構與電腦視覺應用。

<!-- more -->

再者，隨著移動設備的興起，高效率的演算法變得格外重要，因此在本論文中，不僅僅針對精準度來考量模型架構，如何提高其效率也是考量的因素之一。

論文內作者會著重在一個高效率的電腦視覺深層網路架構 --- Inception。

這個名稱來自於電影 "Inception" (全面啟動)，劇中的一句話 " We need to go deeper. " 與論文 " *Network in Network* "的概念符合了這個架構的精神 。在論文中，認為 deep 有兩個層面的意義 : 
1. Inception 是一種全新的組織層次
2. 藉由引入 Inception 架構可以使整個網路更加深層。

相關工作 Related Work
---

這一個部分主要是針對論文中會使用的方法做一個簡單的簡介 : 

* 從 LeNet-5 之後，CNN結構並沒有很大的變化，基本上就是由 Conv Layer -- Normalization & Pooling Layer -- FC Layer 所構成。這樣的架構在 MNIST 或是 CIFAR 上都可以有不錯的表現，但若在 ImageNet 上，通常會利用更深、更廣的網路結構搭配 Dropout 來防止 overfitting 的發生。
* 受到神經科學的啟發，Serre 2014年發表的論文 " *Robust object recognition with cortex-like mechanisms* " 中使用了不同尺寸的 Gabor filters 來萃取不同尺寸的特徵。而此論文用的方法類似，但在 Inception 內部所有的 filters 都是學習而得，且會重複非常多次。
* 在 NIN 中利用 $1\times 1$ 卷積層來增強其泛化能力，萃取出更抽象的特徵，但在此論文中，作者要利用的是 $1\times 1$ 卷積層來達成雙重目的 : (1) 降低維度 (2) 藉此加深、加廣整個神經網路但又不會因此懲罰到性能。
* 最後在論文中採用了類似 (強化) R-CNN 的兩階段物件檢測方法 : (1) 先利用低階特徵來產生目標物件候選區域，(2)再利用 CNN 來對候選區域進行分類。 這樣的方法不但有 bounding box 可對目標物件有更精準的定位，且有 CNN 強大的分類能力。


研究動機及高層次思考 Motivation and High Level Considerations
---

要提高深度學習網路的最直覺方式就是增加它的深度 ( 層數更多 ) 及寬度 ( 每一層有更多神經元 )，這樣的方式是簡單而且安全的，尤其在處理大量的標記資料時。然而這樣的方式會產生兩個問題 : 

1. 這樣強大的網路在資料量不足的情況下容易造成 overfitting，而且要給出非常大量的標記資料來避免這種狀況往往是不切實際的。
2. 這樣的網路結構也對增加計算成本，在一般狀況下，運算成本永遠都是有限的，因此，在提高性能的前提下，計算資源如何有效分配通常會比單純增加其大小來的好。

想要解決上面兩個問題的方法是 : 「利用稀疏連接層取代全連接層 ( 不單指真正的全連接層，也包含了卷積內部的全連接 )」

這樣的方是奠基在 Arora 2013 發表的論文 " *Provable bounds for learning some deep representations.* "之上。在 Arora 的這篇論文中的結論就是當某一個 dataset 的機率分布可以用一個稀疏的網路表達時，就可以透過分析某些激活值的相關性，將相關性高的神經元聚合來或的一個稀疏表示。 ( 雖然在數學上這樣的結論需要在非常強的條件下才能實現，但在現實狀況下，它仍符合赫布理論[^註1] )



但很不幸的是，在當時的計算架構下是無法有效率的處理 non-uniformly 稀疏資料結構的，即使整個演算操作降低了100倍，但整個查詢 ( lookup ) 及快取遺失 ( cache misses ) 仍然佔據的極大的計算資源[^註2] 。



現今大多數的方式都是利用空間的稀疏性在上一層 patch 上利用卷積做運算 ( 即我們所知的 CNN )，LeNet-5中，為了打破對稱性及增強學習表現便利用了 feature maps 稀疏的連接操作來進行卷積，但到了 AlexNet 便偏好在 feature maps 上進行全連接操作。到目前為止，最先進的方法都是採取一個 uniform 的架構，利用更多 filters 以及更大的 batch size 來進行這樣的密集運算。

那麼，我們能不能在這樣的基礎下有更進一步的進展 ? 意即，有沒有一種架構在 filter 層級中具有稀疏性，又能在現今的硬體設備下進行密集運算 ?

在關於稀疏矩陣的文獻[^註3]中都認為，從稀疏矩陣的乘法來看，將稀疏矩陣合併 / 聚類 ( cluster )成相對密集矩陣會有更好的表現 ( 這也符合上面說到的赫布理論 )。這樣的結論，必然會是未來處理 non-uniform 深度學習架構下的基礎。

Inception 架構最早開始於第一作者的研究之中，該研究評估一個複雜網路拓樸結構演算法的假設輸出[^註4]。然而這研究是奠基在一些假設前提下進行的，這有點投機，但實際使用上卻也具有成效，在前期的表現上來看是優於 NIN 結構的。在經由一些調整後，作者們建立了完備的 Inception 架構，在幾篇論文[^註5][^註6]中，也證實了 Inception 在上下文定位 (context of localization) 及物件偵測 (object detection) 上是非常有用的。


值得思考的是，大多數最初的模型架構都是會被質疑的，經過多次的測試、調整才能使其接近局部最優解。這代表了一件事情 : 雖然 Inception 架構在電腦視覺上看起來是成功的，但是否可以歸因於模型的架構本身還有待商榷，這部分必須經過更多的分析驗證才能確定。

模型架構細節 Architectural Details
---

Inception 的主要概念就是如何使一個卷積電腦視覺網路能以局部最優的稀疏結構表示，而這樣的稀疏結構又可以被一些容易取得的密集部件來近似取代。

假設所謂的平移不變性 ( translation invariance ) 指的是我們必須使用局部卷積運算來建構網路的話，那麼我們要做的事情便是找到一個局部最優結構並重複它。在 Arora 的論文中便提到了一個逐層建構的網路必須要分析每一階段與其最後一層的相關性，並且將高相關性的部分聚合 (cluster) 成一個單元 ( unit )。而這單元會與下一層及上一層的單元相互連接。我們再給一個假設，如果前一層的單元都對應到輸入圖像的某個局部，那麼這些單元就會被分配、集中到 fliter banks。而較為低階具有相關性的單元則會聚焦在輸入圖像中的某一個區域。因此我們可以將些單元集中到輸入圖像的單一局部區域，而這些單元會被下一層的 $1\times 1$ 卷積層來覆蓋[^註7]。藉由$1\times 1$ 卷積層的降維功能，我們便可以用更小的區塊來對應到原輸入圖像的局部範圍 ( 更提高 Receptive Field  )



為了避免局部區域對齊[^註8]出現問題，目前的 Inception 架構都被限制在 $1\times 1$ , $3\times 3$ 以及 $5\times 5$ 的 filter 尺寸上，這樣的尺寸選擇主要是以方便性為考量，並非必要的設置。這也意味著一個建議的架構應該要由各層的 filter banks 結合為一個向量輸出並做為下一階段的輸入。除此之外，池化層在現今的 CNN 架構下仍非常有用，因此也建議在每一個階段間都設置池化層來得到附加效益。




![](https://i.imgur.com/8BbdJKk.png =500x)



上圖 (a)，是一個 Naïve 版本的 Inception 模組。
Inception 模組會一直堆疊上去，而輸出的相關性也必然會不同 : 更高度抽象化特徵會被更高層所捕捉，而它們的空間聚集程度也會隨之縮小 ( Receptive Field變小 )，此時  $3\times 3$ 以及 $5\times 5$ filter 的比例應該要增加以利捕捉這些特徵。

但這會衍生出一個大問題，即使我們 $5\times 5$ filter 的數量不多，即使是這樣一個 Naïve 版本的 Inception 模組，一直堆疊上去後的計算成本仍然相當高，如果再加上池化層，那狀況會更嚴重 : 卷積跟池化層的合併，藉由一個階段一個階段的輸出，可能在幾個階段之後就產生計算量爆炸的情形。

而這也衍生了 Inception 架構的另外一種思考 : 是否能在計算量暴增的部分進行降維 ? 經過這樣一個嵌入到低維空間的動作後，或許可以保留原圖像中局部區域內大量的資訊。但嵌入到稠密的空間中，且訊息被大量壓縮後會變得極難處理，因此最好還是如同 Arora 的建議，嵌入後應該要保持稀疏性，且訊息的壓縮會在它們必須要聚合時才會進行壓縮。在這些條件下，$1\times 1$ 卷積層便是一個用來降維的好選擇。



![](https://i.imgur.com/y7BvXR3.png =500x)



上圖 (b) 則是利用 $1\times 1$ 卷積層在計算量可能會爆炸的 $3\times 3$ 以及 $5\times 5$ 卷積層前面先進行降維。在前面我們提到了 $1\times 1$ 卷積層的雙重目的，光是這樣的設置是不夠的，在 Inception 中還加入了 ReLU 做為 activation function 來達成前述的雙重目的。

一般來說，整個 Inception 網路結構是由上述 Inception 模組一直向上堆疊而成，有時候會有 stride=2 的最大池化層會將解析度降為 1/2。從技術的角度來看 ( 為了高效率的使用記憶體來訓練 )，較低階的部分使用傳統的卷積層， 到了較高階層再使用 Inception 模組似乎是一種比較有效益的做法。( 這並非一定要如此設計，只是在作者的實作下有一些結構會造成效率不佳的狀況才會用這樣的方式做設計 )

這樣的結構之所以有用，就是它允許在每一個階段中增加神經元的數量，但卻又不會導致計算量的爆炸，這歸功於 Inception 內 $1\times 1$ 卷積層的廣泛使用，將前一階段對於較大局部區域進行的卷積結果進行降維。除此之外，這樣的設計，以直觀的角度而言，就是可以同時對不同尺度的特徵進行萃取。

也因此，我們可以利用 Inception 模組開發出一個可能表現稍微差ㄧ些，但可以降低大量計算的模型，作者們發現 Inception 內的控制因素只要經過小心謹慎的調整，可以比表現相當但未使用 Inception 模組的模型還要快 3-10 倍的速度。 

GoogLeNet
---

作者們利用 " GoogLeNet " 這個名稱來參加 ILSVRC14 的競賽，這樣的名稱最主要是對 Yann LeCun 的 LeNet 模型致敬，也將參賽所使用的模型取為 " googLeNet "。

![](https://i.imgur.com/4p5jlmG.png)

上圖是 GoogLeNet 的參數表，作者們也有使用過更寬更深的 Inception 架構放入其中，但整個模型表現也只有非常些微的增長，估計是因為雖然加入了更深更廣的模組，但相對於整體模型來說，單一模組內參數的影響相對來說小很多，因此整體來說沒有太大的增長。作者最後利用上圖的網路結構，配合不同區域抽樣技術分別做訓練，得到不同的模型，最後再進行組合 ( ensemble ) 得到最終模型。

在這張表中，輸入尺寸為 3 channel $224\times 224$ 的圖像，#$1\times 1$、#$3\times 3$ 以及 #$5\times 5$ 指的就是 Inception 內部三種不同尺寸的卷積層數量，而 #$3\times 3$ reduce 以及 #$5\times 5$ reduce 則是代表卷積層前面所做的降維數量。最後在所有降維、池化層後都會接上 ReLU 作為 activation function。

整個 GoogLeNet 是基於計算高效率以及實踐性而設計的網路結構，作者相信這樣的網路結構可以在一般獨立設備上面運作，即使這些設備的計算資源 ( 或記憶體 ) 極其有限。

整個 GoogLeNet 具參數的一共有22層 (包含池化層則有27層，加上獨立模組的層數則約100層)。在分類器前設置的 Average Pooling Layer 是參考 NIN 所設計的，後面接上一個 Linear Layer 主要只是為了方便使 GoogLeNet 可以適應其他的 label set，在此對結果並沒有太大的影響。作者發現使用平均池化會比全連接層得到更高的準確率。( 即使替換掉全連接層，dropout 層在整個網路上的重要性還是很高 )

對於深度較深的網路，在使用 Gradient Descent 上面會有很大的挑戰。在這項任務中，作者發現深度若沒有這麼深，整個模型仍然有很不錯的表現，這也表示在這些中間層所萃取出來的特徵事實上具有足夠的鑑別度。因此在 GooLeNet 中間層加入了一些輔助分類器 ( auxiliary classifier )，這些輔助分類器所計算出來的 loss 會乘上一個權重 (0.3) 加到最後的 loss 當中。一來是可以抵抗梯度消失的問題，二來這樣的效果就會跟 model ensemble 的效果類似。

關於輔助分類器有幾個需要注意的地方 : 
* 輔助分類器在整個預測過程中會被捨棄
* 輔助分類器對於整個模型的影響並不大，只需取一個即可得到類似的效果

最後，總結一下整個 GoogLeNet 中輔助分類器的架構 
輔助分類器會接在 Inception (4a) 及 (4b) 之後
1. 一個具有 $5\times 5$ filter 尺寸且 stride=3 的平均池化層
2. 一個具有 128 filters 的 $1\times 1$ 卷積層進行降維，且 ReLU 為 activation function。
3. 最後接一個具有 1024 個神經元且 ReLU 為 activation function 的全連接層
4. 全連接層帶有比例為 0.7 的 dropout layer

最後整個網路結構如下 : 

![](https://i.imgur.com/E358ImA.jpg)


訓練方法 Training Methodology
---

GoogLeNet 利用論文 " *Large scale distributed deep networks.*  "[^註9] 所提出的 DistBelief [^註10]機器學習系統進行訓練，雖然在此論文中，作者們僅使用 CPU ，作者們僅使用 CPU 實現 GoogLeNet 的訓練，但 GoogLeNet 應該可以利用高檔的 GPU 訓練，並在一周內達到收斂。

此論文中，作者們使用 SGD ( momentum=0.9，且每 8 epochs Learning Rate 下降 4% ) 進行優化，最後再使用論文 " *Acceleration of stochastic approximation by averaging.* "[^註11] 提出的 Polyak averaging 來決定最終模型。

有關於資料抽樣的部分，由於幾個月來作者們針對此競賽不斷地改變抽樣方式，也進行了各種超參數的調整，其實很難給出一個最有效率的訓練方式，更麻煩的是，有些模型是利用小尺寸輸入進行訓練，有些則是利用較大尺寸輸入進行訓練。不過還是有些準則是經過驗證過可以有不錯表現的 : 針對資料進行不同尺吋局部採樣，而這局部採樣尺寸分布於原圖的 8%-100% 之間，而長寬比限制在 $\displaystyle{\frac{3}{4}}$ 到 $\displaystyle{\frac{4}{3}}$ 之間。此外，Andrew Howard 提出的光度畸變 ( photometric distortions )[^註12]甚至可以有效避免 overfitting 的發生。





ILSVRC 分類結果
---

( 此部分主要都為實驗結果的闡述，大家有興趣再自行閱讀吧 )

結論
---

本論文提供了一個證據證明我們可以用多個完備、密集的模組來逼近一個稀疏結構，進而增進電腦視覺神經網路效能。這樣做最主要的優點是僅需增加適度的計算成本就可以使性能與淺層且較窄的神經網路相比有顯著的提升。

GoogLeNet 並沒有使用到前後關係，也沒有利用 Bounding Box 回歸方式，但仍然有著頗具競爭力的性能，這也再一次證明 Inception 架構的強大。

最後，不管是分類問題還是物件偵測，我們可以找到一個非 Inception 型態同時具有相似深度、寬度的神經網路來達到類似的成果，但計算成本將會提高非常多。這篇論文等於再一次證明了稀疏結構是可行而且有用的方法。而也說明了未來的研究可以在 Arora 的基礎上創造稀疏結構來增加效率，且也可以將 Inception 的概念應用到其他領域。

參考資料
---

1. [Going Deeper with Convolutions——GoogLeNet论文翻译——中文版](https://blog.csdn.net/Quincuntial/article/details/76457513)
2. [【深度學習經典論文翻譯2】GoogLeNet-Going Deeper with Convolutions全文翻譯](https://www.itread01.com/content/1548988391.html)
3. [SSD中的数据增强细节](https://nicehuster.github.io/2019/05/11/ssd-dataaug/)
4. [TensorFlow -- Wikipedia](https://zh.wikipedia.org/wiki/TensorFlow)


註釋
---

[^註1]: 
兩個神經元之間若常常產生動作電位 ( 同時被激化(fire) )，則此二神經元之間的連結就會變強，反之則變弱。

[^註2]: 
這一部分要去了解整個稀疏矩陣的計算方法才能知道它在講什麼，待日後有空再來了解補充吧。

[^註3]: 
U.V.Çatalyürek, C.Aykanat, and B.Uçar. *On two-dimensional sparse matrix partitioning: Models, methods, and a recipe.* SIAM J. Sci. Comput.,32(2):656–683, Feb. 2010.

[^註4]: 
這種演算法試圖以密集且容易得到的元組件來逼近 Arora 論文中的稀疏結構。

[^註5]: 
D. Erhan, C. Szegedy, A. Toshev, and D. Anguelov.*Scalable object detection using deep neural networks.*In CVPR, 2014.

[^註6]: 
R.B.Girshick, J.Donahue, T.Darrell, and J.Malik.Rich feature hierarchies for accurate object detection and emantic segmentation. In Computer Vision and Pattern Recognition, 2014. CVPR 2014. IEEE Conference on, 2014. 


[^註7]: 
這一個部份我自己的理解大概有兩個重點 
(a) Receptive Field 的重新詮釋 : 
在 Inception 內部過程中會更加提高 Receptive Field
(b) 針對以往 CNN 架構上的調整 : 
傳統 CNN 架構，單一個卷積層會使用單一尺寸的 filter size，然後隨著層數增加，適度調整 filter size 來捕捉高度特徵，這樣密集而連續的架構便是傳統 CNN 的結構。而 Inception 做了一些調整，在同一個 Inception 內同時使用不同 filter size 來進行卷積，這樣相當於在不同尺度下同時捕捉相同特徵，而這也是論文所稱的，將高相關的部分 ( 不同尺度的 filter ) 聚合成一個單元 ( 指的就是 Inception )。

[^註8]: 
Inception 跟一般的卷積過程的對齊方式不太一樣，一般卷積過程中，filter 的左上會對其圖像的左上角，然後依序滑動做內積，這樣的方式確保了 filter 始終會在 Receptive Field 內部。但 Inception 是讓 filter 的中心對齊 Receptive Field 的左上角再依序滑動做內積，這時 Receptive Field外部必須做 zero padding。

[^註9]: 
J. Dean, G. Corrado, R. Monga, K. Chen, M. Devin, Q. Le, M. Mao, M. Ranzato, A. Senior, P. Tucker, K. Yang, and A. Ng."Large Scale Distributed Deep Networks." Paper presented at the meeting of the NIPS, 2012.

[^註10]: 
DistBelief 是 Google Brain 第一代的機器學習系統，之後由 Hinton 等人進行簡化及重構而演變為現在眾所周知的 TensorFlow。

[^註11]: 
Polyak, Boris T. and Anatoli Juditsky. “Acceleration of stochastic approximation by averaging.” (1992).

[^註12]: 
光度畸變 (photometric distortions) 包含了隨機亮度（Random Brightness）、隨機對比度 （Random Brightness）、隨機色調 （Random Hue）、隨機飽和度 （Random  Saturation）以及隨機光雜訊 （Random Lighting Noise）