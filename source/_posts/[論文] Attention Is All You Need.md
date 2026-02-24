---
title: "[論文] Attention Is All You Need" 
date: 2020-05-13 15:03:40
categories:
- 論文 Paper
image: https://i.imgur.com/3Xgg8pc.png
mathjax: true
---


摘要 Abstract
---

主要的 Sequence transduction model 都是基於包含 encoder 及 decoder 的複雜 RNN 或 CNN 結構。而表現最好的模型將 encoder, decoder 與注意力機制 (Attention Mechanism) 連結在一起。這篇論文中提出了一個新的簡單網路結構 Transformer，它是一個基於注意力機制的網路結構，並且完全不含 RNN 與 CNN 。在一些實驗中都顯示 Transformer 具有優勢，除了高度可平行運算外，訓練的時間也明顯降低許多。

<!-- more -->

簡介 Introduction
---

RNN (Recurrent Neural Network, 循環神經網路) ，特別是 LSTM (Long Short-term memory, 長短期記憶) 與 GRU (Gate Recurrent)，已經成為序列建模與傳導 ( Seq2Seq ) 問題，如語言模型及機器翻譯等，的標準處理方式。而後，人們也不斷的努力擴大 RNN 與 Encoder-Decoder 的界限。

RNN 模型通常是根據輸入輸出字符的位置進行計算。先將位置與時間步對齊，然後藉由前一個位置(時間)的隱含狀態 (Hidden State) $h_{t-1}$ 以及當下的位置(時間) $t$ 生成這個時間點的隱含狀態 $h_t$。這樣的序列處理阻礙了平行化訓練，當序列的長度變長，記憶體便會限制了訓練資料的 Batch 數量。即使，較新的成果已經可以透過一些計算技巧 ( 矩陣分解、條件計算...等) 來增加計算效率，同時提高模型的表現，但，序列計算的基本限制仍然存在。

注意力機制已經成為序列建模與傳導 ( Seq2Seq ) 模型中各種任務不可或缺的一部分，不需要考慮輸入輸出序列的長度即可對依賴項進行建模。不過，除了少數情況下，注意力機制仍然會配合 RNN 一起使用。

這篇論文中，作者提出了 Transformer，一種完全基於注意力機制且避開 RNN 的模型結構，用以描繪出輸入輸出序列之間的關係。在實驗中，裡用 8 個 P100 GPU 上訓練約 12 小時，Transformer 可以達到更多的平行化運算，並且有更高水準的翻譯品質。

背景 Background
---

Extended Neural GPU, ByteNet 以及 ConvS2S 都是利用卷積來達到減少序列計算的網路結構，但隨著輸入輸出序列中兩個字符的距離增加，要建構這兩個字符的關係所需要使用到的運算、操作數量也會隨之增加 ( 在 ConvS2S 中呈線性成長，而 ByteNet 中則是呈對數增長 )。Transformer 可以將這些操作控制在一個固定的常數，雖然，注意力權重位置平均後會降低有效的解析度，但這樣的狀況可以利用多頭注意力機制 ( Multi-Head Attention ) 來抵銷。

自我注意力機制 ( Self-Attention )，也被稱為內注意力機制 ( Intra-Attention )，這是一種專注於單序列中不同位置字符關係的注意力機制，目標是要計算出此序列的一種表達形式。自注意力機制已成功用在各項任務上，如 : 閱讀理解 ( Reading Comperhension )、抽象摘要 ( Abstractive Summarization ) 、文本蘊含 ( Textual Entailment ) 以及學習與任務無關的句型表達 ( Task-Independent Sentence Representation ) 。

End2End Memory Networks 是一個基於遞迴注意力機制的網路結構，而非序列對齊的遞迴結構，這樣的模型已經被證明在一些簡單的問答系統以及語言建模任務上有良好的表現。

Transformer 是第一個完全依靠自注意力且不須使用序列對齊的 RNN 或 CNN 來計算輸入輸出序列表達的傳導 ( Seq2Seq ) 模型。在下面的各部分，論文將會描述 Transformer、引導自注意機制並且討論其之於其他模型的優勢所在。

