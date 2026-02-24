---
title: AI Cup Camp on ChatBot
date: 2019-09-01 23:18:07
categories:
- 活動紀錄 Record of Multi-Learning
image: https://i.imgur.com/1Xfylm4.jpg
mathjax: true
---

這是一場以聊天機器人 ( ChatBot ) 為主的座談，地點在台灣大學博理館演講廳，主要與會學者為 : 李宏毅 (台大電機系教授)、蔡宗翰 (智慧型資訊服務研發實驗室指導教授) 、陳縕儂 ( 台大資工系教授 ) 以及陳昱翬 ( Google Health 成員 )。

在這場演講中，我僅針對李宏毅老師的部分作簡單扼要的摘要。

<!-- more -->

AI 如何無師自學人類語言 --- 李宏毅
---

現在的語音辨識系統，除了我們必須要收集大量的資料外，往往需要人工進行資料的標記，進而讓機器進行監督式學習。但全世界有多少種語言，我們要將所有語言都收集大量資料並進行標註是不切實際的，這也就是我們接下來想研究的 : 

機器有沒有辦法利用非監督式學習來進行語音辨識的學習 ? 
也就是說，未來我們是否可以不需要人類的指導、標註，只要給機器足夠多的資料，他便可以自動進行語音的辨識。

在語音領域中，數十年來研究方向都在 Acoustic Token Discovery 上面，也就是，假如我們給予機器大量的語音資料，它是否能從聲音訊號自行對應到特定文字片段/字母 進而理解其語意。

### 語音辨識

我們希望機器聽過大量的語音資料後，能自行將語音片段轉換出一個 Mapping Table，這樣在訓練上的確可行，但轉譯出來的 Mapping Table 往往是人類所看不懂的文字 / 符號。

因此我們可以利用大量的文字資料訓練出一個 Discriminator 可以判斷文字是否是真正人類寫出來的還是透過 Mapping Table 轉出來的文字。

接著利用這個 Discriminator 來 update 生成 Mapping Table 的 Generator，讓機器轉譯出來的文字越來越像人寫的文字，然後再 update Discriminator 讓它更嚴格的來審視 Generator ，就這樣反覆的讓機器轉譯的文字越來越分不出來到底是不是人類寫出來的東西。

利用這樣的方法最後得出來的 Phoneme Error Rate 約有 6X% 的錯誤率，並不是非常好，但這應該是全世界第一個讓機器不靠人工標註自行學習的例子。


我們想要更進一步讓準確度提高，應該怎麼做呢 ?

上面說到的整個過程中其實可以分為兩個部分 : (1) 將聲音訊號轉為 Token，(2) 再將 Token 轉成文字，其實不難想像，由於聲音訊號影響 Token 的因素甚多 ( 口音、語調... )，導致我們的 Mapping Table 很難做到非常好，也因此轉出來的文字就會無法做到完美。(根據研究，即使 Mapping Table 完美，轉出來的文字也頂多拿到 Phoneme Error Rate 約 50% )。

我們試圖將兩個部分合在一起考慮，也就是把 Token Discovery 跟 Mapping Table 兩件事情一起來考慮 --- 一個 End2End 系統。訓練這一套系統，希望最終產生出來的文字可以騙過 Discriminator。而這樣的方式也可以達到 Phoneme Error Rate 約 50%。

