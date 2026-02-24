---
title: 中文自然語言處理 (NLP) 的進展與挑戰
date: 2019-11-01 11:53:09
categories:
- 活動紀錄 Record of Multi-Learning
image: https://i.imgur.com/efhGyok.jpg
mathjax: true
---

時間 : 2019.10.31
地點 : 台灣大學德田館 103 教室
主講者 : 馬偉雲博士

<!-- more -->

筆者對 Natural Language Processing (NLP, 自然語言處理)一直蠻有興趣，因此一個月前看到這個活動資訊便立刻決定參加，上周中研院開放參觀，筆者也有幸跟馬教授交談過，針對自己在進行 LineBot 的一些疑問請益，便更期待這一天的講座，以下是在講座過程中筆者的隨手筆記，或許沒有非常鉅細靡遺，但盡可能的記下重點。

中研院詞庫小組(CKIP)
---
是中研院跨所的一個中文計算語言研究小組，五個主要研究方向：深度學習、知識表達、自然語言理解、知識擷取、聊天機器人。

NLP簡介
---

語言是思惟的外化，代表著人類的智慧，目前比較成功的任務 : 機器翻譯、Chatbot、網頁查詢、詐騙偵測、輿情分析....

### NLU (Natural Language understanding) & NLG (Natural Language Genration)

NLU ( Lang to Structure data ) : 查詢、詐騙偵測、情緒分析..
NLG ( Structure data to Lang ) : 新聞寫作
NLU + NLG : 機器翻譯、聊天機器人

### Difficulty

1. 語言具有歧異性

    * 我很「難過」 VS 這條河很「難過」
    * 這是我們的「學習時間」 VS 我們一起來「學習時間」
    * 他騎「機車」 VS 他很「機車」
    * 小名和小美結婚 VS 小明和小美都結婚

2. Domain knowlege

    * 勇士對尼克，勇士贏了 VS 勇士對巨人，勇士贏了

NLP 進展
---

### 深度學習的引入

* 自動學習特徵 (domain knowledge)
在傳統 Machine Learning 上我們必須要給予一些 feature 來輸入 model 進行任務的分析，但在 Deep Learning 上我們省略了 Feature Extraction 的步驟，機器可以自行學習出 features。在論文 "[*Convolutional Neural Network for sentence classification*](https://www.aclweb.org/anthology/D14-1181.pdf)" 利用 CNN model 來進行 end2end 特徵的萃取及分類。

