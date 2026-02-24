---
title: "[論文] YOLO9000 : Better, Faster, Stronger" 
date: 2020-02-05 11:47:58
categories:
- 論文 Paper
- 物件偵測 Object Detection
image: https://i.imgur.com/kHROlTh.png
mathjax: true
---


概要
---

在這篇論文中，作者們提出了最先進的即時物件偵測系統 YOLO9000 ，可以偵測超過9000 種物件。首先，針對YOLO進行了各種改善，提出了YOLOv2 。YOLOv2 不管在 PASCAL VOC 或是 COCO 資料集中都有最佳的表現。經過最新穎、多尺度的訓練，YOLOv2 可以運行在各種不同尺寸的資料中，進而在速度與精準度上進行權衡。

<!-- more -->

作者在這篇論文中利用一種結合物件與分類的聯合訓練方法，在 ImageNet 與 COCO 資料集中同時對 YOLO9000 進行訓練。經過這樣的訓練方式，YOLO9000 可以預測出沒有被 Label 過的資料，也可以即時進行 9000 多種物件的偵測。


簡介
---

一個通用的物件偵測系統應該要夠快速、準確且能辨識出各式各樣的物件。自從引入神經網路後，整個檢測框架的速度及準確度都已經大幅度提升，但跟分類任務 (classification) 或是標籤任務 ( tagging )來比較，物件偵測的數據集相對受到限制，僅有數十到數百個 label ，也因此整個物件偵測可適用的對象數也受到限制。

最終我們希望的還是物件偵測可以如同分類任務一樣可以偵測多種類的能力，但是我們要拿來訓練的資料集的標註動作遠遠比分類資料及需要更大的標註成本，因此作者在這篇論文提出了一個新的方法可以利用龐大的分類資料集來增強物件偵測系統的能力，這樣的方法利用階層視角 ( hierarchical view ) 來看分類資料集，便可以允許我們將不同的分類資料及整合在一起。

除此之外，這篇論文更提出了一個聯合演算法，該演算法允許我們同時利用物件偵測及分類資料集對物件偵測系統進行訓練，也利用這樣的演算法，訓練出 YOLO9000，可以即時偵測 9000 多種不同的物件。

總結來說，作者們利用改良過後的 YOLO 演算法訓練出了 YOLOv2，再利用上述的資料組合以及聯合演算法對 ImageNet 以及 COCO 兩種資料集同時進行訓練。

這些程式碼以及預訓練模型均可以於作者的網頁中取得 : http://pjreddie.com/yolo9000/


Better
---

與當下 SOTA 的物件偵測系統相比，YOLO 仍然存在許多缺點。從 YOLO 的實驗分析中可以發現，其 Recall 值較 Faster R-CNN 低，且 YOLO 出現不少的定位錯誤。也因此，作者們將重點放在改善 Recall 與定位，並且同時維持其分類的準確度。

在 computer vision 領域中，性能越好的網路結構通常是很大、且深度很深的網路，但在 YOLOv2 中，作者反其道而行，將網路結構簡單化，使其更易於學習，同時也加入了很多新穎的概念於其中以增強其性能。

