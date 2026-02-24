---
title: "[論文] Identity Mappings in Deep Residual Networks" 
date: 2019-11-14 01:00:00
categories:
- 論文 Paper
- 卷積神經網路 Convolutional Neural Network
image: https://i.imgur.com/MJDlCqV.png 
mathjax: true
---

概要 Abstrct
---

深層殘差網路不管在預測準確度以及收斂速度上都有著非常好的表現。在這篇論文中，作者們從前向及反向的信息傳播分析中，提出了將 shortcut connection 以及相加後的 activation 均使用恆等映射 (identity mapping) ，信息能夠藉由前後向傳播在 block 間進行傳遞。這篇論文中，利用一系列的「消融實驗」( ablation experiments )[^註1]來確定這些恆等映射的重要性。

<!-- more -->

這樣的結論激勵了研究團隊提出了一個嶄新的 Residual unit，讓訓練變得更加簡單也能增強模型的泛化能力。研究團隊利用這種結構，在各個資料集的預測結果均有成長，而且在 1001 層的 ResNet 上也能得到很好的結果。( code : https://github.com/KaimingHe/resnet-1k-layers )


簡介 Introduction
---

在 ResNet 論文中，每一個殘差單元 (residual unit) 我們可以用下面的式子來表示 : 

$$
\mathrm{y}_l=h(\mathrm{x}_l)+\mathcal{F}(\mathrm{x}_l,\mathcal{W}_l)\\
\mathrm{x}_{l+1}=f(\mathrm{y}_l)=ReLU(\mathrm{y}_l)
$$
 
$\mathrm{x}_l$ 與 $\mathrm{x}_{l+1}$ 是第 $l$ 個殘差單元的輸入及輸出，$\mathcal{F}$ 是殘差函數 (residual function) ，$f$ 是 activation function，在論文中用的是 ReLU ，而 $h$ 則是恆等映射。

100多層的 ResNet 在各項資料集中都能有極佳的表現，最主要的想法就是學習一個對應於恆等映射 $h$ 的函數 $\mathcal{F}$ 。而這樣的 $h$ 在整個 model structure 中就是藉由 Shortcut connction 來達成。

在此篇論文中，作者們藉由創造一個可以計算整個網路結構 ( 不是只有殘差單元 ) 信息傳導的直接路徑來分析深層殘差網路，藉由整個推導作者發現若讓 $h$ 與 $f$ 同為恆等映射，則信息無論在前向或反向傳播中均可以在殘差單元間直接傳遞。從論文裡的各項實驗數據可以證明當模型越接近上面兩個條件時會更易於訓練。

為了更了解 Skip/Shortcut Connection 的運作，研究團隊分析並比較多種 $h(\mathrm{x}_l)$ 的型態，發現上一篇論文使用的 $h(\mathrm{x}_l)=\mathrm{x}_l$ 可以獲得最低的 error 以及最快的收斂速度。在 skip connection 上的任何操作 ( 縮放、門控、$1\times 1$ 卷積 ) 都會造成較高的 training loss 及 error。這些實驗結果都表明了，一條越 " 乾淨 " 的信息路徑，能使訓練變得更加容易。

![](https://i.imgur.com/bjcq2D4.png)

PS: 上圖跟上一篇論文的殘差塊 ( residual block ) 呈現方式相反。在這篇論文中，灰色箭頭的部分指是 shortcut connection，旁邊分支出來的才是原本的信息路徑。

另一個部分，想要建造一個 $f(\mathrm{y}_l)=\mathrm{y}_l$ 這樣的恆等映射，作者在原本的信息路徑上的 activation function ( $ReLU$ 與 $BN$ ) 從傳統的「後激活」( Post-Actvation ) 轉換成「預激活」( Pre-Activation )。這種新型態的殘差塊 (上圖右(b)) 用在 1001 層的 ReaNet 上面在 CIFAR-10/100 上訓練，整個模型訓練變得更加容易，而且泛化程度更佳。而原本在 200層 ResNet 上會出現的 overfit 現象也可以藉由這種新的殘差塊獲得改善。

這種種結果都顯示，作為深度學習的關鍵，模型的「深度」仍然有很大的拓展空間。

深層殘差網路分析 Analysis of Deep Residual Networks
---

在前一篇論文中，ResNet 是藉由堆疊相同形狀的殘差塊而形成的模組化結構。在本篇論文中，作者將原本的殘差塊改稱為殘差單元 ( Residual Units )。原始論文中的殘差單元是經由下列計算進行

$$
\mathrm{y}_l=h(\mathrm{x}_l)+\mathcal{F}(\mathrm{x}_l,\mathcal{W}_l)\\
\mathrm{x}_{l+1}=f(\mathrm{y}_l)
$$

$\mathrm{x}_l$ 指的是第 $l$ 個殘差單元的輸入特徵向量， $\mathcal{W}_l=\{\mathrm{W}_{l,k}|_{1\leq k\leq K} \}$ 代表了第 $l$ 個殘差單元的權重 (包含偏差 bias)，而 $K$ 則是單一殘差單元內的層數。$\mathcal{F}$ 是一個殘差函數，$f$ 則是在元素級加法 ( element-wise addition ) 後的操作，在原始論文上 $f=ReLU$，而 $h$ 則是一個恆等映射 $h(\mathrm{x}_l)=\mathrm{x}_l$。

若 $f$ 亦為恆等映射，則
$$
\mathrm{x}_{l+1}=f(\mathrm{y}_l)=\mathrm{y}_l=h(\mathrm{x}_l)+\mathcal{F}\mathrm{x}_l,\mathcal{W}_l)=\mathrm{x}_l+\mathcal{F}(\mathrm{x}_l,\mathcal{W}_l)
$$

又， $\mathrm{x}_{l+2}=\mathrm{x}_{l+1}+\mathcal{F}(\mathrm{x}_{l+1},\mathcal{W}_{l+1})=\mathrm{x}_l+\mathcal{F}(\mathrm{x}_l,\mathcal{W}_l)+\mathcal{F}(\mathrm{x}_{l+1},\mathcal{W}_{l+1})$，所以我們可以知道，對任意的 $L\geq l$ 而言 

$$
\mathrm{x}_{L}=\mathrm{x}_l+\sum\limits_{i=l}^{L-1}\mathcal{F}(\mathrm{x}_i,\mathcal{W}_i)
$$

上式有幾個很不錯的特性 : 

1. 一個特徵向量 $\mathrm{x}_L$ 可以由較淺層的特徵向量 $\mathrm{x}_l$ 加上一個殘差函數 $\sum_{i=l}^{L-1}\mathcal{F}$ 來取代，模型在單位 $L$ 到單位 $l$ 間是處於一個殘差的形式。
2. 若 $l=0$，則第 $L$ 殘差塊的輸入可以表示為 $\mathrm{x}_{L}=\mathrm{x}_0+\sum\limits_{i=0}^{L-1}\mathcal{F}(\mathrm{x}_i,\mathcal{W}_i)$，也就是說可以視為 $\mathrm{x}_0$ 加上第 $L$ 單位前的所有殘差函數的輸出。這與普通網路 (plain network) 不同，普通網路第 $L$ 層的輸入特徵 $\mathrm{x}_L$ 是一連串的矩陣向量積 $\prod_{i=0}^{L}W_i\mathrm{x}_0$。

除了這些特性外，上式還有很好的反向傳播性質。假設 Loss function 為 $\varepsilon$，從微積分鏈鎖律 (chain rule) 可以推得反向傳播

$$
\frac{\partial\varepsilon}{\partial\mathrm{x}_l}=\frac{\partial\varepsilon}{\partial\mathrm{x}_L}\frac{\partial\mathrm{x}_L}{\partial\mathrm{x}_l}=\frac{\partial\varepsilon}{\partial\mathrm{x}_L}\Big(1+\frac{\partial}{\partial\mathrm{x}_l}\sum\limits_{i=l}^{L-1}\mathcal{F}(\mathrm{x}_i,\mathcal{W}_i)\Big)
$$

這樣的推導指出梯度 $\frac{\partial\varepsilon}{\partial\mathrm{x}_l}$ 可以被分解成兩項 : $\frac{\partial\varepsilon}{\partial\mathrm{x}_L}$ 與 $\frac{\partial\varepsilon}{\partial\mathrm{x}_L}\frac{\partial}{\partial\mathrm{x}_l}\sum_{i=l}^{L-1}\mathcal{F}(\mathrm{x}_i,\mathcal{W}_i)$。前者指的是直接的信息傳導，而不需考量其他的權重層。而後者則是經由權重層進行信息傳導。加上 $\frac{\partial\varepsilon}{\partial\mathrm{x}_L}$ 這項可以確保信息能傳回任意淺層 $l$。此外，這式也保證了不管梯度有多小， $\frac{\partial\varepsilon}{\partial\mathrm{x}_l}$ 不會發生梯度消失的狀況，因為 $\frac{\partial}{\partial\mathrm{x}_l}\sum_{i=l}^{L-1}\mathcal{F}(\mathrm{x}_i,\mathcal{W}_i)$ 不會恆為 $-1$。

### Discussions

上面的種種推導確定了信號不論前後向均可以在任兩層中間隨意傳播。而這些推導結果的前提就是 $h$ 與 $f$ 均為恆等映射。

上面說的這些「直接」傳導的信息流就是最上面圖片中的灰色箭頭。而要滿足 $h$ 與 $f$ 均為恆等映射就是指這些灰色箭頭的部分都必須不含任何操作，意即，這些灰色箭頭部分都要是「乾淨」的。接下來論文會針對這兩個條件個別進行分析。

論 Identity Skip Connections 的重要性 
---

讓我們考慮一個狀況 $h(\mathrm{x}_l)=\lambda_l\mathrm{x}_l$ 破壞恆等映射的 shortcut connections )