![](https://i.imgur.com/P70R3I8.png)


* Pre-training + training ( unsupervised + supervised )
在 NLP 中，我們希望要先對 text 進行 embedding (pre-training) ，給予一個比較好的初始值，再丟進  model 進行訓練 ( training )，可以節省計算成本。
BERT : pre-training (embedding/model) + training (finetune)

![](https://i.imgur.com/49XotY6.png)

單純的 Pre-training + training 可能會因為語意的歧異性產生問題，但這樣的問題在 BERT 上並不復見，因 BERT 中會做到語意解歧的作用。 

### 知識圖譜的引入

* Crowd soursing 使知識圖譜建立難度降低
例如 wiki、web comment 的產生。
Zhou 等人在 IJCAI,2018年論文 "*[Commonsense Knowledge Aware Conversation Generation with Graph Attention](https://www.ijcai.org/proceedings/2018/0643.pdf)*" 利用知識圖譜來建立聊天機器人。Ghazvininejad 等人於2018年發表的論文 " [*A Knowledge-Grounded Neural Conversation Model*](https://arxiv.org/pdf/1702.01932.pdf) "中利用美食評論來訓練 Model


NLP 挑戰
---

* 對 DL 的 NLG 來說，生成的文字難以控制語意
* 知識與常識的擷取、整合、應用
* Unsupervised Learning、Semi-Supervised、GAN、Distant Supervision (only for NLP)

<img width=500 src="https://i.imgur.com/6N5x3Sq.jpg" >

* Explainable，這是所有深度學習結構的共同問題。

NLP 解決之道
---

* 避免說錯、該說的沒說、嘮叨
"Sementically conditionaled LSTM-based Neural Language Generation for Spoken Dialogue Systems" 在 forget gate 上進行調整


<img width=500 src="https://i.imgur.com/jo8HQuw.png" >


* 知識及常識的整合
打造"[大廣義知網](http://ehownet.iis.sinica.edu.tw/index.php)"來整合知識與常識。

<img width=500 src="https://i.imgur.com/HNX8Lts.jpg" >



QA
---
### Q : 在一些 NLP 應用上常會針對「意圖」進行分析處理，馬教授是否有什麼想法 ?

A : 在任務型的機器人中，我們比然要針對意圖進行分析，確定意圖後再進行回復，但純聊天機器人中，意圖分析或許是比較偷吃步的方式，但是還是會希望整個純聊天的 model 可以把意圖分析融合進 model 中。

### Q : 中研院是否有針對新詞、錯字偵測進行什麼樣的研究 ? 

A : 新詞是 NLP 長年的問題，目前中研院的研究是將新詞與斷詞可以一起處理，錯字偵測的做法大概也是在 NN 上面 training 來做，關鍵還是在資料量不足。或許我們可以從中文教學中取得錯字偵測的資料或是從 wiki 上取得編輯紀錄來收集。

### Q : 中文的 NLP 與英文在處理上有什麼樣子的差異 ?

A : 中英文的差異除斷詞問題、中文的語法結構較英文鬆散靈活、中文上主詞會很常省略。因此，在中文長篇上會較英文難上許多。此外，英文的量詞發達、中文太常使用倒裝句，這些都會造就 NLP 上的處理難度。


CkipTagger
---

這個部分，大多結合實際操作進行講解，因此筆記部分會稍微凌亂。

<img width=500 src="https://i.imgur.com/dov1e1i.jpg" >


[CkipTagger](https://ckip.iis.sinica.edu.tw/service/corenlp/) 是一個結合斷詞、詞性標記、實體辨識的一個國產開源套件。

### 斷詞技巧

* Word-level approach
    1. Maximum length (長詞優先)
    2. 動態規劃查找最大概率路徑
* Character-level approach
    1. character Sequence Labeling
* CkipTagger 則是綜合上面兩種方法，針對 word 及 character 同時進行分析。

### 綜合介紹

* CkipTagger 利用 中央社、wiki (用舊版套件先進行斷詞) 及 ASBC (為期近10年人工標記) 為文本進行 word embedding

<img width=500 src="https://i.imgur.com/aC2b0sV.jpg" >

* 同時要參考 word embedding 跟 character embedding，主要因為在 character 層次中可能會凸顯一些 word 的重要性。舉例來說，如果只單純考量 word embedding，「台北市」這樣的字詞就是一個 embedding，但若同時考量到 character embedding，則可以抓出在上下文中可能會更強調台北市這三個字中「市」這樣的字。

<img width=500 src="https://i.imgur.com/gLorwH2.jpg" >


* 傳統 BiLSTM 無法解決 XOR 問題，因此衍生 Cross-BiLSTM

<img width=500 src="https://i.imgur.com/Zs07aSX.jpg" >

<img width=500 src="https://i.imgur.com/pyrJPtl.jpg" >

* 從 ASBC Corpus 上實驗，如果我們定義大量詞彙，其實對結果影響不大，這表示整個 model 的自動化程度已經做得很好了。


### QA

* 目前 Ckip 無法處理 overlap 的切詞 (眼鏡框切成眼鏡/鏡框/眼鏡框)

* 在製作新詞上，結巴可以給予詞頻，但 Ckip 是允許給定權重，這樣的權重跟詞頻在數學上的概念稍有不同

* 詞類有必要這麼多種嗎？事實上是需要的，這麼細緻的分類對NER的處理上面是有功效的，而且在實際應用上，例如聊天機器人，可以針對細緻的分類進行比較保障的互動。

* 由於 Ckip 並非長詞優先系統，因此在處理「韓國瑜」「韓國瑜珈老師」的切詞就不見得可以切得很好。

* Ckip 考慮的是 sentence 的 Context，因為預設是全文的斷詞不太會有不同的狀況出現。

* 針對未來 Ckip 的期待，主要會集中在 NER 研究上，因為 NER 目前 performance略差，但斷詞已經可以達到 97%。

* 針對本屆科技大擂台，建議是先利用手邊能有的 DL 方式 (例如:BERT)建立初一個 baseline，再試圖加入 Knowledge Graph 進來增強 performance。