![](https://i.imgur.com/LVfiBEU.png)

### 批量標準化 Batch Normalization
Batch Normalization (以下簡稱 BN) 是現在深度學習常用的方法之一，當我們每經過一層神經網路後，資料的分佈必然產生變化，當我們的層數越來越多，資料的分佈會變化的非常難以預測，這時候訓練會變得難以收斂。

BN 就是在每一層中間加上一個標準化，讓每一個經過神經網路的資料都可以重新標準化過，這樣可以使得資料的分佈穩定，以利訓練的收斂速度加快。

（但事實上，BN 的研究很多，使否真的是因為上述原因導致收斂加速也眾說紛紜，但上述說法是最直觀且易理解的方式，相關討論可以參考 ： *[知乎-深度学习中 Batch Normalization为什么效果好？](https://www.zhihu.com/question/38102762)*）


### 高解析度分類 High Resolution Classifier

以往的分類模型訓練都將輸入圖像限制在 $256\times 256$ 以下來進行。YOLO 在 $224\times 224$ 的圖像上訓練分類器，然後提高解析度到 $448\times 448$ 上進行物件偵測。這表示整個模型必須在偵測的過程中同時調整成高解析度進行偵測。

在 YOLOv2 上，作者將分類器的輸入尺寸調整為 $448\times 448$ 在 ImageNet 上進行 10 epochs 的訓練。這樣可以使得分類器可以更適應高解析度的圖像，進而在分類上可以有更好的效果。之後再針對整個偵測網路進行 fine tune，最後可以提昇 4% 的 mAP。


### Anchor Box 卷積 Convolution With Anchor Boxes

在 YOLO 上，最後使用全連接層來進行 Bounding Box 座標的預測，但在 Faster R-CNN 中使用 Region Proposal Network (RPN) 來預測 Anchor Boxes 的偏移量與 confidence。使用偏移量預測來取代座標預測，這樣的方法簡化問題，也讓整個網路更容易學習。

YOLOv2 採用了類似的做法，拿掉原本 YOLO 的全連接層，改由 Anchor Box 來預測 Bounding Box，除此之外，也拿掉一個 Max Pooling Layer 來讓網路卷積部分可以輸出更大解析度的圖像，但作者同時也將原本 $448\times 448$ 的輸入縮成 $416\times 416$，主要的原因是要讓最後的 feature map 長寬為奇數。

奇數的長寬可以確保一些較大物件的中心可以在正中央，而不是在四個位置中的其中一個，以便於訓練。若使用 $416\times 416$ 的輸入圖像，則最後的 feature map 的大小會是 $13\times 13$

使用 Anchor Box 時，作者們將位置的預測與分類的預測分開處理，針對每一個 Anchor Box 都進行類別的預測。

跟 YOLO 一樣，YOLOv2 也會進行 ground truth IOU 的計算，配合每一個 Anchor Box 的類別預測來計算出該類別的條件機率。

$$
P_r(Class_i|Object)\times Confidence\\
=P_r(Class_i|Object)\times P_r(Object)\times IOU_{pred.}^{truth}\\
=P_r(Class_i)\times IOU_{pred.}^{truth}
$$

使用 Anchor Box 來進行預測，針對每一個圖像可以一次預測超過 1000 個預測框 （YOLO 僅能預測 98 個），且召回率達到 88% (YOLO 為 81%)，雖然 69.2% mAP略為下滑 （YOLO 為 69.5% mAP ），但整體表現來看仍然進步許多。 


### 維度聚類分析 Dimension Clusters

現在問題來了，這些 Anchor Box 均為先驗框，應該由使用者進行人工選取，雖然每一個Anchor Box 都會預測偏移量來進行 Bounding Box 的預測，但是如果 Anchor Box 選得夠好，相對來說也會使整個網路的學習更簡單。

與 faster R-CNN 不同，這篇論文利用 k-means 作為 Anchor Box 的決定方式，這樣的方法相較人工挑選來說更為客觀。然而， k-means 必須進行距離的計算，用一般的歐式距離會導致大候選框相對於小候選框產生較大的誤差，系統會傾向於選出小框，但這不是我們想要的結果，我們需要的是挑出 IOU 相對較大的候選框，因此必須重新定義 k-means 所需要的距離公式 ：

$$
d(\text{box,centroid})=1-IOU(\text{box,centroid})
$$

經由這樣的設計，最後選出 $k=5$ 時，可以得到蠻高的 IOU ，也可以得到高召回率及高 mAP。


<img width=500 src="https://i.imgur.com/Z1m5uOB.png" >




這樣的方法選出的 5 個 anchor box 平均 IOU 比 Faster R-CNN 手動選取的 9 個  anchor boxes 還要高上一點，如果利用這樣的方法選擇 9 個 anchor boxes 則會比 Faster R-CNN 的 anchor box 平均 IOU 高出非常多。

![](https://i.imgur.com/yEclzN9.png)

### 直接定位預測 Direct Location Prediction

前面有提到，YOLOv2 不對 Bounding Box 直接做座標預測，而是預測 Anchor Box 的偏移量。如果借鑑 Faster R-CNN 中 RPN 的算法我們可以得到 ( 這邊應該是 $+$ 號而不是論文寫的 $-$ 號 )：

$$
x=(t_x*w_a)+x_a\\
y=(t_y*h_a)+y_a
$$

只要我們能夠預測出偏移程度 $t_x,t_y$ 便可計算出中心 $(x,y)$ ，這樣的方法看起來很合理，但事實上卻會造成訓練難以收斂。最主要的原因在於 $t_x,t_y$ 並不受限，會使在一個給定 anchor box 中心 $(x_a,y_a)$，就可以預測中心在圖像上任何一點的邊界框，這樣在訓練上必須花很長一段時間才能開始收斂。

因此在 YOLOv2 上採取跟 YOLO 一樣的想法，每一個 anchor box 僅針對中心落在此網格內來做預測。要達成這樣的方法便是以網格左上角位置 $(c_x,c_y)$ 為基準，並將偏移程度 $t_x,t_y$ 限制在 $0-1$ 之間 ：

$$
b_x=\sigma(t_x)+c_x\\
b_y=\sigma(t_y)+c_y\\
b_w=p_we^{t_w}\\
b_h=p_he^{t_h}\\
P_r(\text{object})*IOU(b,\text{object})=\sigma(t_o)
$$


<img width=500 src="https://i.imgur.com/JsDTCUt.png" >




從上式我們可以知道，YOLOv2 會預測五個值 $(t_x,t_y,t_w,t_h,t_o)$，從這五個值再來求出邊界框及condidence。藉由這樣的限制，讓整個網路的學習變得更加容易，並且邊界框中心定位準確度也提高了 5%。

**[ 補充 - 利用 anchor box 預測邊界框的流程 ]**

截至目前為止，論文大致上將利用 anchor box 來預測 bounding box 的過程解釋完畢，但是看了幾次還是非常的難以理解，因此特別挪出一個部分來做一個補充。

在 YOLOv2 中，整個預測流程大致上如下 ：

1. 利用 k-means 計算出所需要的 anchor box 數量，同時也會給出這些 anchor box 的大小為何。
2. 當 YOLOv2 經過一連串的卷積過程後，輸出的 feature map 大小為 $13\times 13$，在每一個網格內利用上述過程計算出來的 anchor box 進行邊界框的預測，也就是說，最終一共會有 $13\times 13\times n_{anchor}\times n_{classes}$ 個機率預測。
3. 仍然使用 Non-Maximun Suppression (NMS, 非極大抑制法) 來選出最適合的邊界框。

**[ 補充 - 利用 anchor box 預測邊界框的注意事項 ]**

由於 Anchor Box 是經由資料集計算出來，因此 Anchor Box 的選擇絕對會影響模型本身的 performance，一般我們在使用 YOLOv2 (或v3) 都會直接使用預設的 Anchor Box 參數，如果成效不佳，應該嘗試重新計算 Anchor Box。

另外，我將在文末 APPENDIX 再來補充 k-means 計算 anchor box 的過程。


### 精細特徵 Fine-Grained Features

在 YOLOv2 的整體架構中 （ 後面會提到 ），經過五次 MaxPooling 後的特徵圖大小為 $13\times 13\times 1024$，這對於較大物體的檢測已然足夠，但作者們仍希望整個模型可以在小尺寸的物體偵測上能夠有很好的表現，那麼就必須要有更精細的特徵 （Fine-Grained Features）萃取。


![](https://i.imgur.com/kHROlTh.png)


在 YOLOv2 中使用了「類似」ResNet 中 shortcut 的做法，論文內稱為 "passthrough"，在最後一個 Maxpooling 的輸入層，大小為 $26\times 26\times 512$，經過 reshape 後跟最後的特徵圖 concat 在一起形成一個 $13\times 13\times 3072$ 的特徵圖。


其中，上圖的 Passthrough Module 構造如下圖所示，它將相鄰的部分作為深度疊加上去，這樣便能在縮小的特徵途中仍然保有其細部特徵。


![](https://i.imgur.com/qZngsZB.png)



### 多尺度訓練 Multi-Scale TRaining

前面也有提到，作者將輸入尺寸設定為 $416\times 416$，但由於 YOLOv2 已經沒有全連接層，因此並不需要固定輸出的尺寸，因此，為了讓整個模型可以適應更多樣化的圖像尺寸，作者在訓練中間加進了一點小 trick : 在訓練過程中不斷更改輸入圖像的尺寸。

由於整個模型最後輸出的特徵圖為原尺寸的 $\frac{1}{32}$，所以在訓練過程中，讓模型每訓練 10 epochs 就從下列幾種尺寸 (32倍數) 中隨機選擇輸入尺寸 : $\{320,352,...,608\}$



![](https://i.imgur.com/N1HsG95.png)



這樣的訓練方式迫使模型可以在各種尺寸上都有不錯的預測表現。小尺寸的輸入圖像，YOLOv2 可以在極短的時間內進行預測，也因此，整個模型可以輕易地在速度與精準度上進行權衡。

而在高解析度的圖像上，YOLOv2 更有著極高的 mAP 表現，儘管速度上稍微慢了一些，但亦有 40 fps，仍可達到即時預測的要求。

### 更進一步的實驗 Further Experiments

(略)


Faster
---

除了維持高準確度外，我們也一樣希望整個模型的預測是快速即時的。然而現今的物件偵測系統大多依賴 VGG-16 作為 feature extractor。的確，VGG-16 是一個非常強大的分類器，但它的網路結構過於龐大且複雜。

YOLO 採用的是較快的 GoogLeNet 架構，雖說整體 mAP 表現較 VGG-16 差一些，但是卻換來更快速、更少的預測運算。

YOLOv2 中，使用的是一個全新的架構 : Darknet-19，具有19層卷積層及5層最大池化層的網路結構，類似於 VGG-16，但融合了 NIN (Network in Network) 的平均池化進行預測，以及使用 $1\times 1$ 卷積層來降維。

當然也加進前面提過的 BN 來穩定訓練以及對模型進行正規化。


![](https://i.imgur.com/oCGFUsS.png)

整個模型的訓練分為三個部分 : 

1. 分類訓練 : 首先利用 ImageNet 對 Darknet-19 做訓練，這個階段輸入圖像尺寸為 $224\times 224$，選用 SGD 作為 optimizer，並且搭配一般會使用到的資料增強方式以及 Learning Rate 的遞減來進行 160 epochs 的訓練。
2. fine-tune : 將輸入尺寸調整為 $448\times 448$ 重新進行模型的 fine-tune，共 10 epochs。
3. 偵測訓練 : 這個階段，作者將 darknet-19 調整為物件偵測模型，調整的方式為 (1) 拿掉最後一個卷積層以及整個分類器 ，(2) 接上 3 個 $3\times 3\times 1024$ 以及一個 $1\times 1\times n_{classes}$ 的卷積層，也別忘了前面提到的 passthrough ，在這個階段一共訓練 160 epochs，一樣用到資料增強以及 Learning Rate 的遞減。

以下我整理出 Classification Model 以及 Detection Model 的差異


![](https://i.imgur.com/7Ey9ADk.png)

右邊 Detection Model 中紅色部分即是與 Darknet-19 不同之處，Route 是一個連接的概念，將第 16 層的輸出連接到這邊，然後利用 Reorg 調整維後再度用一個 Route 把第 27 與 24 兩層連接再一起，這一整組就是 Passthrough。

比較值得注意的地方是紅框處，這是論文內沒有提到的部分，但如果去翻閱 Darknet YOLOv2 的[ config ](https://github.com/pjreddie/darknet/blob/master/cfg/yolov2.cfg)檔案會發現多了一層 $1\times 1\times 64$ 的卷積層在 Passthrough 過程中，原因我認為大概也是進行一次降維，使用 $1\times 1$ 卷積也能整合所有跨 channel 的訊息。


Stronger
---

論文的最後一部分就是提出了一個聯合物件偵測及分類資料及共同訓練的演算法，簡單來說，當訓練過程中看見偵測樣本，就會利用整個 YOLOv2 的 Loss function，意即分類誤差項及邊界框誤差項部分，來進行 BP (Backpropagation, 反向傳播法)，如果遇到分類樣本就僅使用 Loss function 中的分類誤差項進行 BP 。

要混合使用物件偵測與分類這兩種不同任務的資料集，最首要會遇到的問題就是物件偵測資料集在分類標註上面通常使用廣泛的標籤，如 : 狗....，但分類資料集的標籤會更加的細緻，如:諾福克 (Norfolk terrier)、約克夏 (Yorkshire terrier)、貝靈頓 (Bedlington terrier) (這三種都是㹴犬)...。

再者，在模型的分類器部分，經常使用 softmax 來計算最後的機率分布，但 softmax 的前提是類別間必須是互斥的。這樣的前提也是混和資料集訓練的一個困難點，因為 「狗」跟「約克夏」並非互斥的集合。

想要混用這兩種資料集，就必須先整合標註上的歧異。

### 階層式分類 Hierarchical Classification

WordNet 是一個語言庫，它將名詞間的關係建構起來，而 ImageNet 也是參照 WordNet 進行標註。然而，語言十分的複雜，每一個名詞也無法單純歸在某一類別之下，如 : 「狗」(dog)同時是「犬」(canine)類別以及「家畜」類別之下。也因此，WordNet 並非是一個樹狀結構，而是一個有向圖結構 (directed graph)。

作者們並不採用整個 WordNet 的圖結構，而是從中抽取其視覺名詞重新製作一個樹狀結構。

每一個視覺名詞都可以循著一條 ( 或多條 ) 路徑到達 root ( physical object物理物件 )，如果每一個名詞僅一條路徑到達 root 者便先加入樹狀結構中，若有多條路徑則採用最短路徑。依照這樣的方式建構出一個 WordTree，一個視覺概念的階層式模型 (Hierarchical Model)。

一旦 WordTree 被建構出來，那麼我們便可以計算出每一個節點之機率以及下位節點之條件機率 : 

$$
P_r(\text{Norfolk terrier})
=P_r(\text{Norfolk terrier}|\text{terrier})P_r(\text{terrier}|\text{hunting dog})*\cdots\\*P_r(\text{mammal}|\text{animal})*P_r(\text{animal}|\text{physical object})
$$

且

$$
P_r(\text{physical object})=1
$$

原本的 Label Space 是 1000 維的空間，現在引進了 WordTree 後，會變成一個 1369 維的空間。而前面有提到，softmax 必須要在類別互斥的前提下進行，因此在機率計算上面也要有所改變，一個節點的所有下位詞都會進行一次 softmax。


<img width=500 src="https://i.imgur.com/0TLQbYX.png" >



這樣的作法，我們提高了 label 維度，但準確度下降不多。再者，若我們給出一張不確定類別的圖象 ( 一張看不出來種類的狗圖片 ) 進行預測，可能在下位詞 ( 各種類別的狗 ) 的預測機率會很低，但在上位詞 ( 狗 ) 的部分可以有著較高的預測機率。


上述的分類預測上可以有著不錯的預測，在物件偵測上仍然可以用這樣的方式訓練。

我們會給定一個閥值，首先偵測模型上會先預測 $P_r(\text{physical object})$，來確認是否存在待預測的物件，之後隨著 WordTree 往下遍歷每一個類別，並給給出每一個節點的機率，分裂時會往機率較高的節點往下走，最後給出一個高於閥值的節點類別作為預測類別。

### 利用 WordTree 結合資料集 Dataset combination with WordTree 
### 聯合分類及偵測演算法 Joint classification and detection



既然 WordTree 可以擴展 ImageNet 的標註，那麼也可以用來結合不同的資料集，為了可以訓練出一個大型的物件偵測系統，論文中利用 WordTree 來結合 COCO 資料及以及 ImageNet 的前 9000 個標註類別資料，這樣龐大的資料集一共有 9418 個類別。

![](https://i.imgur.com/jaS0Qaq.png)

由於 ImageNet 相對來說比 COCO 資料集大非常多，作者有針對 COCO 進行 Oversampling ( 超取樣、過取樣 )，來讓兩者的比例接近 4:1。

利用這樣的資料集來對 YOLOv2 進行訓練，但原本選擇 5 個 Anchor Boxes 改為 3 個。

若今天模型處理「偵測樣本」時，對於分類誤差我們只會分配給相同級別以上的類別，舉例來說，今天如果 label 是 「狗」，那我們便只會將分類誤差往上分配，而不會配給「狗」以下的節點。這其實很直覺，因為以下的節點該怎麼分類我們完全沒有資訊。

![](https://i.imgur.com/Bxfvcgd.jpg)

若今天要處理的是「分類樣本」時，我們只需要處理 Loss function 中的分類誤差項即可，因此，找出擁有此分類機率最高的邊界框，然後在 WordTree 上計算誤差即可。

除此之外，論文中還給了一個條件，預測邊界框必須與真實邊界框有大於 $0.3$ 的 IOU。

經過這樣的聯合訓練後，YOLO9000 可以預測超過 9000種物件，並且在半監督學習的條件下學習到一些成果。(我不會說它非常厲害，因為事實上它的 mAP 也還不到 20%)




結論 Conclusion
---

這篇論文中提出了兩個即時的物件偵測系統--YOLOv2 以及 YOLO9000。

YOLOv2 可以在多種尺寸圖像中運行，並且在速度與精準度之間取得平衡; YOLO9000 則是提供一個即時框架，利用物件偵測及分類的聯合訓練演算法以及 WordTree 資料集來偵測多達9000種物件。

WordTree 的概念可以讓分類標註提供更大的運用空間，並且可以利用來進行弱監督學習，也可以利用這樣的概念結合各種不同任務的資料集，對於分類有很大的助益。


APPENDIX
---

### A. 如何利用 K-means 進行 Anchor Box 選取

簡單一句話來解釋，我們將所有資料的人工標註，embedding 到一個二維 W-H 空間，一個維度代表寬度 W，一個維度代表高度 H，之後在這個二維空間中進行分群，找出各集群的中心 $(w_i,h_i)$。

我們進行人工標註應該還會有中心點的資料，但在這邊不採用的原因是因為中心點不會固定，因此我們只將重點放在 anchor box 的長寬即可。(不過標註中心也不是完全無用，底下距離定義需要用到，在計算距離時會共用標註中心點)

再來，因為 K-means 是利用距離來找出集群中心，根據前面的討論，我們將標註框與集群中心的距離定義為 : 

$$
d(\text{box,centroid})=1-IOU(\text{box,centroid})\\
\Longrightarrow d\big((x_j,y_j,w_j,h_j),(x_j,y_j,w_i,h_i)\big)=1-IOU\big((x_j,y_j,w_j,h_j),(x_j,y_j,w_i,h_i)\big)\\
\text{where } i=1,\cdots,n\text{ and }j=1,\cdots,m
$$

$n$ 是我們進行 k-means 一開始必須指定的集群數量，也就是 anchor box 的數量，而 $m$ 是我們標註資料的數量。

整個流程大約如下 : 

1. 先人工指定我們要的 anchor box 數量 $n$
2. 隨機選取一組集群中心 $\{(w^0_i,h^0_i)\}_{i=1}^n$ 作為初始中心。
3. 計算每一筆資料與 $\{(w^0_i,h^0_i)\}_{i=1}^n$ 的距離，與中心 $(w^0_i,h^0_i)$ 距離最近的視作一個集群，此時我們可以得到以 $\{(w^0_i,h^0_i)\}_{i=1}^n$ 為中心的 $n$ 個集群
4. 重新選定集群中心，計算的方法是利用平均值 :
$$
w^k_i=\frac{1}{N_i}\sum_{N=1}^{N_i}w^{k-1}_t\\
h^k_i=\frac{1}{N_i}\sum_{N=1}^{N_i}h^{k-1}_t\\
\text{where } N_i\text{ is the number of }i_{th}\text{ cluster}  
$$
5. 重覆上述步驟直至收斂



### B. Loss Function of YOLOv2

在閱讀整篇論文的時候，其實會覺得困惑，為什麼從頭到尾都沒有出現 Loss function ?
其實從理論來看，整個 YOLOv2 的 Loss function 與 YOLO 相差無幾

![](https://i.imgur.com/nromwhe.jpg)

$(x_i,y_i)$ 為 anchor box 中心點 （亦為 ground truth 中心點）,
$S=13$, $B=\text{number of anchor box}=5$,
$\lambda_{\text{coord}}$、$\lambda_{\text{noobj}}$ 均比照 YOLO 不更動,  $\mathbb{I}_{i}^{obj}$ 與 $\mathbb{I}_{ij}^{obj}$ 亦與 YOLO 表示意義相同．其餘參數也幾乎都沒有什麼變動。

但是事情沒有這麼簡單，大架構如此，但卻還是有一些細節沒有呈現出來，最基本的就是，到底 anchor box 要怎麼去計算偏移值，在這裡便沒有辦法呈現出來。

在 “ *[YOLO v2 损失函数源码分析](https://www.cnblogs.com/YiXiaoZhou/p/7429481.html)* ” 一文中，便從 Darknet Source [code](https://github.com/pjreddie/darknet/blob/master/src/reorg_layer.c) 中直接分析出 YOLOv2 的 Loss function ：

$$
\lambda_{coord}\sum_{i=0}^{w\cdot h}\sum_{j=0}^{n}\mathbb{I}_{ij}^{obj}(2-w_i\cdot h_i)\big[(x_i-\hat{x}_i)^2+(y_i-\hat{y}_i)^2+(w_i-\hat{w}_i)^2+(h_i-\hat{h}_i)^2
\big]\\
+\lambda_{obj}\sum_{i=0}^{w\cdot h}\sum_{j=0}^{n}\mathbb{I}_{ij}^{obj}(C_i-\hat{C}_i)^2\\
+\lambda_{noobj}\sum_{i=0}^{w\cdot h}\sum_{j=0}^{n}\mathbb{I}_{ij}^{noobj}(C_i-\hat{C}_i)^2\\
+\lambda_{class}\sum_{i=0}^{w\cdot h}\sum_{j=0}^{n}\mathbb{I}_{ij}^{obj}\sum_{c\in class}(p_i(c)-\hat{p}_i(c))^2\\
+0.01\cdot\sum_{i=0}^{w\cdot h}\sum_{j=0}^{n}\mathbb{I}_{ij}^{noobj}\big[(p_{jx}-\hat{x}_i)^2+(p_{jy}-\hat{y}_i)^2+(p_{jw}-\hat{w}_i)^2+(p_{jh}-\hat{h}_i)^2
\big]\\
\text{where }w=h=13\text{ in YOLOv2},\\
n\text{ is the number of anchopr box}
$$

上式第一行對應著 YOLO Loss function 的第一二行，上式第二、三行對應著 YOLO Loss function 的第三、四行，上式第四行則是對應著 YOLO Loss function 的最後一行。

最後一項比較特別，YOLOv2 在訓練前期 （ 前12800個樣本 ），會去計算「未偵測到目標物件」的 Loss。





參考資料
---

1. [YOLOv2--論文學習筆記（算法詳解](https://www.twblogs.net/a/5bafd96b2b7177781a0f6394)
2. [目标检测|YOLOv2原理与实现(附YOLOv3)](https://zhuanlan.zhihu.com/p/35325884)
3. [YOLOv2、v3使用K-means聚类计算anchor boxes的具体方法](https://blog.csdn.net/shiheyingzhe/article/details/83995213)
4. [YOLO v2 损失函数源码分析](https://www.cnblogs.com/YiXiaoZhou/p/7429481.html)

