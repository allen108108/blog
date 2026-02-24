---
title: "Generative Adversarial Network (10) --- Evaluation & Concluding Remarks" 
date: 2019-11-23 02:46:54
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning and having it Deep and Structured
image: https://i.imgur.com/RrdITkw.png
mathjax: true
---

GAN 講了這麼多，重點還是要得到一個夠好的 Generator 可以生成我們想要的東西。可是，什麼樣的 Generator 是夠好的 ? 我們怎麼評估訓練完成的 Generator ?

<!-- more -->

Likelihood
---

傳統的評估方式就是計算資料的 Likelihood。給定一些真實的圖像 ( 不再訓練資料中的 )，則我們稱 Generator 生成出這些圖像的機率為 Likelyhood。

如果 Likelyhood 夠大，則我們可以稱這樣的 Generator 夠好。在計算上我們會針對每一個資料點計算 Likelyhood 後取 $\log$ 再平均。

$$
\log\ Likelyhood=\frac{1}{N}\sum\limits_{i=1}^{N}P_G(x^i)
$$

但在這裡，這樣的計算很困難，因為我們無法找出 $P_G(x^i)$ 是多少，所以也無法計算 Likelyhood。

在文獻上，有一個方式來解決 $P_G(x^i)$ 的狀況，稱為 Kernel Density Estimation。其概念大概是這樣，我們沒辦法對每一個 $x^i$ 計算其 Likelyhood，但我們可以先用 Generator 生成許多資料，將每一筆資料視為是一個 Gaussain Distribution 的平均值 (mean)，然後每一筆資料/均值配合一個 variance 我們可以得到每一個 Gaussain Distribution，再將這些分佈疊成一個 Gaussian Mixture Model，那我們就可以利用這個 Gaussian Mixture Model 來計算生成 $x^i$ 的機率了。

![](https://i.imgur.com/pBHtWof.png)


Likelyhood 來作為評估的作法很直覺，但其實還是會有幾個問題產生 : 

1. 我們不知道要生成幾筆資料做出幾個 Gaussain Distribution 來疊成一個 Gaussian Mixture Model 比較適合。因此許多文獻上用這種方式做出來的結果都不見得會很好。

2. Likelyhood 與 Generator 並不一定相關。低 Likelyhood 頂多表示 Generator 生成出來的跟我們拿出來的真實資料 $x^i$ 「不夠像」，但 Generator 可能生成出夠清楚的圖片，只是跟這些 $x^i$ 不一樣而已。另一方面，Likelyhood 其實很難反映出 Generator 的能力差異，舉例來說，如果有一個 $G_1$ 可以算出其 Likelyhood 為非常大的值 $L$，另外一個 $G_2$ 有 $1\%$ 的機會表現得跟 $G_1$ 一樣，$99\%$ 都生成 noise，那麼計算過後其 Likelyhood$=L-\log 100\approx L-4.6$ 相差根本不大。

![](https://i.imgur.com/nCGkPZ5.png)


Objective Evaluation
---

現在文獻上比較常使用的方式是利用一個 Pre-train Classifier 來進行評估。

如果將 Generator 生成的圖片丟給一個 Classifier (VGG / Inception / ...) 會輸出一個分佈，會針對所有類別給出機率。我們可以看看這些類別的機率，如果分佈越集中 ( 某一個類別的機率很明顯的特別高 ) 那我們就可以稱這個 Generator 的生成圖像越好。

但這樣其實並不夠，因為 Generator可能「只會」生成同一個很清楚的圖，這種情況也可以讓 Classifier 的分佈很集中，但這樣的結果不是我們要的。

因此我們還需要評估這個 Generator 的輸出圖像的 diverse。計算的概念就是將所有的生成圖像丟進 Classifier，將這些這些分佈計算平均，如果平均後的分佈越不集中，則表示這個 Generator 可以生成多種不同的圖像。

結合上述兩個評估面向 : (1) 分佈夠不夠集中 (2) Diverse 夠不夠分散，便可以定義出一個評估分數 Inception Score ，其計算方式如下

$$
Inception\ Score=\sum\limits_{x}\sum\limits_{y}P(y|x)log P(y|x)-\sum\limits_{y}P(y)\log P(y)
$$

第一項計算生成的圖像丟給分類器是否可以得到夠集中的分佈 (Entropy of $P(y|x)$) ; 第二項則是計算分佈平均夠不夠分佈平均夠不夠分散 (Entropy of $P(y)$)。

訓練 GAN 的注意事項
---

### 生成圖像夠好，並不見得 Generator 的能力就夠好。

如果只單純看生成出來的圖像夠不夠好，其實 Generator 只要複製訓練資料的圖片就好，也就是說，我們根本不用 Generator。

一個能力夠好的 Generator 我們除了要它能生成夠好的圖片外，還會希望它可以生成跟訓練資料不同的圖像。

![](https://i.imgur.com/KgXOjIO.png)

上圖為論文 " *A NOTE ON THE EVALUATION OF GENERATIVE MODELS* " 中的一個實驗結果。單純利用兩個圖像的 $L2\ norm$ 來衡量兩張圖像是否一樣，則會產生一些問題。以上圖左為例，當我們將一張山羊的圖像進行平移，就機器而言可能就會判斷成不同的圖片。

因此，怎麼去自動化判斷生成圖像是否跟訓練資料圖像一樣仍是一個尚待解決的問題。

### Mode Drop

另外一個問題是，我們進行生成的時候，到底整個生成圖像的分佈到底可以多大 ? 白話一點說，就是生成的圖像，到底可以生出多少種截然不同的圖像 ?

在 DCGAN ( Deep Convolutional GAN ) 的實驗中，嘗試著想找出機器到底可以生成多少種不同人臉。實驗後結果發現，可以保證生成 400 張人臉的狀況下，我們有 $50\%$ 的機率可以從中找到兩張同一個人臉 (可能只有某些細節不同，但人類可以認定為同一人)。也就是說，整個 DCGAN 可以生成出大約 0.16M 張不同的人臉圖像。

另一個 ALI model 則可以生成出大約 1M 張不同的人臉圖像。


GAN 大集合
---

最後的部分，整理出課程內有介紹(或是簡單講過) 的所有 GAN 種類。

PS : 在筆者的筆記中不見得全部都有介紹到，有些真的只有提過或只是為了湊梗，就沒有多花太多時間筆記。

![](https://i.imgur.com/YSgdswY.png)