$$
\mathrm{x}_{l+1}=\lambda_l\mathrm{x}_l+\mathcal{F}(\mathrm{x}_l,\mathcal{W}_l)
$$

在這裡，$\lambda_l$ 我們稱作調節純量 ( modulating scalar )，且為了簡化討論，我們仍然假設 $f$ 是恆等映射。因此，我們可以將 $\mathrm{x}_L$ 的式子做改寫 : 

$$
\mathrm{x}_L=\big(\prod\limits_{i=l}^{L-1}\lambda_i\big)\mathrm{x}_l+\sum\limits_{i=l}^{L-1}\big(\prod\limits_{j=i+1}^{L-1}\lambda_j\big)\mathcal{F}\big(\mathrm{x}_i, \mathcal{W}_i\big)\\
=\big(\prod\limits_{i=l}^{L-1}\lambda_i\big)\mathrm{x}_l+\sum\limits_{i=l}^{L-1}\hat{\mathcal{F}}\big(\mathrm{x}_i, \mathcal{W}_i\big)
$$

這裡用 $\hat{\mathcal{F}}$ 來表示吸收了 $\prod\limits_{j=i+1}^{L-1}\lambda_j$ 這些純量的 residual function $\mathcal{F}$。我們也可以簡單地改反向傳播式 : 

