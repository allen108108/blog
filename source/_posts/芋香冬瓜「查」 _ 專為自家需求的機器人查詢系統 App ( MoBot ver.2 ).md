---
title: "芋香冬瓜「查」 : 專為自家需求的機器人查詢系統 App ( MoBot ver.2 ) "
date: 2020-06-26 05:59:50
categories:
- 專案 Project
- Line Bot
image: https://i.imgur.com/TdNr0Jn.jpg
mathjax: true
---


系列文章
---

* [MoBot : LINE Bot 開發紀錄 ( LINE / Heroku )](https://allen108108.github.io/blog/2020/03/26/MoBot%20_%20LINE%20Bot%20%E9%96%8B%E7%99%BC%E7%B4%80%E9%8C%84%20%28%20LINE%20_%20Heroku%20%29)
* [Line Bot 在群組中的喚醒/休眠機制 (Django Database 之應用)](https://allen108108.github.io/blog/2021/08/30/Line%20Bot%20%E5%9C%A8%E7%BE%A4%E7%B5%84%E4%B8%AD%E7%9A%84%E5%96%9A%E9%86%92_%E4%BC%91%E7%9C%A0%E6%A9%9F%E5%88%B6%20(Django%20Database%20%E4%B9%8B%E6%87%89%E7%94%A8)/#more)
* [利用 Line Notify 於 Line Bot ( Django + Heroku ) 進行群組推播提醒](https://allen108108.github.io/blog/2022/03/01/%E5%88%A9%E7%94%A8%20Line%20Notify%20%E6%96%BC%20Line%20Bot%20(%20Django%20+%20Heroku%20)%20%E9%80%B2%E8%A1%8C%E7%BE%A4%E7%B5%84%E6%8E%A8%E6%92%AD%E6%8F%90%E9%86%92/)
* [Line Bot 在群組中監控成員的加入與退出](https://allen108108.github.io/blog/2022/03/02/Line%20Bot%20%E5%9C%A8%E7%BE%A4%E7%B5%84%E4%B8%AD%E7%9B%A3%E6%8E%A7%E6%88%90%E5%93%A1%E7%9A%84%E5%8A%A0%E5%85%A5%E8%88%87%E9%80%80%E5%87%BA/)

<!-- more -->

前言
---

前一篇 "*[MoBot : LINE Bot 開發紀錄 ( LINE / Heroku )](https://allen108108.github.io/blog/2020/03/26/MoBot%20_%20LINE%20Bot%20%E9%96%8B%E7%99%BC%E7%B4%80%E9%8C%84%20%28%20LINE%20_%20Heroku%20%29)*" 主要介紹了整個 Line Bot 的整體細節，利用 Django 做為主架構搭配 heroku 伺服器來實現一個簡單的聊天機器人。當時主要是配合參考書目以及網路教學，將自己想做的幾個功能拼拼湊湊的組合在一起。

時隔幾個月之後，有些細節部分需要進行處理 (例如 API 版本更新及網站更新導致爬取資料的失敗)，也正好趁這個機會將整個程式碼整理一番。

此了內在架構做改善外，整個 Line Bot 的外觀也做了一下統一的設計，利用家裡面兩個小孩為主軸來設計，而名稱也從 Mobot 老摩二號改為 -- 芋香冬瓜「查」。 ( 小孩的外號分別為瓜瓜跟芋頭 )

舊版 MoBot 的缺陷
---

*整個後端架構此文不另贅述，整個 Django 結構、細節參數的設置以及 Heroku 上的部屬都請參照[前一篇文章](https://allen108108.github.io/blog/2020/03/26/MoBot%20%20LINE%20Bot%20%E9%96%8B%E7%99%BC%E7%B4%80%E9%8C%84%20%28%20LINE%20%20Heroku%20%29)，在這一個大更新中，筆者並沒有對這個部分進行修改。*

前一篇文章中，整體的功能設定如下圖，當初的 MoBot 採取的是一個實驗性質的聊天機器人，所以試圖想要用不同的方式來達成不同的目的 ( 例如利用 LUIS 來判斷意圖查詢天氣及電影資訊而利用一般邏輯判斷句來判斷是否要查詢星座運勢) ，最後就會顯得有點混亂，這也是這次改版的最主要原因之一。

![](https://i.imgur.com/hBT6sK8.png)

在這次的改版中，主要功能延續 MoBot，主要有 : 天氣查詢、電影資訊查詢以及星座運勢查詢，這三個功能都是家人會時常需要查詢的，因此仍然將這些功能視為 **芋香冬瓜「查」** 聊天機器人的主軸。

MoBot 之前的設計並沒有考慮的很周全，而且筆者也指示抱持著玩票的心態，因此在使用上並沒有太考量使用者的經驗，因此出現以下問題 : 

* 使用者一開始不知道怎麼使用，通常會隨意輸入一些招呼詞 ( 如 :　你好啊、 Hi ... 等 )，但 MoBot 並沒有做這樣的意圖判斷，因此可能導致機器人無法回應。
* 想要了解 MoBot 使用方法，必須輸入 `@使用方法` ，這種方式太不直覺，一般使用者不太會做這樣的操作。
* 針對特定縣市查詢氣象、針對特定電影查詢場次若出現問題，聊天機器人應該仍要給予使用者一些回饋，並引導其正確的輸入查詢。

而這次的 **芋香冬瓜「查」** 在修改中也盡可能的處理上述的問題。


功能更新
---

**芋香冬瓜「查」** ver.1 針對 MoBot 進行了下列的更新 :

* 將機器人名稱由老摩二號改為 **芋香冬瓜「查」**
* 添加圖文選單，方便使用者查詢
* 【Update】 升級 LUIS 版本
* 【Fix】修正氣象查詢 Bug
* 【Fix】修正星座運勢查詢 Bug
* 【Fix】將所有使用者輸入統一由 LUIS 進行意圖判別
* 【Fix】捨棄 YouTube Python API，直接利用 YouTube Data API v3 爬取影片資訊 JSON 檔案
* 【Fix】針對各種可能的 Error 狀況，給予適當的回覆，盡可能地避免 **芋香冬瓜「查」** 出現沒有回覆使用者的狀況。
* 【New Feature】 針對使用者與聊天機器人的招呼語，添加意圖判斷，使其導入使用說明
* 【New Feature】 添加一新意圖判斷，判斷使用者心情低落時可以回覆可愛照片。


更新細節
---

### 星座運勢查詢功能修正

在使用上，此功能最早出現問題，原先使用 Beautifulsoup 針對[*唐綺陽星座運勢週報播放列表*](https://www.youtube.com/playlist?list=PL6Kny3AiOhiD6SSMzEyiyEVvHHQp0MW3J)直接找出需要的影片網址後，再經由 YouTube Python API 爬取影片詳細內容。

但最近開始 Beautifulsoup 會出現時有時無撈不到影片的詭異狀況。最後統整了一下做法，不拆兩段方法來擷取網址跟影片資訊，直接利用 API 一次爬取。

```python=
addressUrl = f"https://www.googleapis.com/youtube/v3/playlistItems?part=snippet,contentDetails,status&playlistId={playlist_id}&key={DEVELOPER_KEY}&maxResults=20" 

addressUrlQuote = urllib.parse.quote(addressUrl, safe=string.printable)
response = urlopen(addressUrlQuote).read().decode('utf-8')
responseJson = json.loads(response)
```

這樣的方式我們可以得到播放清單中每個影片的資訊，下面是其中一部影片的資訊 : 

```jsonld=
{'kind': 'youtube#playlistItem',
   'etag': 'vnFfx9CtaSL-F-g4g3qWSe61168',
   'id': 'UEw2S255M0FpT2hpRDZTU016RXlpeUVWdkhIUXAwTVczSi44OUE4RUIwOURGRUM0MDdG',
   'snippet': {'publishedAt': '2020-06-21T01:30:08Z',
    'channelId': 'UCK7LdglLCApOTaylxX8hW2Q',
    'title': '6/22-6/28｜星座運勢週報｜唐綺陽',
    'description': '◎工作注意：\n雙子：長程計畫，能心想事成\n射手：點子很靈，可盡情暢想\n魔羯：年輕人可帶來刺激或改變\n雙魚：收斂一下，不要心急改變\n\n◎桃花注意：\n金牛：要不要桃花是自己的課題\n處女：能受到安慰與呵護\n天秤：感情甜蜜，約會很愉快\n水瓶：有狀況，把話聊開也不錯\n\n◎財運注意：\n巨蟹：需要資源，喊一聲就來\n天蠍：試試手氣，有中獎好運\n\n◎健康注意：\n白羊：緊繃忙碌，注意腸胃健康\n獅子：別心不在焉，以免意外連連',
    'thumbnails': {'default': {'url': 'https://i.ytimg.com/vi/hE55KGVN-xI/default.jpg',
      'width': 120,
      'height': 90},
     'medium': {'url': 'https://i.ytimg.com/vi/hE55KGVN-xI/mqdefault.jpg',
      'width': 320,
      'height': 180},
     'high': {'url': 'https://i.ytimg.com/vi/hE55KGVN-xI/hqdefault.jpg',
      'width': 480,
      'height': 360},
     'standard': {'url': 'https://i.ytimg.com/vi/hE55KGVN-xI/sddefault.jpg',
      'width': 640,
      'height': 480},
     'maxres': {'url': 'https://i.ytimg.com/vi/hE55KGVN-xI/maxresdefault.jpg',
      'width': 1280,
      'height': 720}},
    'channelTitle': '唐綺陽官方專屬頻道',
    'playlistId': 'PL6Kny3AiOhiD6SSMzEyiyEVvHHQp0MW3J',
    'position': 1,
    'resourceId': {'kind': 'youtube#video', 'videoId': 'hE55KGVN-xI'}},
   'contentDetails': {'videoId': 'hE55KGVN-xI',
    'videoPublishedAt': '2020-06-21T01:30:05Z'},
   'status': {'privacyStatus': 'public'}},
```

這樣一來我們可以從單一次的爬取中可以同時由 `'description'` 得到影片說明，且從 `'contentDetails'` 的 `'videoId'` 可以得到網址資訊。


![](https://i.imgur.com/EF7PNYd.png)



### 電影場次查詢功能修正

當 LUIS 判斷使用者意圖是進行電影資訊的檢索時，同時會針對電影名稱做命名實體識別 (NER, Named Entity Recognition)，因此正常在進行特定電影的場次檢所需求送出時，LUIS 可以利用識別到的電影名稱來向開眼電影網站進行爬取。

但 MoBot 並沒有處理識別不到電影名稱的能力，也就是說當使用者沒有或是錯誤輸入電影名稱， MoBot 會沒有任何反應。

在 **芋香冬瓜「查」** 中，筆者給定 $0.8$ 為意圖的機率閾值，也就是說，當判定為電影檢索意圖 (在筆者的 LUIS 中此意圖命名為 'movieQuery')，且機率大於 $0.8$ 時，便會進行電影的檢索。

```python
(result['prediction']['topIntent'] == 'movieQuery') & (result['prediction']['intents']['movieQuery']['score'] > 0.8)
```

但當 LUIS 識別不到電影名稱時，便會改爬取當週新片回覆給使用者，並且提醒使用者可以怎麼檢索特定電影。

```python=
at_link = 'http://www.atmovies.com.tw/movie/new/'
res = requests.get(at_link)
res.encoding = 'utf-8'
soup = BeautifulSoup(res.text, 'lxml')
a_tags = soup.find_all('a')

new_movies = []

for tag in a_tags : 
    #print(tag.get('href'))
    if ('/movie/' in tag.get('href')) and (len(tag.get('href')) == 20) and (tag.text != ""):
        new_movies.append(tag.text)
text = '想不出來要看什麼嗎 ? \n沒關係，臭瓜跟芋頭來推薦推薦本周新片 : \n'
for i, movie in enumerate(new_movies):
    show = f'{i+1}. {movie.split(" ")[0]}' + '\n'
    text += show
line_bot_api.reply_message(event.reply_token, 
                            [TextSendMessage(text=text), TextSendMessage(text='您也可以加入電影名稱查詢唷!\n例如 : XXX電影資訊')]) 
```
在這樣的狀況下，不管使用者有沒有輸入電影名稱， **芋香冬瓜「查」** 都能給予適當回覆。

![](https://i.imgur.com/i9XIkXg.png)

 
### LUIS 版本修正 & 新意圖添加

在修正的過程中，發現因為 LUIS 版本的更迭，導致獲得的 JSON 檔格式不太一樣，這也影響了筆者由 LUIS 所取得的意圖資訊，大多數時間在 Heroku 後台都會發現我們得不到意圖的資訊。

下面是新版本的 LUIS 預測結果 : 

```jsonld=
{'query': '台北天氣',
 'prediction': {'topIntent': 'weatherQuery',
  'intents': {'weatherQuery': {'score': 0.9923128},
   'badEmotion': {'score': 0.0107163871},
   'None': {'score': 0.00253541139},
   'welcome': {'score': 0.00244946033},
   'ConstellationsQuery': {'score': 0.00190337654},
   'movieQuery': {'score': 0.00117647834}},
  'entities': {'location': ['台北'],
   '$instance': {'location': [{'type': 'location',
      'text': '台北',
      'startIndex': 0,
      'length': 2,
      'score': 0.9983831,
      'modelTypeId': 1,
      'modelType': 'Entity Extractor',
      'recognitionSources': ['model']}]}}}}
```

從這樣的格式可以發現，使用 MoBot 所定義擷取 LUIS 結果的函式可能會有問題，因此在 **芋香冬瓜「查」** 裡面我們必須重新定義`sendLUIS()`函式。


```python=
def sendLUIS(event, mtext):
    try:
        if (result['prediction']['topIntent'] == 'welcome') :
            .
            ..
            ...
        elif (result['prediction']['topIntent'] == 'weatherQuery') & (result['prediction']['intents']['weatherQuery']['score'] > 0.8) :
            .
            ..
            ...
        elif (result['prediction']['topIntent'] == 'movieQuery') & (result['prediction']['intents']['movieQuery']['score'] > 0.8)  : 
            .
            ..
            ...
        elif (result['prediction']['topIntent'] == 'ConstellationsQuery') & (result['prediction']['intents']['ConstellationsQuery']['score'] > 0.8)  : 
            .
            ..
            ...
        elif (result['prediction']['topIntent'] == 'badEmotion') & (result['prediction']['intents']['badEmotion']['score'] > 0.8)  : 
            .
            ..
            ...
    except Exception as e:
        import traceback
        traceback.print_exc()
        line_bot_api.reply_message(event.reply_token, TextSendMessage(text='查詢時產生錯誤，請稍後再查詢唷~'))
```

從這裡不難發現我們訓練了 LUIS 幾種不同的意圖判別 : `'welcome'`(歡迎詞), `'weatherQuery'`(天氣查詢), `'movieQuery'`(電影資訊查詢), `'ConstellationsQuery'`(星座運勢查詢), `'badEmotion'`(壞心情判斷)。其中歡迎詞以及心情的判斷是新添加的意圖分析。

#### 【New Feature】使用者心情判斷

這個功能早在 MoBot 時就一直想要做，希望可以有一個可以隨機回覆給使用者 (家人) 筆者一對兒女的可愛照片，之前一直沒有時間做，便趁著這次更新一次把這功能建立起來。

照片的儲存筆者沒有使用之前用的 imgur，最主要的因素是 imgur 圖床不確定圖片可以保存多久，要放這種較為隱私且希望可以長期調用的還是希望放在專門的網路相簿中，因此，這個功能筆者便選擇使用 Flickr 來儲存相片。

Flickr 也有 python 相對應的 api 工具 [flickr_api](https://github.com/alexis-mignon/python-flickr-api)，筆者利用這個 api 來隨機將特定相本中的圖像做為訊回覆給使用者。

與一般的 API 相同，也要先至官網取得 `KEY` 以及 `SECRET`，然後利用我們的信箱帳號進行使用者確認。

值得注意的是，Flickr 是一個相簿平台，即使我們取得 Flickr 網址也無法做為一張圖像傳送出去，因此我們還是必須從取得的網址中爬取實際的相片網址。

```python=
# 使用者確認
flickr_api.set_keys(api_key=flickr_key, api_secret=flickr_secret)
user = flickr_api.Person.findByEmail(flickr_account)

# 取得相簿 (photosets)
photosets = user.getPhotosets()
# 取得相簿中的相片
photos = photosets[0].getPhotos()
#隨機選取 index
n = np.random.randint(photos.info.total)

# 爬取相片實際網址
res = requests.get(f'https://www.flickr.com/photos/{user_id}/{photos[n].id}/')
res.encoding = 'utf-8'
soup = BeautifulSoup(res.text, 'lxml')


tags = soup.find_all('img')
# 藉由替換字串來得到適當尺寸的圖片
link = tags[0]['src'].replace("n","b")
img_link = f'https:{link}'
line_bot_api.reply_message(event.reply_token, [TextSendMessage(text=f'心情不好嗎 ? 來看看小可愛們吧 ~~~~'), 
                        ImageSendMessage(original_content_url=img_link, preview_image_url=img_link)])            
```

![](https://i.imgur.com/t2gPKcQ.png)



### 圖文選單添加

圖文選單的建立，其實主要是為了讓使用者可以在一開始就上手，也是引導使用者在一開始就能理解使用方式的一種方法。筆者也是從這個地方開始思考整個 LineBot 外觀怎麼樣子可以統一主題。

首先，我們先登入 [Line Official Account Manager](https://manager.line.biz/)，並且選擇我們要加入圖文選單的聊天機器人，在本文中就是 **芋香冬瓜「查」**

![](https://i.imgur.com/vQNUEoC.png)

由左側選取「圖文選單」

![](https://i.imgur.com/rRkWy1d.png)

按下右上角的「建立」以建立新的圖文選單
![](https://i.imgur.com/Gqg4xKD.png)

圖文選單的建立一共有兩個部分需要設定，第一部分主要是關於此圖文選單的詳細資料，這部分隨意設定即可，而時間就可以把區間拉的長一些，以免一陣子就必須重新設定。

![](https://i.imgur.com/bqTYoUz.png)

第二部分則是要為圖文選單設置圖片以及使用的方式，我們先「選擇版型」來確定我們要的圖文選單格式為何

![](https://i.imgur.com/UJFmJvM.png)

有多種樣板可供我們來選擇，在 **芋香冬瓜「查」** 中，由於目前僅有三個主要功能，因此筆者選擇小型三格的版型

![](https://i.imgur.com/rts33e5.png)

點選左側「建立圖片」

![](https://i.imgur.com/7PlRAD7.png)

我們可以為這三格選取不同的圖片。倘若我們希望整個圖文選單的圖片是具有連續性、一致性，也可以利用一整張圖片作為整體背景圖片

![](https://i.imgur.com/B1y6Mde.png)

最後以下圖為例，我們可以為每一格設定一組預設文字，當使用者點選此格圖片，系統便自動會輸入指定文字做為此機器人的輸入。

![](https://i.imgur.com/c1BzEVP.png)

而下圖則是 **芋香冬瓜「查」** 圖文選單的樣貌 : 

![](https://i.imgur.com/R9KnnxC.png)


未來工作
---

**芋香冬瓜「查」** 目前的狀態並非一個已經完結的聊天機器人專案，但可以算是一個初步完整的一個聊天機器人，後續仍然有一些可以增添的功能，也有許多待筆者學習的部分。
 
* 連結 LIFF, Google sheeets 或是 MySQL 來為此機器人添加新功能，如 : 類記事本應用 ? ( 可以參考--[專為高中生設計的管家型聊天機器人](https://drive.google.com/file/d/10Qd-XvznwN_qVkZj2gXK3eLT3PPTqvc3/view) )
* 是否能利用如 Line Notify 加入自動回覆、定時回覆的功能 ?
* 更人性化的對話模式 ?
* 串上 AI 模型，如 : YOLO ? FacnNet ? 或者甚至是 BERT ? 

這些部分都是筆者希望可以嘗試的部分，希望再過一陣子就能迎來 **芋香冬瓜「查」** 在一次的大更新吧 ~~~ ( 希望有時間 =.= )