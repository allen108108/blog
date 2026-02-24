---
title: "Chat GPT 實務應用上的挑戰"
date: 2023-04-30 18:21:59
categories:
- 實作 Implementation
image: https://joshbersin.com/wp-content/uploads/2023/01/openai2.png
mathjax: true
---

前言
---
三年前，剛開始轉職找工作的時候就開始在玩聊天機器人，那時候我一直希望未來可以有一個仿真的聊天機器人技術，這會讓整個真實世界出現極大的轉變。

三年後的現在，Open AI 在生成模型的耕耘上有了極大的收穫，也就是 Chat GPT 的問世。去年底開始爆紅後，這半年內 Chat GPT 在 AI 的領域中熱度一點都沒有削減，甚至平常沒有涉獵 AI 領域的親朋好友也都在問我對它的意見。一直都在玩 Line Bot 的我當然也不能錯過的嘗試了進行這樣的運用。不過，好玩歸好玩，在實際要執行落地、要導入商品的時候，還是會發現一些目前的瓶頸，也是衍生出這篇文章的起點。

<!-- more -->

Chat GPT 的前世今生
---

講到 Chat GPT ，就不能不瞭解到底什麼是 GPT。GPT 是一個語言生成模型，初代 GPT-1 出現在 Open AI 發表的論文 "[Improving Language Understanding by Generative Pre-Training](https://cdn.openai.com/research-covers/language-unsupervised/language_understanding_paper.pdf)" 之中。GPT 這個名稱就是論文標題 "Generative Pre-Training" 的縮寫，藉此我們也可以清楚，GPT 其實就是一個利用「生成式預訓練」而產生的模型。

NLP ( Natural Language Processing，自然語言處理 ) 領域在前幾年基本上都專注在語言的切詞、詞性分析、....等基本的任務，這也因此讓同期的 BERT 更引人注目。坦白說，前幾年我也曾經以為最後 BERT 會稱霸現在的 NLP 領域。BERT 是 Google 發表的模型，與 GPT 相同，也是預訓練的語言模型，但它更著重在雙向編碼上。