$$
\frac{\partial\varepsilon}{\partial\mathrm{x}_l}=\frac{\partial\varepsilon}{\partial\mathrm{x}_L}\frac{\partial\mathrm{x}_L}{\partial\mathrm{x}_l}=\frac{\partial\varepsilon}{\partial\mathrm{x}_L}\Big(\big(\prod\limits_{i=l}^{L-1}\lambda_i\big)+\frac{\partial}{\partial\mathrm{x}_l}\sum\limits_{i=l}^{L-1}\hat{\mathcal{F}}(\mathrm{x}_i,\mathcal{W}_i)\Big)
$$


針對一個極為深層 ($L$ 非常大) 的網路結構，若 $\lambda_i>1, \forall\ i$ 則這些 $\lambda_i$ 的連乘積將會非常大; 若 $\lambda_i<1, \forall\ i$，則這些 $\lambda_i$ 的連乘積將會非常小甚至是消失。這會造成反向傳播的過程中訊號強制要從權重層傳遞，使得優化過程變得非常困難。

根據上面的分析，我們可以瞭解到，在前一篇論文中為什麼會用恆等映射作為 shortcut connections，而不是使用 $h(\mathrm{x}_l)=\lambda_l\mathrm{x}_l$。

如果，skip connections  $h(\mathrm{x}_l)$ 的型態變得更加複雜 ( 加了門控或是 $1\times 1$ 卷積 )，則反向傳播式中的第一項將會被改寫成 $\prod_{i=l}^{L-1}h_i'$，其中 $h'$ 是 $h$ 的導函數。 這些連乘積可能也會阻礙信息的傳播以及訓練過程，如同此論文的實驗所示。

### 在 Skip Connections 上的實驗

( 實驗過程部分我只簡單介紹，如果有興趣的可以詳見論文 )

此部分實驗有幾個重點 : 
* 主要使用 110層 ResNet (ResNet-110) 在 CIFAR-10上做的實驗
* $f$ 使用的是 $ReLU$
* 為了避免隨機性影響結果，每一個實驗都會重複五次取 accuracy 中位數。

