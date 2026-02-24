---
title: "Generative Adversarial Network (9) --- Sequence Generation" 
date: 2019-11-23 02:43:30
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning and having it Deep and Structured
image: https://i.imgur.com/RrdITkw.png
mathjax: true
---

在 GAN 的應用中，前面有提到可以用來做 conditional image generation，這種技術我們可以用來進行圖像風格的轉換、語音上的轉換...等等。

<!-- more -->

當然我們也可以用在 NLP 領域中，我們稱之為 conditional sequence generation，簡單來說，就是一個生成 Sequence 的任務，比較常看見的例子就是語音轉文字、語音翻譯、聊天機器人...等。這樣的輸出就是一串 Word Sequence。

然而，在一般的 Seq2Seq Model 中利用 MLE 來做預測，很容易會碰到一些問題。舉例來說，當一個　Model 接收到 " How are you ? " 這樣的句子時，訓練階段我們認為 " I'm good. " 這樣的回答是好的。訓練結束過後，當我們在進行應用時，遇到相同問題，下列兩個回答，機器人會怎麼做抉擇 ?

1. I'm John.
2. Not Bad.

從人類的角度來看，第二句是好的回答，但 model 卻會認為第一句是應該採取的回應，因為，至少在第一個字上面是正確的。

Supervised Conditional Sequence Generation
---

在上面這種 Supervised Conditional Sequence Generation 任務上，食物通常利用兩種方式來增進 model 的表現 : 

* Reinforcement Learning
* GAN

### Reinforcement Learning

如果我們今天想要引進 Reinforcement Learning 來讓 Chatbot 回答得更好、更準確，那可能會讓機器人不斷的跟人進行對話，最後由人給出一個 Reward，藉由很多次的對話，得到很多的 Reward 來增進機器人的回應。

優化的目標其實就是希望期望 Reward 可以盡可能的高，方式則是使用 Policy Gradient 來進行優化。

如果我們將機器人本身視為一個 Network，當我們輸入一個 $h$，不同參數 $\theta$ 的模型會得到不同的輸出 $x$，當然也就會有不同的 Reward $R(x,h)$，因此我們希望找到一個 $\theta^*$ 可以使整個期望 Reward $\bar{R}$ 最大。( 為了簡化討論，我們讓整個對話過程只有一次問答的過程就會得到 Reward。 )

$$
\theta^*=\arg\max\limits_{\theta}\bar{R}_{\theta}\\
\text{where }\bar{R}_{\theta}=\sum\limits_{h}P(h)\sum\limits_{x}R(x,h)P_{\theta}(x|h)
$$

對於 $\bar{R}_{\theta}$ 我們可以這樣直觀的來解釋 : 

