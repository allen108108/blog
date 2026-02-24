---
title: "MoBot : LINE Bot 開發紀錄 ( LINE / Heroku ) "
date: 2020-03-26 00:05:12
categories:
- 專案 Project
- Line Bot
image: https://i.imgur.com/ve1qLyn.png
mathjax: true
---

系列文章
---

* [芋香冬瓜「查」 : 專為自家需求的機器人查詢系統 App ( MoBot ver.2 )](https://allen108108.github.io/blog/2020/06/26/%E8%8A%8B%E9%A6%99%E5%86%AC%E7%93%9C%E3%80%8C%E6%9F%A5%E3%80%8D%20_%20%E5%B0%88%E7%82%BA%E8%87%AA%E5%AE%B6%E9%9C%80%E6%B1%82%E7%9A%84%E6%A9%9F%E5%99%A8%E4%BA%BA%E6%9F%A5%E8%A9%A2%E7%B3%BB%E7%B5%B1%20App%20%28%20MoBot%20ver.2%20%29/)
* [Line Bot 在群組中的喚醒/休眠機制 (Django Database 之應用)](https://allen108108.github.io/blog/2021/08/30/Line%20Bot%20%E5%9C%A8%E7%BE%A4%E7%B5%84%E4%B8%AD%E7%9A%84%E5%96%9A%E9%86%92_%E4%BC%91%E7%9C%A0%E6%A9%9F%E5%88%B6%20(Django%20Database%20%E4%B9%8B%E6%87%89%E7%94%A8)/#more)
* [利用 Line Notify 於 Line Bot ( Django + Heroku ) 進行群組推播提醒](https://allen108108.github.io/blog/2022/03/01/%E5%88%A9%E7%94%A8%20Line%20Notify%20%E6%96%BC%20Line%20Bot%20(%20Django%20+%20Heroku%20)%20%E9%80%B2%E8%A1%8C%E7%BE%A4%E7%B5%84%E6%8E%A8%E6%92%AD%E6%8F%90%E9%86%92/)
* [Line Bot 在群組中監控成員的加入與退出](https://allen108108.github.io/blog/2022/03/02/Line%20Bot%20%E5%9C%A8%E7%BE%A4%E7%B5%84%E4%B8%AD%E7%9B%A3%E6%8E%A7%E6%88%90%E5%93%A1%E7%9A%84%E5%8A%A0%E5%85%A5%E8%88%87%E9%80%80%E5%87%BA/)

<!-- more -->

(2019.06) 

自學深度學習與 Python 將近一年，也該為這一段學習做一個階段性的總整理。而最好的方式便是試著實作一個專案，大概在今年中我便決定了兩個主要的專題方向 -- LINE Bot 與 人臉辨識。

本篇文章主要以建構一個可用且符合自己需求的 LINE Bot 的過程紀錄，因此會持續更新文章，直到整個 LINE Bot 達到我自己想要的標準為止。



(2020.03)

這篇文章的撰寫，從開始到完成，中間隔了近半年，主要是因為筆者這段期間適逢求職以及就職的過程，加上工作上的專案以及領域的論文閱讀，導致此篇開發記錄一直沒有機會完成，一直到現在才抽空將其補完。


MoBot 功能
---

MoBot 主要都是以自己以及家人為需求進行開發，因此目前 ( 2019.09.20 ) 先完成幾個功能 : 
1. 提供當下36小時的氣象預報
2. 查詢當日電影場次
3. 當週星座運勢


<img width=500 src="https://i.imgur.com/haTp0nw.jpg" >




MoBot 架構
---

* 平台 -- LINE
MoBot 是這個 ChatBot 的名稱，而 MoBot 是一個建立在 LINE 平台之上的 ChatBot。之所以選擇以 LINE 為平台，主要原因也是台灣對於 LINE 的高度依賴性所致。

* 架構 -- Django
能夠實現 LINE Bot 的方式有很多，由於我是參照書上指示進行建構，因此便以 Django 為 MoBot 的設計架構。

* 伺服器 -- heroku
目前非自架伺服器的選擇大多是 ngrok 以及 heroku，前者重新執行後都會改變網址，對於要常態性使用上會相對不是這麼好用。heroku 有免費的方案，在架構上只要滿足其要求，往後的更動都只須調整代碼即可，其餘的都不太需要多操心，可以算是初級 LINE Bot 開發者的首選。

* 語句理解 -- LUIS
LUIS ( *Language Understanding Intelligent Service* )，是微軟開發的一套雲端 NLP 服務，可藉由使用者輸入語句及實體標識來進行訓練，訓練完成後，我們便可以將接收到的訊息藉由 API 判斷訊息的意象 ( Intent ) 做語句上的 NER ，進而針對接收到的訊息來做出適當的回覆。

* 資料擷取 -- 中央氣象局 API
現階段 (2019.09.19) MoBot 開發出來的功能有二 : 
( 1 ) 當日天氣查詢 ( 2 ) 電影場次查詢(限大台北地區) 。
天氣查詢部分接收到訊息後確認待查詢縣市後依靠中央氣象局公開資料 API 擷取「36小時天氣預報資料」資訊，將我們要的縣市資訊抓出來再回覆給使用者。

* 資料擷取 -- imgur API
這部分算是很額外的部分，電影場次查詢本來只預計從「開眼電影網」抓到當日台北各戲院場次網址回覆就好，但我希望可以將電影的海報也一併回覆出去，因此必須要多繞一圈。由於 LINE 圖片回覆網址必須要是 https，但許多的圖片都是 http，因此必須將擷取出來的圖片網址上傳到 imgur 後，再傳回給使用者，這部分就需要經過 imgur API 來完成。

* Google YouTube API ( 2019.09.20 新增 )
增加了可以查詢當周運勢的部分，以唐綺陽的分析為爬蟲對象，原本想要從臉書粉絲專業來進行資料爬取，但最後發現由 YouTube 來爬會相對簡單很多。利用 google API 可以用簡單的方式來取得 YouTube 內容。



LINE 平台佈屬
---
要在 LINE 上製作 ChatBot，首先必須先申請 LINE 開發者帳號。申請過程不算太困難。

### 1. 先至 https://developers.line.biz/en/ 進行註冊

![](https://i.imgur.com/lWHkqdV.jpg)

### 2. 建立新的 Provider
於 Provider 頁面按下右上角 `Create New Provider`

![](https://i.imgur.com/SyUJxL1.jpg)

### 3. 填入 Provider 名稱後便成功建立了一個新的 Provider

![](https://i.imgur.com/TSzHT27.png)

### 4. 選取 `Messageing API` 以建立一個 LINE Bot


![](https://i.imgur.com/CeLeY3w.png)


### 5. 設定 LINE Bot 基本資料

![](https://i.imgur.com/cY1oP4e.png)
![](https://i.imgur.com/oOwCpcJ.png)

### 6. 確認建立 Provider
版權部分按下同意鍵後，於 Confirm 步驟最下方兩個選項均打勾後按下 `Create`

![](https://i.imgur.com/J3KBxpn.png)

### 7. 設定 LINE Bot 
完成上述過程後，便會跳到剛剛建好的 Provider 頁面，此時你也可以看到剛建立的 LINE Bot 已經出現，此時點選 LINE Bot 進行設定。

![](https://i.imgur.com/kjMqvuY.png)

###  設定 LINE Bot 有幾個部分需要注意的地方 ( 其他部分都不需更動 ) : 
* #### **Channel ID、Channel secret、Channel access token (long-lived)**
這三個部分務必記住，之後在 LINE Bot 程式碼以及 Heroku 佈署中都會使用到，其中 Channel secret、Channel access token 旁邊的 issue 鍵每按一次就會變更一次，因此不要隨便按，issue 鍵按下後會跳出視窗，可以設定權限 key 的時效，如果為 0 便表示沒有時效限制。

![](https://i.imgur.com/ihnsuHV.png)


* #### **Use webhooks**
按下右邊 Edit 鍵，將值從 `Disaled` 改為 `Enabled`




* #### **Webhook URL** 
這部分暫時先保留，待佈署 Heroku 後會需要回填進 URL 來跟 Heroku 做連結。

![](https://i.imgur.com/IlQSPKm.png)


* #### **Auto-reply messages、Greeting messages**
這兩個部分都可以將值先改為 `Disaled` ，這樣方便在測試時不會有太多額外的干擾存在。

![](https://i.imgur.com/Kx98ocs.png)


<img width=500 src="https://i.imgur.com/ahPZDeK.png" >




設定完成後也代表 LINE 上面的佈署告一段落，接著我們要進行 Django 的架構佈署。

Django 架構佈屬
---

這邊我不多介紹 Django ，簡而言之 Django 就是為了網站開發人員所設計的以 Python 所編寫的網站框架[^1]。



### 1. **安裝 Django 及 LINE Bot API**

於終端機上利用 `pip install django`、`pip install line-bot-sdk` 進行安裝

### 2. **建立 Django 專案**

確認專案要放置在哪一個路徑下，切換至此路徑後輸入
`django-admin startproject 自訂專案名稱` 即可於此路徑下建立此專案。

建立完成後切換至專案資料夾中，此時可以發現整個專案的結構如下 : 

<img width=500 src="https://i.imgur.com/kgTalrm.jpg" >




在 Django 中，一個專案可以對應到多個應用程式 (Application)，我們要建立一個 LINE Bot 前就必須建立一個 application 應用

`python manage.py startapp 自訂應用程式名稱`

建立 application 應用以後，專案資料夾底下就會多出一個應用程式資料夾。這是一個最基本的 Django LINE Bot 架構，而下圖紅色標註的部份便是我們接下來要進行更動的部分。



<img width=500 src="https://i.imgur.com/qqm0Zvi.png" >
( 假設應用程式名稱自訂為 `mybot` )

 

雖說基本架構如上，但還是建議建立一個完整的 Django 專案，因此依照下列方式在專案資料夾內建立 templates、static資料夾，以及 migration 資料檔

```
md templates 
# 放置模板 (.html檔)

md static 
# 儲存圖形檔、CSS 及 JavaScript 檔案

python manage.py makemigrations 
#此步驟會在 mybot app資料夾中產生 migration 資料夾
```

* ### 設定 **settings.py**

settings.py 是整個專案的環境設定檔案， 
找到下面兩列程式碼
```=22
# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = '*********************'
```
在後面加上兩列，並填入在上述 LINE 平台設定上取得的 Channel access token 以及 Channel secret

```python=24
LINE_CHANNEL_ACCESS_TOKEN='LINE 的 Channel access token'
LINE_CHANNEL_SECRET='LINE 的 Channel secret'
```
設定外部連結均可以連到本機伺服器

```python=30
ALLOWED_HOSTS = ['*']
```

將我們的 app 加入 INSTALLED_APP 中

```python=35
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'mybot',  #新增的app
]

```

設定我們已經建立的模板路徑

```python=57
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR,'templates')], 
        # 加上 templates 路徑
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

設定語系及時區

```python=109
LANGUAGE_CODE = 'zh-Hant' # 繁體中文

TIME_ZONE = 'Asia/Taipei' # 台北時區
```

設定 static 路徑

```python=120
# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/2.2/howto/static-files/

STATIC_URL = '/static/'
STATICFILES_DIR=[os.path.join(BASE_DIR,'static'),]
```

* ### 設定 **urls.py**

在 Django 中要做出一個能夠回應使用者的網頁，主要靠的就是 urls.py 與 views.py 這兩個檔案的互動，從 urlpatterns 來確認我們要從 views.py 中調用什麼函式來作回應。

因此我們先進行 urls.py 的設定如下 : 

```python=16
from django.contrib import admin
from django.urls import path
from django.conf.urls import url
from mybot import views

urlpatterns = [
    url('^callback',views.callback), 
    # 從 views.py 調用 callback 函式來回應
    path('admin/', admin.site.urls),
]
```

* ### 設定 **views.py**

views.py 主掌了整個 LINE Bot 的回應，因此在這邊我們會有較大的篇幅來進行解說。此外，我仍建議剛開始進行 LINE Bot 開發的人，可以先完成 Echo Bot 的佈署，確定整的佈署到使用端都沒有問題後，再進行其他功能的增加。

想要確定 views.py 該怎麼寫，要定義哪函式，那麼就得先了解自己需要 LINE Bot 擁有什麼樣的功能，又要怎麼實現，下圖則是目前 MoBot 的整個功能架構圖。

![](https://i.imgur.com/ve1qLyn.png)


在確定了 callback 函式大概要做的事情後，我們先來定義幾個函式 : 

`sendUse()` 處理使用者輸入 `@使用說明` 時 MoBot　會回傳的一個簡易使用說明。

其中值得注意的是　`line_bot_api.reply_message(
            event.reply_token, TextSendMessage(text='腦子故障中，請稍等 Q_Q'))`　這是 LINE API 的一個固定回傳文字的方式 `text` 後面則是接希望 LINE Bot 回覆的文字。

```python=
def sendUse(event):
    try:
        text1 = '''
查詢天氣  : \n請輸入「 XXXX 天氣如何? 」\n例如 : 「高雄天氣如何?」\n
查詢電影場次  : \n請輸入「 XXXX 電影資訊 」\n例如 : 「第九分局電影資訊」\n
查詢本週星座運勢  : \n請輸入「 本週運勢」\n
              '''
        message = TextSendMessage(
            text=text1
        )
        line_bot_api.reply_message(event.reply_token, message)
    except:
        line_bot_api.reply_message(
            event.reply_token, TextSendMessage(text='腦子故障中，請稍等 Q_Q'))
```

`sendLUIS()` 則是處理天氣查詢以及電影場次查詢的部分。

[LUIS](https://www.luis.ai/) (Language Understanding Intelligent Service) 是微軟開發的一套線上雲端語言處理系統，藉由使用者給定的 examples 以及實體命名，線上訓練模型，可以針對輸入文字進行命名實體識別 ( NER ) 以及語句意向的預測[^2]。



MoBot 利用 LUIS 對使用者的語句進行意向預測，確認使用者要取得天氣還是電影資訊。如果偵測使用者要取得天氣資訊，則利用氣象局 API 針對使用者的需求將文字資料提供出來。

電影資訊的取得相對來說比較麻煩。

我希望可以 MoBot 不單單可以回覆某一個電影的台北戲院場次，在連結之前還要先丟出一張海報圖。因此 MoBot 裡面使用 BeautifulSoup 來進行「開眼電影網」的資料取得。由於每一部電影在開眼電影網內都有一個專屬 ID，我們必須將其找出後，嵌入當天場次表的網址之中，就可以得到每一部電影的當日場次表。

![](https://i.imgur.com/fNF5KO6.png)

其中 a02 為大台北地區代碼 ( 我們也藉此可以找出不同地區的場次表 )
![](https://i.imgur.com/5S5glc3.png)

圖片的部分比較麻煩，因為在開眼電影網裡面，電影的海報被包在頁面內的 json 裡面，因此需要用 `pic_link = json.loads(p_soup.find(
                    'script', {'type': 'application/ld+json'}).get_text())["image"]` 來取得 image 網址。
                    
此外，LINE Bot 要傳送圖片訊息必須是 https 開頭網址，但絕大多數我們可以撈的到的圖片都是 http 網址，因此必須借助 imgur API 將我們撈到的圖片先上傳到 imgur 後再利用其生成的網址來傳送。 

```python=
def sendLUIS(event, mtext):
    try:

        r = requests.get('LUIS 終結點網址'+mtext)
        result = r.json()

        cities = []
        movies = []

        if result['topScoringIntent']['intent'] == '縣市天氣':
            for en in result['entities']:
                if en['type'] == '地點':
                    cities.append(en['entity'])
                    break
        elif result['topScoringIntent']['intent'] == '電影時刻':
            for en in result['entities']:
                if en['type'] == '電影名稱':
                    movies.append(en['entity'])
                    break

        if cities:
            cities[0] = cities[0].replace('台', '臺')
            api_link = f"http://opendata.cwb.gov.tw/opendataapi?dataid={氣象局資料代碼}&authorizationkey={使用者API授權碼}"

            report = requests.get(api_link).text

            xml_namespace = '{urn:cwb:gov:tw:cwbcommon:0.1}'
            root = et.fromstring(report)
            dataset = root.find(xml_namespace+'dataset')
            locations_info = dataset.findall(xml_namespace+'location')

            location = cities[0]

            target_index = -1

            for i, loc in enumerate(locations_info):
                locationName = loc[0].text
                if location in locationName:
                    target_index = i
                    break
            if target_index != -1:
                show = location+'的天氣如下 :\n'
                tlist = ['天氣狀況', '最高溫', '最低溫', '舒適度', '降雨機率']
                for j in range(5):
                    element = locations_info[target_index][j+1]
                    timeblock = element[1]
                    data = timeblock[2][0].text
                    show = show+tlist[j]+' : '+data+'\n'
                line_bot_api.reply_message(
                    event.reply_token, TextSendMessage(text=show[:-1]))
            else:
                line_bot_api.reply_message(
                    event.reply_token, TextSendMessage(text='無此地點天氣資料！'))

        elif len(movies) != 0:
            at_link = 'http://www.atmovies.com.tw/movie/now/'
            res = requests.get(at_link)
            res.encoding = 'utf-8'
            soup = BeautifulSoup(res.text, 'lxml')
            a_tags = soup.find_all('a')
            film_id = []
            for tag in a_tags:
                if movies[0] in str(tag.string):
                    film_id.append(tag.get('href')[7:-1])

            if len(film_id) != 0:
                movie_link = f'http://www.atmovies.com.tw/showtime/{film_id[0]}/a02/'
                p_res = requests.get(movie_link)
                p_res.encoding = 'utf-8'
                p_soup = BeautifulSoup(p_res.text, 'lxml')
                pic_link = json.loads(p_soup.find(
                    'script', {'type': 'application/ld+json'}).get_text())["image"]
                img_link = upload_photo(pic_link.strip())
                #line_bot_api.reply_message(event.reply_token, ImageSendMessage(original_content_url=img_link, preview_image_url=img_link))
                line_bot_api.reply_message(event.reply_token, [ImageSendMessage(
                    original_content_url=img_link, preview_image_url=img_link), TextSendMessage(text=f'{movies[0]} 台北戲院播映場次 :\n{movie_link}')])
            else:
                line_bot_api.reply_message(
                    event.reply_token, TextSendMessage(text='無此電影資訊！'))

    except Exception as e:
        import traceback
        traceback.print_exc()
        #line_bot_api.reply_message(event.reply_token, TextSendMessage(text='查詢時產生錯誤，請稍後再查詢唷~'))
```
這裡我們必須多定義一個函式，return 圖片在 imgur 中生成的網址，函式內部的運作可以參照 [imgurpython library](https://github.com/Imgur/imgurpython) 。

```python=
def upload_photo(image_url):
    client_id = '使用者 imgur ID'
    client_secret = '使用者 imgur secret'
    access_token = '使用者 imgur access_token'
    refresh_token = '使用者 imgur refresh_token'
    client = ImgurClient(client_id, client_secret, access_token, refresh_token)
    album = None  # You can also enter an album ID here
    config = {
        'album': album,
    }

    #print("Uploading image... ")
    image = client.upload_from_url(image_url, config=config, anon=False)
    # print("Done")
    return image['link']
```
LINE API 上對於圖片訊息的處理如下，我們只要將上面的函式回傳的網址填進去即可。

`
line_bot_api.reply_message(event.reply_token, ImageSendMessage(original_content_url=img_link, preview_image_url=img_link))
`

但這裡有一個部分是比較少人關注的，因為我希望 LINE Bot 是可以回覆一張海報圖後再接著傳送場次網址，等同於要傳一張圖片訊息跟一個文字訊息。

原本我以為可以這樣

```python
line_bot_api.reply_message(event.reply_token, ImageSendMessage(original_content_url=img_link, preview_image_url=img_link))
line_bot_api.reply_message(event.reply_token,  TextSendMessage(text=f'{movies[0]} 台北戲院播映場次 :\n{movie_link}'))
```

但這樣始終會跑不出來，後來這個問題在 LINE Developer 的 Message API 解說頁面找到答案

![](https://i.imgur.com/1gGjUGp.png)

連續回覆必須要用 array 的型態來表示，因此上面那兩行應該改為 : 

```python
line_bot_api.reply_message(event.reply_token, [ImageSendMessage(
                    original_content_url=img_link, preview_image_url=img_link), TextSendMessage(text=f'{movies[0]} 台北戲院播映場次 :\n{movie_link}')])
```


Heroku 伺服器佈屬
---

* ### 創建 Heroku app 

要部屬到 Heroku，我們得先進行註冊，註冊過程與一般網站大同小異，這裡就不贅述，我們直接跳到部屬的階段。

![](https://i.imgur.com/ICztWOv.png)

按下 `Create new app` 以建立新的 app 後，先輸入 app 的名稱，這邊要特別注意的地方是不能宇其他使用者所創建的 app 名稱重複。

![](https://i.imgur.com/C0nR95f.png)


* ### 安裝 Git 版本控制系統

Heroku 上的專案部屬是基於 Git 版控系統，因此在部屬 Heroku 前必須要安裝 Git 系統。安裝很簡單，至 [Git 下載頁面](https://git-scm.com/download/win)選擇適合的版本下載並無腦安裝即可。

![](https://i.imgur.com/clSlAC9.png)

題外話，Git 系統在版本控制上面非常好用，無論是在 Heroku 的部屬，或是在 Github 上面要進行專案的協作，都是不可或缺的工具之一。



* ### 安裝 Heroku CLI

Heroku CLI 全名 Heroku Command Line Interface，是 Heroku 官方的一種命令行介面，藉由 Heroku CLI 可以利用 Git 讓整個專案在 Heroku 上的部屬變得更加容易簡單。


[Heroku CLI 下載頁面](https://devcenter.heroku.com/articles/heroku-cli)中一樣選擇適合的版本下載安裝。
![](https://i.imgur.com/SJJSnoN.png)

安裝結束後，我們可以開啟提示命令字元輸入 `heroku --version` 來驗證是否 Heroku CLI 已安裝完成。

* ### 登入 Heroku，並上傳專案至 Heroku

依序輸入下列指令


登入 Heroku
```
$ heroku login 
```
在本機專案資料夾中建立 git 版本庫
```
$ git init
```
本機專案資料夾與 Heroku app 專案作連結
```
$ heroku git:clone -a <app_name>
```
將資料夾中所有資料加入 git 版本庫追蹤
```
$ git add .   
```
將這次 commit 執行命名為 "update project"
```
$ git commit -am "update project"
```
把 master 的內容提交到遠端的 Heroku 上面
```
$ git push heroku master
```

* ### 專案修改

掌握了上面的步驟，日後在本機上修改專案內容後只需要依序執行下列命令即可

```
$ git add . 
$ git commit -am "update project"
$ git push heroku master
```

Heroku 作為一個免費的機器人伺服器算是蠻好用的，但有些要注意的狀況。由於 Heroku 仍存在免費流量額度，為了讓使用者能夠節約流量，Heroku 會在 30 分鐘未使用的狀態下進行休眠，再次喚醒需要約 20 秒的時間。

因此當機器人未收到訊息就會進入休眠，下一次丟訊息給機器人時，需要等待一段時間機器人才會進行回覆。

後記
---

這個專案是求職前的一個簡易 QA 聊天機器人的實現，有一部分也可以算是一個實驗性質的專案，筆者在這個專案裡面用不同的方式進行機器人的應答過程。

這個專案仍然不夠完善，筆者最希望可以加入類似 BERT 的模型，讓機器人不只有 QA 功能，還能針對使用者的訊息進行更人性化的閒聊，達成真正的「聊天」機器人，但後續由於工作的關係，便沒有繼續進行開發，望日後能夠繼續補強 MoBot 的功能。



參考資料
---

1. 文淵閣工作室。[*Python與LINE Bot機器人全面實戰特訓班*](https://www.books.com.tw/products/0010830739)。碁峰出版社。
2. 何敏煌, 林亮昀。[*Python新手使用Django架站技術實作：活用Django 2.0 Web Framework建構動態網站的16堂課*](https://www.books.com.tw/products/0010790747)。博碩出版社。
3. [Git教學：如何 Push 上傳到 GitHub？](https://gitbook.tw/chapters/github/push-to-github.html)

註釋
---

[^1]: 
"*[Python新手使用Django架站技術實作：活用Django 2.0 Web Framework建構動態網站的16堂課](https://www.books.com.tw/products/0010790747)* " , page 1-04。

[^2]: 
LUIS 並非是唯一的選擇，目前也有許多不同的工具可供選擇，例如 google DialogFlow 或是威盛電子 OLAMI 都是不錯的選擇。