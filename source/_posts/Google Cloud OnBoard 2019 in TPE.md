---
title: Google Cloud OnBoard 2019 in TPE
date: 2019-05-14 19:18:07
categories:
- 活動紀錄 Record of Multi-Learning
image: https://i.imgur.com/GzXaj0u.png
mathjax: true
---


其實我根本忘了我在哪邊報名的，上週 google 打電話來跟我確認時間我還問他到底內容是什麼，再回去找信箱才發現我竟然真的有報名。

有人前幾年參加過，戲稱這是 Google Cloud 的佈道大會，其實免費的場次，美其名都是研討會、課程，但終究會回到商業目的，可能是產品銷售、也可能是徵才，不過今天聽下來還是有一些不錯的心得跟感想。

<!-- more -->

## Google Cloud Platform

![](https://i.imgur.com/J9rXsgN.png)

今天大概 2/3 的時間都是圍繞在「如何在 Google Cloud platform 中的這些產品中處理大數據」，上述的產品我們可以依照功能性來大致做以下的分類 : 

![](https://i.imgur.com/FVn270C.png)

其實從這裡可以看出來， google 想做的就是，將所有資料儲存、分析、運算、檢索全部在同一個系統中實現，而且這個系統必須

1. 可以處理非常大量的資料
2. 系統擴展性極佳 (不管資料型態是 batch 或 stream 均適用)
3. 運算能力超好，對客戶的計費上面可有彈性空間 ( 使用才付費 )
4. 所有的設備維護均由 google 進行

看到這裡人家以前常常說，AI開始盛行後人們會逐漸失業，我倒覺得 google 這整套系統真的搞起來才是讓人失業的主因 (誤) .....

以下我大概簡單介紹一下幾個還不錯的東西 : 

### Google Cloud Storage

顧名思義就是雲端儲存，不過計費方式可以依照使用頻率而做調整，甚至用戶可以針對檔案的可能會有的閒置時間做策略性調整，用來減輕費用。

EX: 我們可以依照需求對檔案做設定，多少時間後，檔案存取改為一年幾次，在久一點，頻率再降低，最後幾年後設定刪除，系統便會依照這樣的設定分段收費，刪除後就不繼續收費這樣。

但是 google 並非只有 Google Cloud Storage 可以做儲存的動作，我們可以依照檔案、用戶需求將檔案、資料儲存在不同的地方

![](https://i.imgur.com/ta0vSmb.png)

### Google Dataproc 

Dataprocess 的縮寫，整合了許多知名的大數據框架，如 hadoop , Spark ...等，用戶可直接在系統內使用自己熟悉的或是過往資料所使用的框架，不需因為轉移資料而改變語言。
建立 cluster 後也可以快速擴張，並能依照自己需求對節點數、虛擬器類型、磁碟機大小...做調整。

### Google Cloud Spanner

Cloud Spanner 是一個可水平擴展且具有強一致性的關聯資料庫，與一般傳統資料庫不同之處在於 : 

![](https://i.imgur.com/O5LdFbg.png)

過去我們曾學過，在計算機理論中對任何的分散式計算系統來說，必然會符合 CAP theory[^註1] ，但 Cloud Spanner 是唯一能夠打破 CAP theory 的系統。


     
![](https://i.imgur.com/sfvJgXE.png)

一般關聯性資料庫 (如 MySQL) 在資料大於某種程度時會產生嚴重的 Latency ，但在 Cloud Spanner 中我們可以利用增加 nodes 來減緩這樣的 Latency。

### Google Bigtable

Bigtable 利用獨特的 Row key 讓資料可以快速地被檢索，這樣的 solution 適合用於快速且大量生成的資料，譬如股市資料。

![](https://i.imgur.com/mhGvx1l.png)

### Google BigQuery

BigQuery 是一個全託管的資料倉儲 (data warehouse) 服務，讓用戶可以在級大量欄位資料中進行特定的檢索。

特別的是，我們可以利用各種方式 load data ，使得用戶可以利用 BigQuery 進行快速同步的 SQL 查詢。

![](https://i.imgur.com/qHE2xs2.png)



---

除了上述幾個蠻重要的服務外，Google 也有一個 Datalab ，可以讓用戶在一個基於 jupyter notebook 上的環境進行各種 coding work : 

![](https://i.imgur.com/NbFQL1E.png)

介面跟我們熟悉的 jupyter notebook 幾乎一樣

![](https://i.imgur.com/mKQZSuv.png)

也支援 BigQuery 

![](https://i.imgur.com/hJrfotR.png)


---

說了這麼多，其實不難理解 Google 其實就是試圖在營造一個能夠實現任何大數據操作的環境，當然包括 Machine Learning / Deep Learning

我們可以利用剛剛說到的所有工具，在 Datalab 中利用 TensorFlow 來進行 Deep Learning ，如果你以為這樣已經整合到一個極致，那你就錯了。

### Google CloudML and Google AutoML

Google 幫你準備好各式各樣的 Machine Learning APIs，讓用戶可以針對自己的需求利用已經 pre-training 的 model 直接進行各式各樣 Maching Learning 的工作

![](https://i.imgur.com/Aye9EAu.png)

大家可以搜尋上圖右方的各種 APIs，都可以進行試用，在試用的時候，會需要一點時間，因為每一個 API 都有許多 pre-train models 在同步進行。

其中我想特別介紹的是 Video Intelligence API ，以往我們再對於 Youtube 的推薦系統都停留在基於用戶的點擊、評分上進行推薦，但新型態的分析已經可以利用用戶在觀賞影片時所進行的快轉、回放以及在特定畫面的停留、觀看等資訊進行分析並推薦。

![](https://i.imgur.com/o8vS3LJ.png)

更進一步的， Google 也提供了 AutoML 服務，只要用戶將資料放進去，後續的工作幾乎都由 AutoML 來完成，完完全全的 ML 客製化服務。

![](https://i.imgur.com/5iRLh7x.png)


### Google Cloud Pub/Sub & Google Dataflow

在 Big Data 的處理上，常會面臨大量串流以及批次的資料，我們必須在限定時間內不僅面對批次資料，甚至也要加入持續產生的大量串流資料一起做資料分析，而最後介紹的這兩個服務便能讓用戶可以輕鬆處理串流及批次資料， Google Dataflow 讓用戶可以使用 Apache Beam SDK 設定各種時間區間以及多資料來源的接收等獨特的方式在串流及批次資料同時發生的情況下也可以快速地進行開發。

![](https://i.imgur.com/KTueXMi.png)



---
## 實務分享 --- 愛料理

![](https://i.imgur.com/RlyGi8q.png)

以下是愛料理基於 GCP 之上的各式應用

### Recipe Q&A Interactive Chatbot --- 聊天機器人
![](https://i.imgur.com/4eodqna.png =200x)
● Dialogflow
● Cloud Firestore
● Cloud Functions

### Actions on Google --- Google語音機器人
![](https://i.imgur.com/F4hPJKm.png =200x)
● Dialogflow
● Cloud Functions
### VIP feature - Popularity sorting --- VIP獨特的食譜排序
![](https://i.imgur.com/nraEUuS.png =400x)

● BigQuery
● Cloud SQL
● Composer
● AutoML Natural Languages
● AutoML Vision (Testing)
### Social listening on Instagram --- 社交媒體上的留言判別並回覆
![](https://i.imgur.com/r7JUwlJ.png =400x)

● BigQuery
● Cloud Functions
● AutoML Natural Languages
### Product recommendation on LINE --- Line 的推薦系統
![](https://i.imgur.com/dNPWx9l.png =200x)

● BigQuery
● Dialogflow
● Dataflow
● Cloud Pub/Sub
● Cloud Functions

## 實務分享 --- 17直播

![](https://i.imgur.com/0hKSJDv.png)

### 即時監控串流情況
![](https://i.imgur.com/EhIh9Hf.png =200x)![](https://i.imgur.com/ROft4yi.png =400x)
很多人認為監控就是一直刷每一個直播主的直播，但事實上比較多的是是監控每一個直播主的流量、CPU運作、溫度...等硬體資訊，來確認直播的絣直是否受影響

![](https://i.imgur.com/cNBJexA.png)

![](https://i.imgur.com/vgHjqUU.png)

整場聽下來，其實心裡面有很大的感觸，在學習AI 的過程中，我們是不是也會成為被機器學習的對象 ? 在 Google 這波自動化過程中，我們應該如何自處 ? 我們又應該思考，如若這樣的風潮持續，我們應該怎麼看待我們自身的存在 ?


所有的危機都會是轉機，或許我們都該想想在這樣的情況下，我們是不是真正有能夠不被取代的價值。

雖感覺有點悲觀，但是我還是從這場會議中看到很多以前沒想過而且可以應用的機器學習問題 ，而這些想法或許可以作為未來的專題的發想 : 

* 在圖片辨識的學習過程中，我們好像都還在討論「如何辨識」的階段，但是在商務、實務應用上，這樣的階段其實只是一個開始，重點還是放在「我們該利用這樣的辨識來對用戶做什麼回饋 ?」給予推薦 ? 提出警示 ? 這或許是未來我自身在專題製作上最重要的目的。

* 原本我對於NLP (  Natural Language Processing  ) 並沒有太大的興趣，但今天的實務分享讓我覺得 NLP  其實很有趣，聊天機器人如何能夠更精準、更人性化的回饋，並且給予最符合用戶需求的推薦，這問題實在有趣。

---

## 參考資料

整個會議的主要內容大部份都講完了，很有價值的還有整個過程中所有的 Resourse，這些都會是未來學習的材料 : 

### About Google Cloud Platform
* Google Cloud Platform (https://cloud.google.com/)
* Data Centers (https://www.google.com/about/datacenters/)
* Google IT security (https://cloud.google.com/security/overview/whitepaper)
* Why Google Cloud Platform?
(https://cloud.google.com/why-google-cloud/)
* Pricing Philosophy (https://cloud.google.com/pricing/principles)
* Google Cloud Platform https://cloud.google.com/compute/)
* Datacenters (https://cloud.google.com/storage/)
* Pricing (https://cloud.google.com/pricing/)
* Cloud Launcher (https://cloud.google.com/launcher/)
* Pricing Philosophy (https://cloud.google.com/pricing/principles)

### Google Cloud Platform services
* Cloud SQL (https://cloud.google.com/sql/)
* Cloud Dataproc (https://cloud.google.com/dataproc/)
* Cloud Solutions (https://cloud.google.com/solutions/)
* Cloud Spanner (https://cloud.google.com/spanner/)
* Cloud Bigtable (https://cloud.google.com/bigtable/)
* Google BigQuery (https://cloud.google.com/bigquery/)
* Cloud Datalab (https://cloud.google.com/datalab/)
* TensorFlow (https://www.tensorflow.org/)
* Cloud Machine Learning (https://cloud.google.com/ml/)
* Cloud Pub/Sub (https://cloud.google.com/pubsub/)
* Cloud Dataflow (https://cloud.google.com/dataflow/)
* Processing media using Cloud Pub/Sub and Compute Engine
(https://cloud.google.com/solutions/media-processing-pub-sub-compute-engine)
* Reverse Geocoding of Geolocation Telemetry in the Cloud Using the Maps API
https://cloud.google.com/solutions/reverse-geocoding-geolocation-telemetry-cloud-maps-ap)
* Using Cloud Pub/Sub for Long-running Tasks
https://cloud.google.com/solutions/using-cloud-pub-sub-long-running-tasks)


### Google CloudML APIs
* Vision API https://cloud.google.com/vision/
* Translation API https://cloud.google.com/translate/
* Speech API https://cloud.google.com/speech/
* Video Intelligence API
https://cloud.google.com/video-intelligence

### Google Publication database
https://ai.google/research/pubs/

### Another
* Big data and machine learning blog (https://cloud.google.com/blog/big-data/)
* Google Cloud Platform blog (https://cloudplatform.googleblog.com/)
* Google Cloud Platform curated articles (https://medium.com/google-cloud)

註釋
---

[^註1]: 
在理論計算機科學中，CAP定理（CAP theorem），又被稱作布魯爾定理（Brewer's theorem），它指出對於一個分布式計算系統來說，不可能同時滿足以下三點：
1. 一致性（Consistency） （等同於所有節點訪問同一份最新的數據副本）
2. 可用性（Availability）（每次請求都能獲取到非錯的響應——但是不保證獲取的數據為最新數據）
3. 分區容錯性（Partition tolerance）（以實際效果而言，分區相當於對通信的時限要求。系統如果不能在時限內達成數據一致性，就意味著發生了分區的情況，必須就當前操作在C和A之間做出選擇。）