![](https://i.imgur.com/jaO7jb8.jpg)

經由這樣的解釋我們可以改寫 $\bar{R}_{\theta}$

$$
\bar{R}_{\theta}=\mathbb{E}_{h\sim P(h)}\Big[\mathbb{E}_{x\sim P(x|h)}[R(x,h)]\Big]\\
=\mathbb{E}_{h\sim P(h),x\sim P(x|h)}[R(x,h)]
$$

但我們不可能遍歷所有可能的 $h$、$x$，因此我們試圖去利用手上有的資料來做估計 : 

$$
\bar{R}_{\theta}\approx\frac{1}{N}\sum\limits_{i=1}^NR(x^i,h^i)
$$

看似一切合理的推導，但卻發生了一個很嚴重的問題，當我們試著要藉由改變 $\theta$ 來優化 $\bar{R}_{\theta}$ 時，剛剛推導下來的結果根本沒有 $\theta$ 這個參數，或者換個說法，$\theta$ 這個參數被隱藏在這 $N$ 個抽樣資料中 (不同的 $\theta$ 就有不同的抽樣資料 )，導致我們無法對 $\bar{R}_{\theta}$ 直接找梯度。因此有了 Policy Gradient 的方式出來。

既然我們沒辦法對平均 Reward 求梯度，那我們就直接從整個得到 Reward 的「策略」來求梯度。

$$
\bar{R}_{\theta}=\sum\limits_{h}P(h)\sum\limits_{x}R(x,h)P_{\theta}(x|h)\\
\nabla\bar{R}_{\theta}=\sum\limits_{h}P(h)\sum\limits_{x}R(x,h)\nabla P_{\theta}(x|h)\\
=\sum\limits_{h}P(h)\sum\limits_{x}R(x,h)P_{\theta}(x|h)\frac{\nabla P_{\theta}(x|h)}{P_{\theta}(x|h)}\\
=\sum\limits_{h}P(h)\sum\limits_{x}R(x,h)P_{\theta}(x|h)\nabla\log P_{\theta}(x|h)\\
=\mathbb{E}_{h\sim P(h),x\sim P(x|h)}[R(x,h)\nabla\log P_{\theta}(x|h)]
$$

一樣的我們用手中現有的資料來做估計

$$
\nabla\bar{R}_{\theta}\approx\frac{1}{N}\sum\limits_{i=1}^NR(x^i,h^i)\nabla\log P_{\theta}(x|h)
$$

這樣我們就可以使用 Gradient Ascent 來進行優化

$$
\theta^{n+1}\longleftarrow\theta^n+\eta\cdot\nabla\bar{R}_{\theta^n}
$$

在實作上，我們整個過程大概是如下 : 

1. 讓 Machine 跟人進行 $N$ 輪對話 (抽象)，得到 $N$ 筆資料 $(h^i,x^i)$ 跟 Reward $R(x^i,h^i)$。
2. 利用 $\theta^{n+1}\longleftarrow\theta^n+\eta\cdot\nabla\bar{R}_{\theta^n}$ 進行優化，Reward 高的策略出現機率會上調，反之，機率則下降。
3. 重新讓 Machine 根據新的參數跟人進行 $N$ 輪對話 (重新抽樣)。

其中第三點非常重要，一般的 GD 只要更新完參數就可以繼續更新下去，但在 Policy Gradient 中我們必須要重新進行抽樣才能再繼續更新。( 這樣的計算成本十分高昂，在實作上是有方式可以處理這個問題的，就留待 MLDS 的 RL 課程再介紹。 )

以下是 RL 與 MLE 的比較表 : 

![](https://i.imgur.com/LZqZJlz.png)

最後，補充一點的是，現實上要機器跟人類進行上百上千次對話其實也並不是這麼容易的事情，所以也有人利用兩個機器人互相對話來進行 RL。


### GAN

利用 GAN 其實跟 RL 非常相似，甚至我們可以說 RL 的方式就是 GAN 的一種特例。我們可以先從 Conditional GAN 的架構來配合 Seq2Seq 來看

![](https://i.imgur.com/lnACrB4.png)

我們輸入一個 Sentence $c$ 利用一個 $G$ 來生成另一個 Sentence $x$，將 $c,x$ 輸入 $D$ 進行真偽判斷。這不是太困難，但之所以稱 RL 可以是 GAN 的一個特例主要是因為，如果把 $G$ 看作是一個 ChatBot，而 $D$ 的工作讓人來做，這不就是 RL 在做的事情 ?

* 初始化 $$G$ (Chatbot) and $D$ 
* 迭代進行 : 
    * 從訓練資料中隨機抽樣句子與回應 $(c,x)$
    * 從訓練資料中隨機抽樣句子 $c'$，並且利用 $G$ 生成 $\tilde{x}\Longrightarrow\tilde{x}=G(c')$
    * 藉由更新 $D$ 來增加 $D(c,x)$ ，並減少 $D(c',\tilde{x})$ 
    * 藉由更新 $G$ 來減少 $D(c,x)$ ，並增加 $D(c',\tilde{x})$


然而，Sentence 上的訓練，並非直接輸入一個 Sentence 向量，$G$ 是一個 Seq2Seq Model，可能使用的是一個類似 RNN 的方式在運作，每一次吐出一個 token[^註1]。但是 RNN 在運作的過程中，其實是藉由抽樣 (sample) 來得到 token。

![](https://i.imgur.com/y7hHuFT.jpg)

現在遇到的困難是，$G$ 利用這樣抽樣的過程得到 token，當我們要進行 $G$ 的優化時，這個過程是沒有辦法微分的。[^註2]

無法微分就不能進行 GD 優化，但在許多 paper 中有許多種不同的方式來解決這樣的問題 : 

1. Gumbel-Softmax
2. Continuous input for $D$
3. Reinforcement Learning

#### Gumbel-Softmax

課程中沒有對 Gumbel-Softmax 有比較深入的介紹，簡單來說這種方法是利用一個特殊的 Softmax 來對離散的分佈進行抽樣，這樣的抽樣是連續的、可微分的，而且逼近原本的離散分佈。

#### Continuous input for $D$

這個方法比較直觀，原本我們說 RNN 利用取樣來輸出 token，而這導致了佈可微分的狀況，既然如此，那我們就不要取樣了，直接把取樣前的分佈輸入 $D$。

![](https://i.imgur.com/S9q3lVW.png)

這樣的做法固然解決了不可微分的狀況，但也產生了訓練時的問題。

試想，我們在 training data 中的真實資料通常都是以 one-hot encoding 呈現，但我們生成的 word distribution 卻都是依些連續數值。這樣在訓練的過程中，$G$ 為了騙過 $D$ 就會盡可能地生成出類似 one-hot 型態的分佈，但其實這樣的分佈產生的 token 其實很可能是毫無語意可言的。

在實作中藥解決這樣的問題，比較可能成功的方式就是利用 WGAN。原因是因為 WGAN 中有一個 1-Lipschitz 的限制，這樣的限制導致 $D$ 在進行真偽判定的能力不會這麼「強」。李宏毅課程中的舉例更生動，其實就是 $D$ 看東西的能力變得模糊，這樣$G$就不見得非得將分佈壓成 $0,1$。


#### Reinforcement Learning

第三個方法，我們利用 RL 訓練的方式來進行，可以解決這個不可微分的狀況。

作法很簡單，其實就把 RL 中的人改成 $D$，把 Reward $R(c^i,x^i)$ 改成 $D(c^i,x^i)$，然後訓練方法跟上面介紹的 RL 訓練過程一模一樣即可。

但是上面的步驟只有 train $G$ ，但現在我們還必須要 train $D$，方式跟一般的 Discriminator 訓練其實一樣，我們用真人對話的資料跟生成對話的資料輸入給 $D$，讓其盡可能地分辨出真偽。最後再反覆地對 $G$ 及 $D$ 進行訓練即可。 

現在假設我們輸入一個 sentence $c^i=$" What's your name ? "，而我們得到一個生成回應 $x^i=$ " I don't know "

這樣的回應並不是很好，因此我們會希望得到的 Reward 是負值，也會希望$x^i$ 的機率 $P(x^i|c^i)$ 下降。但我們知道整個 $x^i$ 的生成是利用 RNN 一個一個 token 生成出來的。在這個例子中，可以切出三個 token : " I ", "'don't","know"，而且 $x^i$ 的發生機率應該是這三者的個別發生機率之乘積，而當我們希望 $x^i$ 的機率 $P(x^i|c^i)$ 下降 也就是希望上面這三個 token 的發生機率均下降。

$$
\nabla\bar{R}_{\theta}\approx\frac{1}{N}\sum\limits_{i=1}^{N}D(x^i,c^i)\nabla\log P_{\theta}(x^i|c^i)
$$

![](https://i.imgur.com/euBGH4d.png)

但是 "I" 這個 token 其實並沒有這麼差，如果要將其產生機率下降其實有些不合理。但在「理想狀況」下，這樣的狀況其實並沒有什麼問題，因為 " I don't know " 是抽樣出來的結果，我們也可能抽樣到 $x^i=$ " I am John ", "I am Mary" 之類的 Sentence，這些回應句都該得到正值的 Reward，而且希望 $P(x^i|c^i)$ 上升，當然這也代表了 "I" 出現的機率在這樣的前提下應該要被提高。

於是在整個抽樣下，只要我們抽樣得夠多，"I" 這個 token 一加一減後其實會抵消，不會造成負面的影響。但這前提就是 「抽樣得夠多」，我們往往會知道這種理想狀態通常不會發生。

所以我們會希望當 $P(x^i|c^i)$ 上升或下降時，每一個 token 的機率可以個別進行估計，不要一起上升或下降。

底下是採用的修改方式 ( 課程中簡單帶過，我們也簡單看過就好 )

![](https://i.imgur.com/hF2pffJ.png)


### 實驗的延伸討論

當我們進行 GAN 當我們進行 GAN 或是最早之前的 MLE 方式進行，其實在實驗結果的呈現上有蠻大程度的不同。

![](https://i.imgur.com/f8kOFAn.png)

從上圖來看，MLE 會生成的語句通常都是比較短的、模稜兩可的句子。而 GAN 則是會比較偏向於生成較長也相對複雜的句子。

Unsupervised Conditional Sequence Generation
---

這種 Unsupercvised Learning 在 GAN 前面的部分講過很多了，例如畫風轉換、語音轉換...等等。所採用的方法以 Cycle GAN 為大宗，其他還有 X-GAN、Star GAN 等方法。在 Seq2Seq Model 中，也有類似的 Unsupervised 問題，最基本的就是 Text Style Transfer。

### Text Style Transfer

簡單一點的例子就是將正面文字 ( it's a good day ) 轉換為負面文字 ( it's a bad day )。在實作上我們就是把正面跟負面視為兩個不同的 domian 利用 Cycle GAN 等方法進行生成。

當然一樣會遇到上面離散分布導致不可微分的狀況，處理的辦法也就如同上述介紹，這邊不多贅述。

### Abstractive Summarization

另一個廣泛的問題是摘要擷取，也就是給定一篇文章，我們希望機器可以自行學出摘要。這個問題我們當然可以利用 Supervised Learning 的方式進行處理，先對一堆文章進行人工摘要標註，再來訓練 Model，但這樣的方式一來曠日廢時，二來我們要進行自動摘要擷取的文章通常也都還是機器沒看過的，要進行摘要擷取會變得相對困難。

其實擷取摘要並非新的問題，早在幾十年前就有人在研究，當時作的主要是  Extracted Summarization，這是指將文章中的一些重要文句抓出來拼接成一個摘要。但這樣處理通常會造成語句不通順，而且這樣的摘要也是我們真正想要的。最終我們希望的是機器可以利用自己的學習能力來自己生成出摘要，不是只是照抄而已，這就是我們說的 Abstractive Summarization。

在實務上的做法一樣是將文章跟摘要視為兩個不同的 Domain，而我們的任務就是訓練一個 model 可以在這兩個 domain 間互轉。

![](https://i.imgur.com/WqoC3Eo.png)

跟 Cycle GAN 一樣，我們必須在中間加上一個 Discriminator 來使生成的摘要更真實。

下圖可以明顯看出，僅使用了 500k 筆資料進行的 Unsupervised Learning，其準確度進逼使用 3.8M 筆資料進行 Supervised Learning。也就是說，用了相較少量的資料我就可以做出幾乎相同的成果。

![](https://i.imgur.com/oxZdYWX.png)

### Unsupervised Machine Translation

課程這部分沒講什麼，有興趣的可以看看課程中給的兩篇 Paper : " *[Word Translation Without Parallel Data](https://arxiv.org/abs/1710.04087)* " 以及  " *[Unsupervised Machine Translation Using Monolingual Corpora Only](https://arxiv.org/abs/1711.00043)* "

比較有趣的還是實驗結果，在下途中可以看到，如果我們手中的資料十萬筆，利用 Unsupervised 的方式是可能會勝過 Supervised 的。

![](https://i.imgur.com/rL2as6D.png)


### Unsupervised Speech Recognition

將語音跟文字一樣視為兩個不同 Domain，利用 Cycle GAN 技術來讓機器學習到怎麼將語音轉文字，甚至機器可以學習怎樣的語音訊號會對應到什麼樣的文字。

![](https://i.imgur.com/CpJ0uVJ.png)

以下是實驗結果

![](https://i.imgur.com/byVjRGy.png)

語音使用 TIMIT Corpus ，而文字使用的是 WMT Corpus，這兩個語料庫間並沒有對應關係。從上面的實驗解果看來，我們可能會覺得 Unsupervised 的效果不如預期，但事實上這樣的表現遠遠勝過隨機亂猜，這表明了機器仍有學習到一些東西。



註釋
---

[^註1]:
Token 在 NLP 領域中指的是句子經過切詞後產生的不同長度的字詞，不一定是"字"(word)，而每一個 token 我們都會以一個向量來表示。  


[^註2]: 
我們可以從一個比較直觀的角度來看這個取樣的過程，取樣其實不會是一個平滑的過程，既然不是平滑的，那我們就無法對這樣的函數($G$)進行微分。