![](https://i.imgur.com/BnB9s1Q.png)

研究團隊利用上述五種不同結構的 Skip connections 來看看是否會對結果造成影響 : 
* Constant scaling
針對所有 shortcut connection 都乘上 $0.5$ 倍。而且針對 $f$ 有/無乘上 $0.5$ 倍均進行分析。
* Exclusive gating
由 Highway Network 衍生出來的門控系統，透過 $1\times 1$ 卷積來產生 $g(\mathrm{x})=\sigma(W_g\mathrm{x}+b_g)$。利用 $\mathcal{F}\cdot g$ 以及 $h\cdot (1-g)$ 來個別調節權重層及 Skip 連結的信息傳遞。
* Shortcut-only gating
$\mathcal{F}$ 不進行信息調節，僅 Skip 連結進行信息傳遞的調節。
* $1\times 1$ convolutional shortcut
利用 $1\times 1$ 卷積來取代恆等映射。
* Dropout shortcut
在 Skip 連結上使用了 0.5 dropout。

這些不同的構造所得出的實驗結果如下

![](https://i.imgur.com/4TC59Vw.png)


### 討論

結論如同最前面說過的，在 Skip Connection 上進行任何操作都會阻礙訊息的傳遞，使優化過程變得極為困難。

在這裡比較值得注意的部分是 $1\times 1$ convolutional shortcut。在這樣的捷徑設計中，確實引入了較多的參數，理應有更強的表述能力。而我們仔細看一下上述五種結構，其中 Shortcut-only gating 以及 $1\times 1$ convolutional shortcut 這兩種的解空間應該都是包含恆等映射的解空間，但從實驗中看來，這兩者的訓練誤差卻仍然比恆等映射的誤差還要來的高。這也告訴我們，模型退化的問題跟表述能力並沒有關係，主因還是在優化過程困難。


論 Activation Function 的使用方式
---
上面的所有分析上，我們都是建立在 $f$ 是兩個路徑相加後接一個恆等映射的前提下進行。然而上一個部分的實驗我們仍然是使用 $f=ReLU$。因此我們可以說，前面推導的公式就是實驗的估計。接下來的部分就是討論 $f$ 的影響。

作者想要利用 activation function ( $ReLU$ 與 $BN$ ) 的重新排列來使 $f$ 也是一個恆等映射。

![](https://i.imgur.com/MJDlCqV.png)

* 上圖( a ) : 這是上一篇論文的結構。$BN$ 接在權重層後，而除了最後一個 $ReLU$ 是接在連接加法之後，其餘所有的 $ReLU$ 都接在 $BN$ 之後。
* 上圖( b-e ) : 則是此論文研究的重點，將在以下作介紹。

### 在 Activation 上的實驗

( 實驗過程部分我只簡單介紹，如果有興趣的可以詳見論文 )

這部分的實驗重點如下 : 
* 作者們使用的是 110層 ResNet (ResNet-110) 以及 164層 Bottleneck 結構 (ResNet-164) 於 CIFAR-10 上做實驗。

Bottleneck 的殘差單元包含了一個 $1\times 1$ 卷積用來降維，$3\times 3$ 及$1\times 1$ 卷積來復原維度，這樣的複雜度幾乎等同於兩個$3\times 3$ 卷積層所構成的殘差單元。

* BN after addition 上圖( b ) 
在將 $f$ 轉換成恆等映射之前，我們先反其道而行，將 $BN$ 放置在連結加法之後，這樣 $f$ 就包含了 $ReLU$ 與 $BN$。利用這樣的結構進行實驗，結果會比 Baseline 還差。研究團隊認為 $BN$ 改變了經由 Skip connections 的信息並阻礙了信息的傳遞。這些從一開始訓練就發生訓練損失下降困難的狀況可以看的出來。


<img width=500 src="https://i.imgur.com/jhWVp93.png" >


* ReLU before addition 上圖( c )
要使 $f$ 成為恆等映射的一個天真的作法是將 $ReLU$ 放在連接加法之前。然而這卻使原本對應域在 $[-\infty,+\infty]$ 的 $\mathcal{F}$ 變成一個非負函數。這樣的結果會導致整個信息傳導會變成單調遞增，影響了表徵的能力。想要控制 $\mathcal{F}$ 的對應域在 $[-\infty,+\infty]$，就必須由下面兩種方式來呈現。

* Post-activation or pre-activation ?
在原始的結構設計上， activation function 會直接影響到下一個殘差塊的兩條路徑 ( 權重層路徑與捷徑 ) : $\mathrm{x}_{l+1}=f(\mathrm{y}_l)$。研究團隊試圖研究一個非對稱結構使 activation function $\hat{f}$ 在任何的 $l$ 殘差單元只對權重層路徑有影響 : $\mathrm{x}_{l+1}=\mathrm{x}_l+\mathcal{F}(\hat{f}(\mathrm{x}_l),\mathcal{W}_l)$ 。這樣的設計等價於我們用了一個「預激活」(Pre-activation) $\hat{f}$ 於下一個殘差單元。
<img width=1000 src="https://i.imgur.com/wxH5VAP.png" >
「預激活」(Pre-activation) 與「後激活」 (Post-activation) 的差別在於元素級加法所導致。對一個一般的 $N$ 層神經網路來說，其中 $N-1$ 個 activation function ( $BN$ 與 $ReLU$ ) 是「預激活」或是「後激活」其實並沒有太多的差別。但對於有分支的結構來說，activation 的位置就大有關係。
在此論文中，作者們使用了兩種設計來實驗 : (1) 僅使用 $ReLU$ 作為預激活，以及 (2) 使用 $ReLU$ 與 $BN$ 一起作為預激活項。結果發現前者單獨使用 $ReLU$ 無法享受 $BN$ 帶來的好處，導致結果與原始結構差不多。但後者卻呈現了極佳的表現。
<img width=1000 src="https://i.imgur.com/wCDltJq.png" >

### 分析

作者們發現，預激活的影響是雙重的 : 
1. Ease of optimization
由於 $f$ 是恆等映射，使得訓練變得更佳容易。
2. Reducing overfitting
使用 $BN$ 作為預激活相當於是對模型的正則化[^註2]。

#### Ease of optimization

這樣的影響在訓練 1001 層的 ResNet 更加明顯。當我們使用原始模型設計時，在一開始訓練初期，訓練誤差降低的速度會非常慢。對於 $f=ReLU$ 來說，訊號為負值時會被影響，而且當殘差單元數量很多時，這樣的影響會變得非常明顯，前向或反向傳播式就無法進行很好的估計。

換一個角度來說，若 $f$ 為恆等映射，則訊號可以直接在任一兩個殘差單元間傳遞，即使在 1001 層 ResNet　中訓練誤差可以快速降低。

但作者們也發現，如果　ResNet 並不深，$f=ReLU$ 所造成的影響就不會太嚴重。雖然在訓練初期仍然有一點點影響，但隨即的可以被調整回來。藉由對模型的回應進行監控，作者們觀察到，經過一段時間的訓練後，權重會被調整使 $\mathrm{y}_l$ 恆正。而且 $f$ 不會進行截斷。( 因為 $ReLU$，因此 $\mathrm{x}_l$ 非負。若要使 $\mathrm{y}_l$ 小於0，則 $\mathcal{F}$ 必須要是極負值時才會成立。 )

然而，當層數增加到 1000 時，這樣的截斷效果會變得極為頻繁。


#### Reducing overfitting

另外一個預激活造成的影響就是 regularization。預激活版本的模型雖在 training loss 收斂在較高的地方，但卻可以使 test loss 更低。這樣的現象幾乎在各種架構上均觀察的到。造成這種現象的主因想必就是 $BN$ 的正則化影響。

在原始的殘差單元中，雖然 $BN$ 標準化了訊號，但立刻被加到 shortcut 上面。然而這樣合併後的訊號並沒有被標準化，又被作為下一個殘差單元的輸入。但是預激活的架構可以確保每一個權重層的輸入都是被 $BN$ 標準化過的。


結論 Result
---

( 這部分大概是就上述實驗進行結果闡述，有興趣的就看一下原論文吧~~ )


結論 Conclusions
---

本論文針對深層殘差網路中關於捷徑連結機制做形式化探討。作者們的推導顯示恆等映射的 shortcut connections 以及元素及加法後的 activaiton 對於訊息傳導的平滑性來說是絕對必要的。

一系列的消融實驗支持了我們的推導結果，並且我們可以使 1001層的 ResNet 訓練更加容易且可以得到更好的表現。

後記
---

前一篇論文 " *Deep Residual Learning for Image Recognition* " 可謂經典必讀論文，然而其中許多的細節並沒有在當下有很好的解釋及說明。想要更理解 ResNet 的整個運作過程及原理，本篇論文不可不讀。藉由各種消融實驗可以說明為什麼 ResNet 架構可以有這麼好的表現。


註釋
---

[^註1]: ablation experiments 其實就是藉由控制變量來尋找對結果最有直接影響的變因。當一個結果可能由許多因素所導致，那我們藉由一連串實驗，將這些變因進行刪除、替換等方式，來觀察結果的變化，進而找出最關鍵的變因。香菇討論可以參閱 " *[深度学习里的ablation experiment？](https://www.zhihu.com/question/263837982/answer/273653126)* " 及 " *[消融实验是什么？](https://www.zhihu.com/question/291655038)* "


[^註2]: Batch Normalization 的意義是限制每一層的輸出都要同分布，以符合神經網路的假設，然而這樣的限制在某種層面來說就是一種正則化。相關詳細討論可以參閱 " *[关于batchnormalization和正则化的一些问题？](https://www.zhihu.com/question/288370837)* "