![](https://i.imgur.com/woMGw7D.png)
(圖片來源 : [从 Transformer 说起](https://tobiaslee.top/2018/12/13/Start-from-Transformer/))
  
模型構造 Model Architecture
---


![](https://i.imgur.com/3Xgg8pc.png)



大多數具競爭力的神經序列傳導模型都具有 Encoder-Decoder 構造。Encoder 將輸入字符序列 $(x_1,x_2,\cdots,x_n)$ 映射到一個連續的表達序列 $\mathrm{z}=(z_1,z_2,\cdots,z_n)$ 。給定 $\mathrm{z}$ ，再由 Decoder 生成一個字符序列作為輸出 $(y_1,y_2,\cdots,y_m)$ 。 模型的每一個步驟都是自迴歸 ( Auto-Regressive ) 的，利用先前的輸出作為輸入來預測下一個輸出。

Transformer 遵循這樣的架構，對 Encoder 及 Decoder 都使用自我注意力機制的堆疊、Point-Wise 以及全連接層來構造，整體構造如上圖。

### Encoder 與 Decoder 堆疊

#### **編碼器 Encoder**

Encoder 的部分是由 $N=6$ 個相同的層堆疊而成，而每一個層都有兩個子層。第一個子層是多頭注意力機制，而第二個子層就是一個簡單的 Position-Wise 全連接層。除此之外，每一個子層都會設置一個 Residual Connection ，並且加入 Layer Normalization 在兩個子層中間，也就是說，每一個子層的輸出應該是

$$
\text{LayerNorm}(x+\text{Sublayer}(x))\\
$$

這裡的 $\text{Sublayer}(x)$ 指的就是將每一個子層視為一個函數來看待。為了加強這些 Residual Connection，所有子層的輸出都設定為 $d_{model}=512$。

#### **解碼器 Decoder**

Decoder 的部分一樣是由 $N=6$ 個相同的層堆疊而成，不同的地方是，在原本的兩個子層中間插入一個新的子層，其連接 Encoder 的輸出進行多頭注意力機制。與 Encoder 部分相同，每一個子層均有 Residual Connection，並進行 Layer Normalization。

除此之外，作者們修改了第一個子層，加上一個遮罩 ( Mask ) ，確保對位置 $i$ 的預測只能依賴位置小於 $i$ 的已知輸出來預測。


### 注意力機制 Attention

注意力函式，可以視為將 Query 與一組 Key-Value 映射輸出成一組向量，其中Query, Key 與 Value 亦為向量。這個函式先利用一個相容性函數 ( Compatibility Function ) 將 Query 與 Key 映射成為 Value 的權重，再將 Value 經過加權總和算出輸出。

#### 自我注意力機制 Scaled Dot-Product Attention

作者將 Transformer 中特別的注意力函式稱之為 " Scaled Dot-Product Attention "，由 Query 向量, 維度 $d_k$ 的 Key 向量以及維度 $d_v$ 的 Value 向量作為輸入。先將 Query 與所有的 Key 進行點積 ( dot product ) 後通通除以 $\sqrt{d_k}$，再使用 Softmax 函數得到 Value 的權重。

在實務上，Query, Key 與 Value 都會包在矩陣 $Q$, $K$ 與 $V$ 中進行矩陣運算 : 

$$
\text{Attention}(Q,K,V)=\text{Softmax}\Big(\dfrac{QK^T}{\sqrt{d_k}}\Big)V
$$

在注意力函式中，最常用的是 Additive Attention 與 Dot-Product Attention 兩種。Dot-Product Attention 與論文中的 Scaled Dot-Product Attention 只差在 $\sqrt{d_k}$ 的倍數關係。而 Additive Attention 則是將相容性函數由 Softmax 函數替換成單層神經網路。理論上，這兩者的複雜度接近，但在實務上，Dot-Product Attention 的運算速度快得多，而且空間效率高，因此可以利用高度最佳化矩陣乘法來實現。

當 $d_k$ 值很小時，兩種注意力函式的表現也差不多，但對於較大的 $d_k$ 來說，Additive Attention 則相較 Dot-Product Attention 有較好的表現。作者認為，較大的 $d_k$ 值，可能會使點積的結果躍升成為更高的量級，放進 Softmax 中，就會將梯度推向較小的區域內。為了沖銷這種影響，因此衍生出 Scaled Dot-Product Attention，利用 $\sqrt{d_k}$ 進行縮放。


#### 多頭注意力機制 Multi-Head Attention

與使用固定 $d_{model}$ 維度的 Queries, Keys, Values 的 Attention 函式相比，作者們發現，將 Queries, Keys, Values 分別進行 $h$ 次線性映射到維度 ${d_k}$, ${d_k}$ 與 ${d_v}$ 是有利的。每一次 Queries, Keys, Values 的映射都會平行進行 Attention 函式的運算，各產生一個維度 $d_v$ 的向量值。最後將這 $h$ 個向量拼接在一起再做一次線性映射得到最終的結果值，Scaled Dot-Product Attention 與 Multi-Head Attention 的過程可參考下圖 : 

![](https://i.imgur.com/iuSiROs.png)


多頭注意力機制允許模型在不同的位置上可以同時關聯不同表徵子空間 ( Representation Subspace ) 的訊息，如果僅用單一注意力機制，它會採平均值來削弱這個訊息。

$$
\text{MultiHead}(Q, K, V)=\text{Concat}(head_1, head_2,\cdots,head_h)W^O\\
\text{where }head_i=\text{Attention}(QW_i^Q, KW_i^K, VW_i^V)\\
\text{and }W_i^Q\in\mathbb{R}^{d_{model}\times d_k}, W_i^K\in\mathbb{R}^{d_{model}\times d_k},W_i^V\in\mathbb{R}^{d_{model}\times d_v},\text{and } W_i^O\in\mathbb{R}^{d_{model}\times d_k}
$$

在此論文中，作者們設定 $h=8$ 個平行的注意力機制層，且 $d_k=d_v=\dfrac{d_{model}}{h}$，這可以降低每一個注意力機制的維度，進而減少運算成本，保持跟單一注意力 (Single-Head Attention) 機制一樣的維度。

#### 注意力機制在 Transformer 的應用 Applications of Attention in our Model

在 Transformer 中，針對多頭注意力機制採取了三種不同的模式 : 

* **在 Encoder-Decoder Attention 層** 
使用先前 Decoder 的輸出作為 Query，以 Encoder 的輸入作為 Key 與 Value  (因此這一層並非自我注意力層 Self-Attention)。這樣的設計可以使每一個位置在 Decoder 中都可以得到輸入序列的其它位置的訊息。

* **在 Encoder 中**
Encoder 中包含了一組自我注意力層 ( Self-Attention Layer )，所有的 Queries, Keys, Values 都來自於同一個位置，在這個情況中，是指 Encoder 前面幾層的輸出。這樣的設計也使每一個位置在 Encoder 中都可以得到先前層中所有輸出。 
 
* **在 Decoder 中** 
Decoder 中的遮蔽自我注意力層 ( Mask Self-Attention Layer )，只允許 Decoder 可以關注到包含自身位置之前所有位置的訊息。會這樣設計的緣故主要是為了防止訊息往左 (前) 流動，畢竟每一個位置的預測都應該只與前面的位置有關，不該讓後面位置的資訊影響到當下位置的預測。在論文中，便將非法連接 (後向前傳遞) 的 Softmax 值均設置為 $-\infty$，達成 Mask 的效果。


#### **[ 補充 -- About Attention ]**

看了論文中上述對注意力機制與 Transformer 的介紹，或許還是沒辦法太能理解究竟在 Transformer 中是怎麼運用注意力機制的，因此筆者嘗試利用一個實際例子再根據一些參考資料重新解釋一次 Transformer 與注意力機制的關係。

**例子 : 利用 Transformer 進行 " *I arrived at the* " 語句進行翻譯。**

在 Encoder 的部分，輸入語句先利用一次權重生成 Query, Key 與 Value，再分別給予個別不同的權重生成不同的 Query, Key 與 Value 並同時進行注意力機制，利用這樣多頭注意力積只可使系統能夠更廣泛的抓取輸入語句的表徵，最後再將不同注意力機制下生成的輸出拼接成一個向量，詳細如下圖 : 

![](https://i.imgur.com/opC8sB0.png)

然而，Transformer 並非完全沒有使用到 RNN 的方式，在 Decoder 中，要生成語句或翻譯，還是得利用 RNN 的模式來循環輸出。當我輸入 " *I arrived at the* "，在 Decoder 的部分就會「依序」生成 " *Je suis arrivé* "。

在這個部分，輸入不單單使用 Encoder 的輸出，也會將先前生成的語句一同作為輸入，如此一來便可獲取輸入的資訊。

![](https://i.imgur.com/G9zAO4m.png)

我們可以利用 [Google AI Blog "Transformer: A Novel Neural Network Architecture for Language Understanding"](https://ai.googleblog.com/2017/08/transformer-novel-neural-network.html) 中的一個 GIF 動畫來理解 Transformer 的動態生成過程 : 

![](https://i.imgur.com/5cmTFpu.gif)

#### **[ 補充 -- About Mask ]**

我們要先了解到，真正會用到 Mask 的部分是在訓練的階段，因為在訓練過程中，可能因為每個 Batch 輸入的序列長度不同或是最後生成的結果完全可見的原因，我們必須要做某種程度上的 Mask 來保證訓練過程不會有問題。

真正在 Transformer 中使用到的 Mask 技巧有兩種 : Padding Mask 與 Sequence Mask。在實務上，無論是 Encoder 還是 Decoder 層都會使用到 Padding Mask，主要是限制輸入序列的長度必須要一樣，具體來說， Padding Mask 就是語句長度的對齊，將短語句的後面通通補上 0。

比較重要的是 Sequence Mask，當我們在訓練 Transformer 時，由於結果可見，但又必須要避免生成某位置時不能被其後的「答案」影響到生成，因此必須要進行 Mask，在實際操作上，會利用類似在實際操作上，會利用類似一個三角矩陣的概念來對應到訓練資料的Label上。

如果拿上面翻譯的例子來看，" *Je suis arrivé* " 在訓練過程中應該會像下面這樣 : 

<center>

![](https://i.imgur.com/Jh53wzI.png =350x)


</center>

有人認為 Mask 的概念正是 Transformer 成功的關鍵之一，利用上述的概念，我們可以用來表徵語句的各種順序。

![](https://i.imgur.com/R8mXjVh.png)


### Position-Wise 前饋神經網路 Position-Wise Feed-Forward Network

除了注意力機制層外，無論在 Encoder 還是 Decoder 部分都還包含一個全連接前饋神經網路，這個神經網路對每一個位置的向量個別進行相同的操作。在這個全連接層中，包含了兩次的線性轉換以及中間加入一個 ReLU activation function。

$$
\text{FFN}(x)=\max(0,xW_1+b_1)W_2+b_2
$$

不同位置的輸入共用同一個前饋網路，但，這個兩層的全連接層的權重並不共用。換個角度來說，這個全連接層可以看做是兩個 $1\times 1$ 的卷積層。輸入、輸出的維度都是 $d_{model}=512$，中間層維度則是 $d_{ff}=2048$。

### 向量嵌入與 Softmax Embedding & Softmax

與一般的 Seq2Seq 模型類似，我們可以利用已經學習到的模型將輸入輸出 token 轉換成維度 $d_{model}$ 的向量，然後再利用模型中的線性變換以及 Softmax 函數將 decoder 的輸出轉換成為預測 token 的機率。在 Transformer 中，作者們設計使兩個嵌入層以及 Softmax 變換間共享權重，並且在嵌入層終將所權重均除以 $\sqrt{d_{model}}$。


### 位置編碼 Position Encoding

由於 Transformer 並不包含 RNN 與 CNN，因此如果我們仔細看注意力機制會發現整個過程獨立於序列的順序 ( 也就是說，" *I arrived at the* " 與 " *I at the arrived* " 並無相異 )，為了讓模型的運作可以考慮到序列的順序，因此，作者在 Encoder 與 Decoder 底部輸入均加上了位置編碼 ( Position Encoding )。由於輸入序列與位置編碼的維度相同因此我們可以直接進行相加。至於如何為位置進行編碼 ? 可以利用學習而得也可以使用固定編碼 ( 類似 One-Hot enoding )。

在 Transformer 中，作者們將位置編碼做了如下的設計 : 

$$
PE_{(pos,2i)}=\sin(pos/10000^{2i/d_{model}})\\
PE_{(pos,2i+1)}=\cos(pos/10000^{2i/d_{model}})
$$

$pos$ 指的是這個 token 在序列中的位置，而 $i$ 代表的是這個位置編碼的維度位置。然後整個位置編碼的維度會對應到一個正弦曲線。上面的設計使得波長會介於 $2\pi$ 到 $10000\cdot 2\pi$ 之間。使用這樣的函數設計主要是因為作者們認為這樣的函數可以有利於作者們認為這樣的函數可以有利於模型輕鬆的學習到位置的編碼，因為對任意的偏移量 $k$ 來說，$PE_{pos+k}$ 都可以表示成為 $PE_{pos}$ 的線性組合。

#### **[ 補充 : Position Encoding 設計  ]**

最直覺的位置編碼就是直接利用位置 $i$ 或是 One-Hot Encoding 的概念進行編碼，但是這樣的方式在長文本的資料中前者會造成數值爆炸，而後者，則很容易導致維度大爆炸。

因此，在整個位置編碼上面，我們會希望這樣的編碼可以符合幾個條件 : 

1. 必須可以識別一個單字(詞)在不同位置上在不同位置上可能有不同的意義
2. 必須可以體現不同先後順序的關係
3. 編碼數值落在區間 $[0,1]$ 之間
4. 編碼維度可以不受限於文本長度

這就是為什麼論文會引入三角函數來做為位置編碼之中。利用 $\sin$ 及 $\cos$ 有上下界以及具穩定週期循環的特性，來達成上面的條件。

倘若使用如下簡單的三角函數位置編碼 : 

$$
PE_{pos}=\sin(\dfrac{pos}{\alpha})\\
\text{where }PE_{pos} : \mathbb{Z}\rightarrow[-1,1] 
$$

雖然可以利用 $\alpha$ 來調節三角函數波長，但仍顯過於單調，且仍然可能造成不同位置具有相同編碼的情況出現。

因此，為了使整個 Word embedding 的維度是 $d_{model}$，又希望位置編碼具有一定的複雜性，讓每一個維度都可以用不同的函數來進行編碼，所以論文中將每一個維度都用不同的 $\alpha=10000^{2i/d_{model}}$ 來控制波長，且將 $\sin$ 及 $\cos$ 交替在不同的維度上使用。


#### **[ 補充 : 將輸入序列融入位置資訊為何可以用相加而不是直接拼接的方式 ?  ]**

在李宏毅教授的 Machine Learning 中，在 Attention 的部分有針對這個地方做解釋，先說結論，其實兩個概念最終會得到相同的結果。

假設輸入序列向量為 $x^i$ ，而其位置編碼使用最簡單的方式 $p^i=(0,0,\cdots,0,1,0,\cdots,0)$

從整個矩陣的運算可由下圖清楚了解 : 

![](https://i.imgur.com/KgdqdW2.png)

我們將位置編碼直接與 token 拼接在一起經過線性轉換後的結果，會與token 與位置編碼各自進行線性轉換後再相加的結果相同。

為什麼是自我注意力機制 ? Why Self-Attention ?
---

在這個部分，作者比較了在這個部分，作者將自我注意力機制與遞迴層、卷積層這種常用來對符號序列表徵 $(x_1,x_2,\cdots,x_n)$ 映射到另一個相等長度的表徵序列 $(z_1,z_2,\cdots,z_n)$ , $x_i,z_i\in\mathbb{R}^n$ 的方式 ， 例如典型 Encoder / Decoder 中的隱藏層 ， 進行比較。

作者們從三個部份來解釋為什麼要使用自我注意力機制。

![](https://i.imgur.com/dd28sYK.png)


1. **每層的總計算複雜度**
2. **可平行運算的能力** : 這部分是利用所需序列操作的最少數量來衡量
3. **網路之間針對遠距離依賴性的路徑長度** : 在許多序列傳導任務中，學習遠距離依賴性是非常關鍵的挑戰之一。而影響學習依賴性的一個關鍵因素是訊號向前向後傳遞所必須經過的路徑長度。輸入與輸出序列中任意位置之間的組合距離越短，將會使得長距離依賴性的學習更加容易。因此在這邊作者們便比較了不同型態網路層所組合成的網路中任意輸入、輸出的位置的最大路徑長度。

在上表中自我注意力層僅需要 $O(1)$ 的複雜度，也就是固定數量的序列操作就可以連結全部的位置，且當序列長度 $n$ 小於維度 $d$ 時， 這是大多數先進模型在機器翻譯中句型表徵 ( Sentence Representation ) 會常見的情形 ，自我注意力的複雜度也比遞迴層要來的低。

為了提高極長序列任務的計算能力，可以在自我注意力機制上添加一些限制，例如僅考慮輸入序列中以每個輸出位置為中心，半徑為 $r$ 的鄰域。也就是說，我們不對整個序列進行注意力機制，僅對以 $r$ 為半徑的相鄰位置進行自我注意力。如此一來，便可以增加依賴距離到 $O(n/r)$，作者們將會在往後的研究中討論這個部分。
 
 
一個 kernel 寬度為 $k<n$ 的單一卷積層，無法連接所有成對的輸入輸出位置。想要這樣做就必須在連續 kernels 中堆疊 $O(n/k)$ 個卷積層，而在擴張卷積 ( Dilated Convolution ) 層中則需要 $O(\log_{k}n)$。這樣的方式，增加了長距離的依賴性，但卷積層的複雜度也比遞迴層多了 $k$ 倍。雖然，可分離卷積 ( Separable Convolution ) 層將複雜度降低至 $O(k\cdot n\cdot d+n\cdot d^2)$ ，但這樣的複雜度，即使在 $k=n$ 的狀況下，也相當於論文中採用的自我注意力層與全連接層的複雜度相加。

自我注意力機制另外有附帶的優點，即自我注意力機制可以產生更多的解釋力，讓整個模型的可解釋性增加，在論文的附錄中將會介紹並討論範例。在多頭注意力機制中，每一個 head 不僅可以學習到不同的任務，而且多頭注意力機制似乎可以將句法句型與語意結構進行連結。

訓練 Training
---

### 訓練資料與批量 Training Data and Batching

論文中對標準 WMT 2014 英-德語言資料集進行訓練，其中包含了大約 450 萬組句子對。使用 Byte-Pair Encoding 對句子進行編碼，其中共享一個約 37000 tokens 的來源-目標詞彙集。對於英-法語言，作者們使用一個更大的 WMT 2014 英-法語言資料集來做訓練，此資料集包含了約 36M 組句子對，且將標註拆成 32000 筆字彙。

將序列長度相近的句子對視為一組，每一次批量訓練都包含了一組句子對，各包含了約 25000 個來源 tokens 與目標 tokens。


### 硬體與時程 Hardware and Schedule

使用一台具備 8 顆 Nvidia P100 GPU 的電腦進行訓練，對於論文中所設置超參數的基本模型，每一次訓練 ( training step ) 約 0.4 秒，共進行約  100,000 次或 12 小時的訓縣。對於大型模型而言，則每次訓練時間需要約 1.0 秒，一共接受了 300,000 次或約 3.5 天的訓練時間。 

### 最佳化器 Optimizer

訓練中所使用的 Optimizer 為 Adam，其參數設置如下 : 

$$
\beta_1=0.9\\
\beta_2=0.98\\
\epsilon=10^{-9}\\
LR=d_{model}^{-0.5}\cdot\min(step\text{_}num^{-0.5},step\text{_}num\cdot warmup\text{_}steps^{-1.5})
$$

利用這樣的學習率設置方式，可以在訓練中間根據訓練的狀況不斷調整學習率，先根據 $warmup\text{_}steps$ ( 論文中設置為 4000 ) 線性增加學習率，再經由訓練次數的平方根倒數來降低學習率。


### 正規化 Regularizer

在整個訓練過程中，一共用了三種正規化方式 : 

* **Residual Dropout**

在每一個子層的輸出後，Normalization 前都添加 Dropout，此外，在 Encoder 與 Decoder 中，對於 token 與 位置編碼加總的部分也添加 Dropout。 其中， $P_{drop}=0.1$

* **Label Smoothing**

訓練中還進行了 $\epsilon_{ls}=0.1$ 的 Label Smothing。雖然這樣會讓整個模型變得更加不穩定，但卻提高了準確度與 BLEU 分數。



結果 Result
---

### 機器翻譯 Machine Translation

在 WMT 2014 英德語言資料翻譯任務中，大型的 Transformer 模型的表現比先前最佳的模型高了 2.0 BLEU 分數，這締造了 BLEU 的最高 28.4 分紀錄。大型 Transformer 的模型設定可以參考下表 Table 3 ，利用 8 顆 Nvidia P100 GPU 訓練 3.5 天的時間。從結果來看，Transformer 勝過了之前其他的模型及其組合，且訓練成本相較於這些模型來說僅占一小部分。

在 WMT 2014 英法語言資料翻譯任務中，大型 Transformer 的 BLEU 分數可以高達 41.0，勝過先前發布的任一個模型，而且訓練成本不到這些模型的 1/4。在英法翻譯任務中，所訓練的 Transformer ，其 Dropout 設定為 $P_{drop}=0.1$ ，而非 $0.3$。

對於基本的 Transformer 模型來說，作者使用的是最後 5 個 Checkpoints 之平均來得到最終的模型，而對於大型 Transformer ，則是透過最後 20 個 Checkpoint 平均來取得最終模型。而作者們利用 $B=4$ 及 $\alpha=0.6$ 的定向搜尋 ( Beam Search ) ，這些參數的決定是經過實驗後選擇的。在推論的部分，作者們設置了最大輸出長度=輸入長度 $+50$，但可以提早終止。

下表總結了實驗的結果，並將 Transformer 與其他的模型架構進行翻譯直向語訓練成本的比較。其中，透過 GPU 數量、訓練時間以及單一 GPU 持續單精度浮點能力 ( Sustained Single-Percision Floating-Point Capacity ) 的估計相乘來估算浮點運算的數量。

![](https://i.imgur.com/YMX5gsZ.jpg)

### 模型變體 Model Variations

為了評估 Transformer 各個不同組件的重要性，作者們利用不同的方式改變基本 Transformer ，並在 newatest2013 開發資料集中測量了英德翻譯的變化。使用了前面提到的定向搜索但不進行 Checkpoint 平均，整個結果呈現在下表 Table 3 中。

在 Table 3 (A) 列中，作者們改變了 Head 數量、Key 與 Value 的大小使其計算量保持不變，儘管單頭注意力機制較多頭注意力機制的表現來的差，但 Head 數量過多時，整個模型質量也會下降。

在 Table 3 (B) 列中，作者發現當減少 $d_{Key}$ 值會損害模型的質量，這表明了決定相容性函數並非是一件容易的事情，且相較於點積運算更複雜的相容性函數可能會有其益處。在 (C) (D) 列中，可以更進一步的發現，正如期待較大的模型表現會較好，且 Dropout 有助於避免 Overfitting。在 (E) 列中，作者將學習而得的位置編碼以正弦函數來做替換，可以發現結果會與基本 Transformer 的結果幾乎相同。

![](https://i.imgur.com/rinckGs.png)



結論 Conclusion
---

在這篇論文中，提出了 Transformer，這是完全基於注意力機制的第一個序列傳導模型，使用多頭注意力機制代替了 Encoder-Decoder 結構中常用的遞迴層。

對於翻譯任務，與基於遞迴層或是卷積層的結構相比，可以大大增加 Transformer 的訓練速度，驗 WMT 2014 英德語言資料與 WMT 2014 英法語言資料中，Transformer 都可以達到最佳的表現。在前者中，甚至可以超過所有的模型及其組合模型。

作者們對於基於注意力機制的模型未來感到興奮，並且計畫將其應用在其他任務中。未來計畫將 Transformer 擴展到非文本以外關於輸入、輸出模式上的問題之中，並且進一步研究局部、受限制的注意力機制來有效的處理更大的輸入、輸出序列，例如圖像、音訊以及影像。另外，讓生成具有較低順序性也是另一個研究的方向之一。

參考資料 Reference
---

1. [论文解读:Attention is All you need](https://zhuanlan.zhihu.com/p/46990010)
2. [Transformer -encoder mask篇](https://medium.com/data-scientists-playground/transformer-encoder-mask%E7%AF%87-dc2c3abfe2e)
3. [Transformer -decoder mask篇](https://medium.com/data-scientists-playground/transformer-decoder-mask%E7%AF%87-77130461cd63)
4. [从 Transformer 说起](https://tobiaslee.top/2018/12/13/Start-from-Transformer/)
5. [从语言模型到Seq2Seq：Transformer如戏，全靠Mask](https://kexue.fm/archives/6933)
6. [transformer的Position encoding的总结](https://zhuanlan.zhihu.com/p/95079337)
7. [如何理解Transformer论文中的positional encoding，和三角函数有什么关系？](https://www.zhihu.com/question/347678607/answer/864217252)
8. [Transformer: A Novel Neural Network Architecture for Language Understanding](https://ai.googleblog.com/2017/08/transformer-novel-neural-network.html)
9. [The Illustrated Transformer](http://jalammar.github.io/illustrated-transformer/)
10. [Hung-yi Lee : Transformer](https://www.youtube.com/watch?v=ugWDIIOHtPA&t=2026s)


 