![](https://i.imgur.com/prm7NRh.png)

利用 GAN 的方式我們可以訓練出一個 Model V1，產生可以騙過 Discriminator 的文字，我們把這些文字當作 Label，進行下一次的訓練產生 Model V2，...

因為產生的Label 並非人類標註的，都會有一定程度的錯誤率，因此我們稱之為 Pseodo Label，利用 Boostrap 一次又一次的 Pseodo Label來訓練，最後也可以發現 Pseodo Label 的錯誤率持續下降。( 最後大概可以降到 33% 的error rate )

在同一個 Corpus 下，監督式學習可以達到約 18% error rate，但如果我們利用上述方式，再增加一些額外的 Label data，進行一個 Semi-Superviced Learning，錯誤率可以一路降到 15% 的錯誤率，也就是說，我們對機器進行ㄧ些提示，事實上機器是可以學習的更好的。

#### [後記]
1. 
李宏毅的部分後來有人詢問，既然我們的 Pseodo Label 都會有一定程度的錯誤率，為什麼經過重複的訓練後也能降低 Pseodo Label 的錯誤率呢 ?

李宏毅老師認為，這些錯誤是 random 的，而正確的部分是一致的，利用 Boostrap 或許可以藉由錯誤的 Label 來讓整個方向往更好的地方去。換一個角度來說，這些錯誤的方向或許在訓練的過程中會互相抵消，導致結果越來越好。這或許不是一個嚴謹、完美的答案，但是一個可以稍微合理解釋這樣現象的答案。

究竟這裡面發生了什麼事情? 可能在未來是可以有很大的探討空間的。

2. 
第二個問題是有人提出，今天演講的許多技術都用了 GAN，但我們知道 GAN 在訓練過程中非常不穩定，而像 Seq2Seq2Seq 這樣的 Model，再加上 GAN 我們應該如何讓整個訓練過程穩定下來 ?

李宏毅老師回答，Seq2Seq2Seq 因為中間是 Discrete ，實際上我們用到的除了是 Reinforcement Learning 訓練外，我們還使用了 Step GAN 來讓整個訓練穩定下來，意即，我們不讓整個 Reinforcement Learning 跟 GAN 看完整個文章在來看它夠不夠好，取而代之的是我們在每一個 Time Step 都評估一次，這樣的方式也的確有比較好的結果。

當然，如果可以找到不用 GAN 可以訓練得比較好的方法也是一種選擇，舉例來說，李宏毅老師實驗室在 Voice Conversion 上最近就利用完全不用 GAN 的方式來訓練，有興趣的可以查詢 " *One-shot Voice Conversion by Separating Speaker and Content Representations with Instance Normalization* " 論文。


### Voice Conversion 語音轉換

過去進行 Voice Conversion 是採用 Superviced Learning，亦即，如果我們要把 A 的聲音轉成 B 的聲音，我們必須請 A,B 兩人念相同的文字進行標註，再來訓練一組模型 (Seq2Seq) 來做轉換。

但這樣的做法非常不切實際，我們希望的是未來機器可以不需要經過上述的 Label 過程，即使是兩者不同語系都能直接進行語音轉換。

作法一樣是引進 GAN 的概念 : 

首先在訓練階段，我們收集大量的、不同人說話的語音資料，然後丟進一個 Encoder 裡面，再訓練一組 Discriminator 藉由這些 Encoder 的輸出來去判斷 Speaker 是誰。而訓練過程中，Encoder 為了對抗 Discriminator 便會漸漸地將語者的個人特徵去除，只保留說話內容特徵來騙過 Discriminator。

如果我們今天希望 B 說話然後模擬成 A 說話要怎麼做呢 ?

將 A 的語音資料被丟進訓練好的 Encoder 中產生只包含語句資訊沒有個人特徵的向量，再利用這個向量丟進 Decoder 中訓練試圖還原 A的聲音訊號。整個過程類似 AutoEncoder 的概念。

接著測試階段便是將 B 的聲音訊號丟進利用所有語者訓練出來的 Encoder 以及 語者A訓練出來的 Decoder中，就會變成 B 說的句子用 A 的語調唸出來。

但現實中，最後的測試結果並不會太好，因為訓練階段是 A2A，而測試階段是 B2A，兩階段的條件並不相同，想當然結果並不會太好，因為我們的訓練資料中可能並沒有 B 的相對應語音資料，所以 Decoder 的輸出很有可能會有很多問題。

所以我們可以在這邊另外加上兩個部分，第一個部分是另外的 Discriminator 來判斷 Decoder 的輸出是不是人聲，另外一個部分則是接上一個 Speaker Classifier 來讓機器判斷這是誰說話的聲音，這樣便可以讓 Decoder 的輸出越來越像人的聲音，且越來越像 A 說的。

實驗結果，這樣的方式都可以有不錯的結果。

這裡可以看 (聽) Voice Conversion 的一些成果 : 
https://jjery2243542.github.io/voice_conversion_demo/

### 文章摘要

過去的文章摘要方式大多是收集大量資料後，進行人工摘要，接著訓練一個 Seq2Seq Model 來進行摘要。

但這樣的方式需要非常非常大量的資料，我們不可能針對每一個語系都做這樣的語料收集，最終我們仍然希望機器可以無師自通的學習，不需要人工進行摘要標註。

一樣引入 Discriminator。

我們可以在 Seq2Seq 的輸出後加入一個 Discriminator 來判斷輸出的文字是不是人寫的文字，夠不夠自然。然而這樣一樣會有問題，反正機器只要輸出一組像人寫出來的文字即可，即使跟原文一點關係也沒有也可以成功騙過 Discriminator。

最後解決這問題的方式便是使用一個 Seq2Seq2Seq AutoEncoder Model，讓文章產生摘要再利用摘要產生回一篇文章。這樣也可以省去人工標註摘要的時間。跟上面的問題一樣，可能在中間層的輸出會是一段毫無意義或是人類看不懂的摘要，加入一個 Discriminator 即可。

這樣的模型就像是一個 GAN + AutoEncoder 的 Model 。

這裡有幾個舉例，有好有壞 : 


![](https://i.imgur.com/ON6x9br.png =500x)





![](https://i.imgur.com/AQKff6t.png =500x)



### QA system

想要教機器看文章並且回答問題，我們要有相當大量的資料，並且我們必須要想非常多的問題跟答案。但全世界有非常多的語言並沒有這麼大量的語料可供訓練，既然英文上有這麼大量的資料，我們可不可能先在英文上面進行學習，再利用訓練出來的模型運用在別的語種上 ?

要實現這樣的想法，我們可以利用上面所講到的方法。

我們可以利用中文資料與英文資料，先訓練出各自的模型，output 只剩下語義而抹去不同語言本身的特徵。再利用這種語義的特徵向量丟進這個 Shared pre-train model 來回答問題。然而為了避免 這個 pre-train model 最後仍然會將中英文各自產生的向量，最後仍然各自以中英文回答，我們可以在中間加上一個 Discriminator 來對前面的模型做對抗，讓最後的輸出可以讓語言本身的特徵越來越少，形成越來越單純的語義向量。

不過這樣的方式已經被 BERT 所取代。利用 BERT 我們甚至可以完全不需要中文訓練資料，直接以英文訓練出來的模型套用在中文上。

