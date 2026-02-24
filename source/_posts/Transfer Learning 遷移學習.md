---
title: "Transfer Learning 遷移學習"
date: 2019-10-11 10:39:20
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning
image: https://i.imgur.com/i5jt7QR.png
mathjax: true
---


當我們要進行某一個任務時，或許可能會遇到一個問題 : 
「與這個任務直接相關的資料並不多，但與任務不直接相關的資料卻不少」

( 當然，這是一個比較冠冕堂皇的理由，大多時候我們想問的是，我們能不能從別人已經完成的任務中，借鑒對方的經驗來讓我們的任務能夠更有效率地完成 )

<!-- more -->

<img width=500 src="https://i.imgur.com/HMkuJoZ.png" >


我們能不能從 Youtube 的中英文語音資料來進行台語的語音辨識 ? 或者從動物的影像辨識模型來進行醫療影響的判讀 ? ....等等。

以下我們以李宏毅老師所整理的架構來一一針對不同的狀況所使用的遷移學習方法做介紹

![](https://i.imgur.com/0o1uDDK.png)

這裡有幾個名詞解釋與前提必須要先提醒 : 

* Source Data : 與我們要進行的任務不直接相關的資料
* Target Data : 與我們要進行的任務直接相關的資料
* 這邊我們會有一個前提是 Target Data 數量遠小於 Source Data

## Labelled Sourse Data , Labelled Target Data

### Fine-Tuning 

一個很直覺的想法是，既然我們有比較多的 Source Data ，那我們就直接利用它先 Train 一個 Source Model 出來，利用 Source Model 作為初始值，拿 Target Data 針對我們的任務 Train 一個 Target Model 出來。

![](https://i.imgur.com/rXEBZAM.png)

但因為 Target Data 數量過少，很容易造成 Target Model 會有 Overfitting 的狀況，因此，我們可以做一些 Constraint。( 讓 Target Model 的輸出必須要與 Source Model 的輸出接近，或是要求兩個 Model 的參數要相近.... )

還有另外一種方式，稱為 " Layer Transfer " 。

保留 Source Model 裡面的其中幾層固定放到 Target Model 中，剩下的層在 Target Model 中進行訓練調整。

![](https://i.imgur.com/tbylCrx.png)

因為 Target Data 數量並不多，這樣的方法減少許多訓練參數，可以有效避免 Overfitting。( 當然，如果我們手邊有夠多的 Target Data，仍然可以 Fine-Tune 全部的 Layers )

怎麼選擇 Source Model 要放到 Target Model 的 Layer ?


<img width=500 src="https://i.imgur.com/xmsHK1E.png" >


從實務上來看，在語音的任務中，固定後面幾層的效果會比較好，原因是因為每一個人說話的語調或許不同，但最後組合出來的語音特徵則是不變的。相同的道理，在圖像任務中，固定前面幾層的效果則會比較好，因為初期的特抽取較為低階，可通用於不同的 Object 之上，但後期組合出來的高階特徵就不見得適用於所有的 Object 上了。

這樣的狀況在 Bengio 2014 年的論文 " *How transferable are features in deep neural networks?* " 中有清楚的實測結果。

![](https://i.imgur.com/VjrrnfS.png)

在這篇論文中，將 ImageNet 共1000個類別的圖片隨機分成兩組 A 與 B ， 各包含了 500個類別與大約一半數量 ( 60萬張 ) 圖片。然後各自訓練一個 8 層的 CNN Model --- Base A 與 Base B。

* BnB : Base B 前 n 層 Layer 固定，重新於 Dataset B 訓練後面的 Layer
* AnB : Base A 前 n 層 Layer 固定，重新於 Dataset B 訓練後面的 Layer

1. **白色圓點** : 代表 Base B 的表現，Accuracy 大約 0.625 。
2. **深藍色圓點** : BnB 的表現，B1B 或 B2B 其實對於整個 Model 並沒有太多的影響，但有趣的是 B3B -- B6B 卻有著明顯的性能下降狀況。作者稱之為 " **fragile co-adapted features** "，此狀況顯示了 3--6 層之間有強烈的依賴性，層與層之間在特徵萃取上面是會互相影響的，所以當我們將這一整組相依的 Layers 固定一部分再訓練其他部分，便會破壞它們的相依關係，訓練出來的模型便會整個壞掉。而 B7B 後模型性能又再度恢復，因為參數調整的部分已經剩一兩層而已，因此與 Base B 的差異便不會太多。
3. **淺藍色圓點** : BnB 中將固定的 Layers 參數進行微調，可以有效的改善 BnB 的狀況。
4. **紅色方點** : 可以看見前面 1-3 層，對於整個模型來說並沒有太大差別，再次應證了圖像辨識的過程中，初期特徵萃取的通用性。但畢竟是以 Base A 對於 Dataset B 做遷移學習，因此越後面模型崩壞的狀況會越明顯，這是可以預見的。
5. **粉紅色方點** : 這是一個真正的遷移學習，利用 Base A 進行 fine tune 後的性能可以大大增加。 

上面的 Dataset A 與 Dataset B 是隨機進行分組，或許可能因為兩個 dataset 間有相似類別導致上面的實驗結果可以這麼好 ? 因此 Bengio 也做了另外一組實驗，將 Dataset A (非自然類別) 與 Dataset B (自然類別) 做有差異的分組。

![](https://i.imgur.com/FqG2tRE.png)

我們可以看見，不管怎麼樣子的分組，前面幾層的遷移是有絕對的效果的 ( 紅、橘點 )。[^註1]


### Multitask Learning

除了上述的方法外，我們也可以從另外一個角度來看。


<img width=500 src="https://i.imgur.com/WCDIbC2.jpg" >


將 Source Data 與 Target Data 合起來，作為一個大的 Task，模型可以同時對 Source Data 與 Target Data 做不同的輸出。

在語音任務上，這樣的方式的確可以獲得模型性能上的增強。


<img width=500 src="https://i.imgur.com/QldNwsO.png" >


### Progressive Neural Network

當然也有人提出很多變體，像是 Progressive Neural Network，但這些變體並不見得很好用，或是很有幫助，但可以提供另外一種思維。


<img width=500 src="https://i.imgur.com/tYZgw0l.png" >


Progressive Neural Network 主要的概念是這樣，當我們有 許多不同的 Tasks，我們可以將這些 Tasks Model 的各層 Layers 接在我們最終想要處理的 Target Model Layer 之後。

這樣的好處是，這樣的模型性能最差就是單純 Train Target Model 的結果 ( 可以把其他 Tasks 的 Layers 參數調成 0 就會變成 Target Model )，但卻可能可以保留住其他 Task Model 的 information。

## Labelled Sourse Data , Unlabelled Target Data

### Domain Adversarial Training


<img width=500 src="https://i.imgur.com/okylctH.png" >




Labelled Sourse Data , Unabelled Target Data 似乎很像我們的 Training Data 與 Testing Data 的關係，但這邊其實會有一個前提是，Target Data 與 Sourse Data 其實還是有某些機器不見得可以判別的差異。

譬如 : MNIST 與 MNIST-M 的差異

![](https://i.imgur.com/urhkDjk.png)

首先，我們先將 Source Data 與 Target Data 理解為不同的空間 ( Domain ) 中的資料點。由於 Source Data 已經有標籤完成，也就是說在 Source Domain 上已經有一個分類器可以將其分類。但在 Target Domain 上我們無法找出這樣的分類器。

而我們希望做的事情就是將兩個不同 Domain 的資料點 map 到另外一個空間，在這個新的空間中，讓 Source Data 與 Target Data 的分布盡量接近，而且Source Data 在新空間中仍然可以被分類。這樣我們就可以利用 Source Data 協助 Target Data 進行分類。


<img width=500 src="https://i.imgur.com/MDvRQsc.png" >
(圖片 (1) 來源 : [[ EYD与机器学习 ] 迁移学习：DANN域对抗迁移网络](https://zhuanlan.zhihu.com/p/51499968) )


在這樣的概念下就衍生出了 Domain Adversarial Training of Neural Network ( GANN )。

![](https://i.imgur.com/02dhSS4.png)

DANN 主要分成三個部分 : 

1. **Feature Extractor** :
前面幾層的特徵萃取我們可以視為圖(1) 的 $\phi$ ，作為一個空間轉換，將 Source Data 與 Target Data map 到一個新的特徵空間中。
2. **Domain Classifier** : 
Domain Classifier 用來確認特徵空間中的點是來自 Source Domain 還是 Target Domain。
3. **Label Predictor** : 
Label Predictor 則是對特徵空間中的點進行分類。(也就是根據特徵萃取後的特徵來決定要怎麼分類資料)

所以整個 DANN 在做的事情就是，資料進入 Feature Extractor 後，先要將來自 Source Domain 及 Target Domain 的資料分布讓其盡量相似。但是這個 $\phi$ 實際上是什麼我們並不知道，因此要利用 Domain Classifier 做 GD 來訓練出 $\phi$。

但 Domain Classifier 本來是希望能將 Domain 分得越開越好，因此必須在 Domain Classifier 與 Feature Extractor 之間加一個梯度反轉層 ( Gradient reversal layer ) 便可以達到相反的效果。

那麼兩者進到特徵空間後呈現相同的分布之後，我們必須還要確定 Source Data 經過特徵萃取後的轉換還能夠被進行分類，此時便要用到 Label Predictor 。

綜觀上面三個部分，我們就可以訓練出一套模型，既可以將 Source Data 準確的分類，又可以藉由這樣的分類器來對 Target Data 進行分類。

### Zero-Shot Learning

假如 Source 與 Target 的任務是不同的情況呢 ?



![](https://i.imgur.com/yPLgZYw.png =450x)



![](https://i.imgur.com/XITVQ9J.png)

* ### Attribute Embedding

一種方式是，我們可以針對每一個分類，製作一個 Attribute Table，然後再將資料與 Attribute 通通 map 到 Attribute Space 中。這樣我們可以針對沒見過的資料進行 Attribute 的相似性比對來進行分類。

![](https://i.imgur.com/pmEQmpT.png =400x)
![](https://i.imgur.com/ayI27Qu.png)



* ### Attribute Embedding + Word Embedding

但現實狀況是，我們根本不太可能會有 Attribute Tabel 這樣的 Database。此時我們可以運用文字上可能會帶有的意義來進行 Word Embedding。

![](https://i.imgur.com/W4gtVbK.png)

那問題來了 ，我們要怎麼在 Attribute Space 中進行相似性比對 ? 什麼是 「相似性」?

如果我們將簡單其定義為 

$$
f^*,g^*=arg\min_{f,g}\sum\limits_{n}\|f(x^n)-g(y^n)\|^2
$$

這樣在訓練過程中，整個 NN 會直接把所有點 map 到同一點，這樣通通都很接近，但這並不是我們要的結果，必須對這式子加入一些 Constraint。

$$
f^*,g^*=arg\min_{f,g}\sum\limits_{n}\max(0,k-f(x^n)\cdot g(y^n)+\max_{m\neq n}f(x^n)\cdot g(y^m))
$$

$k$ 我們可以視為一個門檻值，從上面的式子來看，Loss=0 的狀況會是

$$
k-f(x^n)\cdot g(y^n)+\max_{m\neq n}f(x^n)\cdot g(y^m)< 0\\
f(x^n)\cdot g(y^n)-\max_{m\neq n}f(x^n)\cdot g(y^m) > k
$$

簡單解釋就是我們希望這個 model 可以呈現的狀況是將 $x^n$ 與 $y^n$ map到 Attribute Space 中，而且 $x^n$ 與 $y^n$(正確類別) 的相似度必須大過 $y^m$(錯誤類別)， 相似度差異至少 $k$。

這樣的條件便可以使模型不會將所有資料都 map 到 Attribute Space 的同一點，使的模型有更好的鑑別度。

* ### Convex Combination of Semantic Embedding ( ConSE )

這也是另外一個 Zero-Shot Learning 的方法，但它比較簡單。

這個方法出現在 2013年 的論文 " *Zero-Shot Learning by Convex Combination of Semantic Embeddings* " 中，概念其實就是 " $CNN+Word2Vec$ "。

如果我們有一個現成的 ( off-the-shelf ) CNN 以及一組現成的 Word Vector，那我們就可以做出一個完全不用訓練的 Zero-Shot Learning : 


<img width=500 src="https://i.imgur.com/T6xbz1s.png" >


我們可以將一個未知的 object 輸入我們手上有的 CNN Model，那麼輸出就會給出各分類的機率值。

將這些機率作為各個分類的向量進行線性組合的係數，我們便可以找到這個待測 object 的 Word Vector。


<img width=500 src="https://i.imgur.com/O41Uklr.png" >


隨後在從我們手邊有的 Word Vector Set 中找出一個與此待測物向量最接近的 Word Vector 即可給出這個待測物的名稱。


![](https://i.imgur.com/1bf71uY.jpg)

這樣的結果其實表現得還不算太差。加上不用訓練可以省下許多的訓練成本。



### Zero-Shot Translation

這是李宏毅老師在這部分最後提出來的 Zero-Shot 的例子，這是一個蠻新的論文，出自 2016年 " *Google’s Multilingual Neural Machine Translation System: Enabling Zero Shot Translation* "的論文中。

![](https://i.imgur.com/VPuNO7l.gif)

作者們訓練一個 GNMT 模型可以進行 日英、韓英 的互譯 ( 藍色實線 )，接著他們想問的是 :「這樣的模型能不能針對沒有見過的語言來做翻譯 ?」

驚人的結果是，這答案是肯定的。

這個模型可以直接產生合理的日韓翻譯，即使這個模型從來沒有處理過這樣的問題。這樣的過程， Google 稱之為 " Zero-Shot Translation "，而這也是第一個將遷移學習應用在機器翻譯 (Machine Translation) 的例子。

![](https://i.imgur.com/D4DneTX.jpg)

這樣的結果，證實了機器學習到的是一種語言的抽象形式，使相同語意的語句能以不同語言但相似的方式來呈現 --- 稱之為 interlingua 。

上圖是針對日英、韓英互譯使用 t-SNE 進行 Semantic Embedding 的結果 : 
* 圖 ( a ) 顯示整個 Semantic Space 的鳥瞰圖，其中同樣語意的會以相同顏色來表示。我們可以發現同樣語意的幾乎都會呈現 Cluster 的狀況。
* 圖 ( b ) 則是抽出其中一個語意的 Cluster 出來檢視。
* 圖 ( c ) 中，我們將這裡面不同語言的部分也以不同顏色來表示。我們也可以發現不同語言在這個大的 Cluster 中也會呈現不同的小 Cluster。

## Unabelled Sourse Data , (Un)Labelled Target Data

這裡的狀況跟 Semi-Superviced Learning 很像，差別在於，Semi-Superviced Learning 的假設會是在 Label Data 與 Unlabel Data 之間是相關的，然而在這邊我們討論的前提通常是 Source Data 與 Target Data 之間的相關性不高。

### Self-Taught Learning

Self-Taught Learning 的概念是，我可以先用 Source Data 先訓練出一套 Feature Extractor，在利用這個 Feature Extractor 對 Target Data 來抽特徵。

而論文內做了很多實測發現這樣的方法確實也可以得到不錯的結果。

註釋
---

[^註1]:
綠色部分是參考 Jarrett 於 2009年提出的論文 " *What is the Best Multi-Stage Architecture for Object Recognition?* " 提出來的論點，如果隨機決定 CNN Kernel 的個數是否能增強模型性能 ? 此論文中於 Caltech-101 上確實有性能上的增加，但在 Bengio 的論文中卻沒有辦法有相同的性能增加，猜測是與 dataset 的大小有關。