![](https://i.imgur.com/G6tdTEP.png)
( 圖片來源 : "[BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding](https://arxiv.org/abs/1810.04805)" 論文 )

BERT 與 GPT 都是基於 Transformer 結構下的預訓練模型，但 BERT 捨棄的生成的能力而換取雙向編碼結構來讓模型更符合「當時」NLP 領域更注重的任務。但 Open AI 對於語言「生成」的堅持，讓 GPT 從初代一路轉變到 GPT-3.5 甚至是現在的 GPT-4。(前陣子論壇內也流傳了 GPT-5 即將問世，但印象中 Open AI 有出來闢謠 ?) 也是這份堅持讓我們能看到 Chat GPT 現在的成功。

而今，我們看到的 Chat GPT 主要是以 GPT-3.5 模型為基礎，GPT-3 到 GPT-3.5 也經過了一番波折，這中間的技術脈絡如果大家有興趣，建議可以看這篇文章 : "[ChatGPT的各项超能力从哪儿来？万字拆解追溯技术路线图来了！](https://mp.weixin.qq.com/s?__biz=MzA3MzI4MjgzMw==&mid=2650864144&idx=4&sn=1270624988d70f44d4059af7ac4ae4e0&chksm=84e53e6eb392b7785418e8257952284cfe6dd801d84404958fb917c461da792039626e172c31&scene=21#wechat_redirect)"，這裡面可以看到整個技術跟模型的遞移過程，也可以看到程式碼生成的 CodeX 在整個過程中的位置。

![](https://i.imgur.com/Q1pSOmy.png)
(圖片來源 : "[ChatGPT的各项超能力从哪儿来？万字拆解追溯技术路线图来了！](https://mp.weixin.qq.com/s?__biz=MzA3MzI4MjgzMw==&mid=2650864144&idx=4&sn=1270624988d70f44d4059af7ac4ae4e0&chksm=84e53e6eb392b7785418e8257952284cfe6dd801d84404958fb917c461da792039626e172c31&scene=21#wechat_redirect)")

想要使用 Chat GPT，現在主要有兩種方式，一種是直接跟 Chat GPT 進行對話，另外，Open AI 也有提供 API 可以讓使用者可以將 Chat GPT 串接到我們自己的服務上面，不過這兩種本質上還是有一些差異點。

與 Chat GPT 對談
---

我們可以直接到 Chat GPT 的網站 ( https://chat.openai.com/ )，直接與 Chat GPT 對話。

![](https://i.imgur.com/lf5lpdE.png)

這樣的使用方式，其實是把我們當作是一個接受 Chat GPT 服務的角色，在這樣的使用中，我們可以發現 Chat GPT 是可以理解前後文後，進行對話。

![](https://i.imgur.com/Mvmlr6C.png)

上面的簡單範例可以看的出來，每一句對話 Chat GPT 都有理解到 「我不想吃火鍋」這樣的概念，不斷的進行回答。

當然，我們也可以根據我們的需求請它寫出一段程式碼。

![](https://i.imgur.com/ycIkBOc.png)

API 串接
---

當我們利用網站去使用 Chat GPT，這其實是 Open AI 提供給我們一個多功能的整合平台，在這個平台中，我們可以去對話、寫程式碼甚至是生成圖片。然而，要利用 Open AI API 時必須了解到，我們是提供 Chat GPT 服務的角色，網站中我們使用到的這些服務，其實是由多種 API 功能組合而成的，必須了解 Open AI 提供之 API 特性組合後應用到我們的應用場景中。


從 Open AI 提供的[ API 文件](https://platform.openai.com/docs/api-reference/introduction)中可以看到，Open AI 依照功能的不同提供給我們幾種 API 使用 : 

1. [Chat](https://platform.openai.com/docs/api-reference/chat) : 將訊息利用列表的方式給 Chat GPT 進行回覆。
2. [Edits](https://platform.openai.com/docs/api-reference/edits) : 將文字及指示包成一個物件，Chat GPT 會根據指示對文字進行編輯。
3. [images](https://platform.openai.com/docs/api-reference/images) : 可利用文字生成圖片。
4. [Embeddings](https://platform.openai.com/docs/api-reference/embeddings) : 將輸入文字進行 embedding，可應用於機器學習模型中。
5. [Audio](https://platform.openai.com/docs/api-reference/audio) : 將聲音轉變成文字。

除此之外，現在使用這些 API 可是必須收費才能使用，根據功能的不同，以 1K token 為計費單位，轉換成中文字約 400 個中文字。

![](https://i.imgur.com/hIDDt2O.png)

從上圖可以知道，如果使用 Chat API，每使用 400 字中文，就要花掉 0.002 美金。

#### **Open AI API 註冊**

這邊，筆者僅對 Chat API 進行簡要的說明，畢竟這是絕大多數人會使用到的模式。要使用 Open AI API 之前，我們的環境必須先安裝 `openai` 套件。

```python
$pip install openai
```

除此之外，使用者也必須於 [Open AI](https://openai.com/product)進行帳號註冊，註冊完成後可以到右上角點擊使用者頭像後選擇 `View API Key` 

![](https://i.imgur.com/cQ9NjT1.png)

下方的 `Create new secret key` 便可以創建新的 API key。**這邊要注意的地方是，創建 API Key 完成後，會出現 Secret Key，但只會出現這一次，未來將不能再顯示完整的 Secret Key。**未來若遺忘，便只能重新建立一個。

#### **Chat Completion 使用**

API 本身是使用 Http Post Method 進行呼叫，因此要使用 Python 呼叫 API 我們也需要使用到 `requests` 模組。

```python=
import openai
import requests
```

我們必須先載入我們的 API Key，才能開始使用 API。

```python=3
openai.api_key ="你自己的 Secret Key"
openai.Model.list()
```

`openai.Model.list()` 可以列出目前我們可以使用的模型有哪些，如果以一般的聊天場景來說，目前最新的模型通常是 `gpt-3.5-turbo` 或是 `gpt-3.5-turbo-0301`，這兩者在使用上的細節主要在於前者需要主定具體的角色跟問題內容，但後者比較著重問題內容本身，另外，後者有效期限只會到 2023/06/01，而前者仍會持續更新。

模型確定後，我們便可以開始使用 API 來創建聊天情境並藉由輸入來取得 Chat GPT 的回覆。

```python=
completion = openai.ChatCompletion.create(
  model="gpt-3.5-turbo",
  messages=[
        {"role": "system", "content": "向模型提供初始指令、場景需求"},
        {"role": "assistant", "content": "機器人所回覆之訊息"},
        {"role": "user", "content": "用戶所提出之問題"}
    ]
)
```

從上面的程式碼，我們可以瞭解到，目前 Chat GPT Chat Completion 提供了三個角色來建構一個對話場景，每一個角色 (role) 及訊息內容 (content) 包成一個物件，在陣列中依序排列，形成一個對話陣列 : 

* 系統角色 (system) : 系統角色並非必須存在的角色，但這個角色的介入，有助於讓整個模型了解使用者需求以作出適合且貼切場景的回覆。
* 助手角色 (assistant) : 助手角色其實就是對話中機器人的角色，藉由系統角色的指令及使用者的提問來作出回覆。
* 使用者角色 (user) : 使用者提出的問題內容。


現在假設一個場景，我希望 Chat Bot 可以將使用者輸入之文字直接翻譯成日文，而使用者輸入的文字是 「今天天氣如何 ?」

```python=
completion = openai.ChatCompletion.create(
  model="gpt-3.5-turbo",
  messages=[
        {"role": "system", "content": "assistant 是一個翻譯，會將 user 輸入的文字直接翻譯成日文。"},
        {"role": "user", "content": "今天天氣如何 ?"}
    ]
)
```

接著我們便可以提取 Chat GPT 的回覆如下 :

```python=
print(completion.choices[0].message.content) #今日の天気はどうですか？
```


#### **Chat Completion 實現前後文理解**

仔細思考一下我們的使用場景，現在我們可能有一個聊天機器人，當使用者輸入一段文字後，我們便會串連 `openai.ChatCompletion.create` 來創建一個聊天過程，說到這邊，你是否發現了什麼 ?

沒錯，當我們串接 API 時，每一句話都會創建一個新的 Chat Completion，根本不可能實現前後文理解。在實務上，要進行前後文理解必須要將所有對話過程，遞迴地持續增加到下一輪的對話中。在網頁版本的 Chat GPT 中，Open AI 幫我們解決了這樣的問題，而串接 API 我們必須自己進行這個遞迴過程。


```python=
chat_list = []

def ask(prompt):
    chat_list.append({"role":"user","content":prompt})
    response = openai.ChatCompletion.create(model="gpt-3.5-turbo",messages=chat_list)
    answer = response.choices[0].message['content']
    #當使用者提出一個問題後，我們可以得到一個回覆
    
    chat_list.append({"role":"assistant","content":answer})
    #將這個回覆也加入到對話列表中，作為下一輪的輸入
```

這樣的遞迴過程便可以打造一個理解前後文的聊天機器人，但......每一輪的輸入都要將前文所有對話納入，想當然耳，會耗掉不少 token 數，而且隨著對話次數越來越多，這樣的對話成本會急速上升。


Chat GPT 落地應用的難點
---
#### **1. 無法取得最新資訊**

Chat GPT 目前的運作，並非利用使用者的問題上網去撈取最新的資料來進行回覆，而是當初模型本身 1750億個參數所儲存生成語言回覆給使用者，因此 Chat GPT 並沒有辦法回覆使用者 「即時」的資料。

舉例來說，筆者利用 API 串接到 Line Bot 後，詢問它目前上檔的電影有哪些，得到的回覆竟然是「黑寡婦」、「永恆族」等，這大概是兩年前的資訊了，因此，Chat GPT 的導入應避免使用在需要最新資訊的場景中。


#### **2. 成本高昂**

從上面的使用方式我們可以發現，如果今天要在我們的服務上導入 Chat GPT 上，想要使其具有理解前後文的能力，讓使用者可以體驗到更仿真的對話，那我們必須要不斷的將前面的對話都作為下一輪訊息輸入到 Chat GPT 中。

這表示，若我們希望將服務商品化，顧客的成長也意味著對化成本的急速成長，我們應該利用什麼樣的 Business Model 來支撐這樣的服務成本 ? 這將會是未來商品化要嘗試解決的痛點。

#### **3. 無法控制**

若要將 Chat GPT 導入到服務之中，並且要將其商品化，我們怎麼樣避免 Chat GPT 的回覆可能會損及公司及商品商譽的風險 ? AI 模型本身就是一個黑盒子，我們本來就無法限制模型本身的決策，但對商品本身而言，這將會是一個無法控制的風險成本。

簡單來說，若有使用者向 Chat GPT 詢問我們服務的優缺點，或是，有使用者請 Chat GPT 比較我們與其他競爭廠商的優劣時，我們應該怎麼限制它的回答 ?

當然，我們可以用 API 中的 system 來給予它系統性的限制，但從實務來看，使用者是有可能利用一些特別的 prompt 來避開這些限制，這些都會是商品化後的風險成本。


後記
---

Chat GPT 以及 Midjourney 在這半年中的確改變了整個世界，這樣的風潮席捲了不只是 AI 的領域，也的確讓很多的企業、新創公司看到了一線商機。但這股商機的背後，也代表的是需要思考一種創新的商業模式來作支持，在一股腦投入資源的同時，我們應該先了解這些新技術背後的成本跟隱藏的風險，才能實實在在的為我們的服務提供一種新的營利模式。

