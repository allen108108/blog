---
title: "Unsupervised Learning (3) -- Word Embedding"
date: 2019-10-08 00:42:04
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning
image: https://i.imgur.com/Y6gh48b.png
mathjax: true
---

在正式進入 Word Embedding 之前，我覺得應該先來談談什麼是 Word Vector。

今天看了幾個 Medium 文章，有技術部落格甚至認為 Word Embedding 就是 Word Vector。這才讓我覺得，似乎要對 Word Embedding 要有更深刻的了解才是。

<!-- more -->

## Word Vector

在 Machine Learning 中，資料的特徵表示是整個領域中最核心的部分，不同的特徵表示，可能會訓練出天差地遠的 Model。當然，在 Natural Language Processing ( NLP ) 中亦然。

然而於 NLP 中我們應該要用什麼樣子的方式來表示資料 ? 怎麼樣子的方式才能真正有效的表現文本裡面「字」、「詞」、「句」之間的意義呢 ? 此時便有了 Word Vector 的概念。

### 1-of-N Encoding ( One-Hot representation )

這是 NLP 中最直覺也是最多人使用的方法，將 One-Hot Encoding 的概念引入 NLP ，每一個字都是一個很長的向量，維度是我們手中擁有的詞數量。



![](https://i.imgur.com/Q2yK9cs.png =300x)



上面的示例，只是為了讓我們理解 1-of-N Encoding 的概念，實務上，每一個 Word Vector 可能都會是好幾萬維的稀疏矩陣。

1-of-N Encoding 直觀、簡潔，而且在演算法上計算也沒有什麼太大的瓶頸，但是這樣的 Word Vector Representation 卻有一些問題 : 

* 隨著文本的內容增加，維度會越來越大。
* 每一個 Word Vector 之間都是 Independent，無法體現上下文之間的語意相關，而這在 NLP 中是極為嚴重的問題。
* 無法加入新詞。

### Word Embedding ( Distribution representation )

Distribution Representation 最早是在 1986 年 Hinton 的論文 -- " *Learning distributed representations of concepts* " 中被提出。而 Word Embedding 則是 Distribution representation 的一種型態，可以克服 One-Hot repesentation 的缺點 。


 

![](https://i.imgur.com/Y6gh48b.png =300x)
( 圖片來源 : [DeepNLP的表示学习·词嵌入来龙去脉·深度学习（Deep Learning）·自然语言处理（NLP）·表示（Representation）](https://blog.csdn.net/scotfield_msn/article/details/69075227) )


Word Embedding 的核心概念是將每一個詞映射到一個 (相對於 One-Hot representation) 低維度的賦距空間內，而這些帶有文本訊息的低維度向量形成一個稠密的「詞向量空間」，然後利用距離來判斷其 ( 語法、語意 ) 相似性。

這才是真正「詞嵌入」的精隨所在。

值得一提的是，當我們以 Neural Network 建模時，中間的 hidden layer 都可以視為是一種 feature extraction ( 特徵萃取 )，如果利用 NN 來做 NLP 模型時，這些低維度 hidden layer 也是有捕捉上下文的能力。換句話說， Word Embedding 可以視為 NN 處理 NLP 問題下的副產品。

說起來簡單，但真正在 NLP 要取得詞向量並不是這麼容易，因為映射方式的不同、嵌入空間的大小不同...等因素，詞向量並不會是唯一的。

Word Embedding 可以簡單地做一個分類 : 

* Count-Based : 利用詞頻統計來建立 Word Vector 的一種方法。著名的方法有 -- Count Vector、TF-IDF Vector 以及 Co-Occurence Vector ( GloVe[^註1] )



* Prediction-Based : 利用神經網路來建立 Word Vector 的一種方法。常見的方法有 -- Continuous Bag of Words ( CBOW ) 及 Skip-gram。( Prediction-Base 為李宏毅教授的重點 )



![](https://i.imgur.com/K4EvdrZ.png =500x)



假如果我們要以前一個字 $w_{i-1}$ 來預測下一個字 $w_i$，那我們可以利用一組 NN 架構以及一些文本訓練資料，將文本中 $w_{i-1}$ 的 1-of-N Encoding 作為輸入，輸出則是所有文本內文字的機率值 ( 輸入輸出雖是同樣維度，但是意義卻不同。 ) ，藉此訓練出一套預測模型。



![](https://i.imgur.com/IW9Ve1s.png =500x)



當然我們也可以用前兩個字 $w_{i-2},w_{i-1}$ 來預測下一個字 $w_i$。
就直觀來看，$w_{i-2}$ 與 $w_{i-1}$ 理應具有相同的性質 (語意、上下文關係...)，才能預測出 $w_i$。因為這樣的理由，我們可以將其權重共享來體現其上述的概念。當然，也可以藉此降低參數量，減少計算成本。

理解了 Prediction-Base 概念後，前面所講的 CBOW 或是 Skip-gram 也只是其變形應用。



![](https://i.imgur.com/adgSMho.png =500x)



## Advantage of Word Embedding 

當我們可以建立出 Word Embedding 後，就可以對 Word 進行觀察、操作

* 我們可以藉由可視化來觀察字詞之間的關聯性

![](https://i.imgur.com/xBv13wS.png =330x)![](https://i.imgur.com/z0Pp3P0.png =330x)

* 我們也可以利用向量的運算法則對 Word 進行運算。

<img width=500 src="https://i.imgur.com/orAfN63.png" >

## 參考資料

1. Hung-yi Lee ,  [Machine Learning](http://speech.ee.ntu.edu.tw/~tlkagk/courses_ML17_2.html)(2017)
2. [DeepNLP的表示学习·词嵌入来龙去脉·深度学习（Deep Learning）·自然语言处理（NLP）·表示（Representation）](https://blog.csdn.net/scotfield_msn/article/details/69075227)
3. [GloVe: Global Vectors for Word Representation](https://nlp.stanford.edu/projects/glove/)
4. [GloVe:另一种Word Embedding方法](https://pengfoo.com/post/machine-learning/2017-04-11)
5. [GloVe详解](http://www.fanyeong.com/2018/02/19/glove-in-detail/)
6. [Word Embedding的发展和原理简介](https://www.jianshu.com/p/2a76b7d3126b)


註釋
---

[^註1]: 
Global Vectors for Word Representation 的簡稱。這是 Stanford University 於2014年所發表的 Word Embedding 方法，利用 Co-Occurence Matrix ( 共現矩陣 ) 與上下文的關係建立一套 Loss Function，找出最優化解時的 Word Vector。
