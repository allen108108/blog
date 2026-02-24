---
title: "利用 Line Notify 於 Line Bot ( Django + Heroku ) 進行群組推播提醒"
date: 2022-03-01 19:53:50
categories:
- 專案 Project
- Line Bot
image: https://i.imgur.com/ewQkjSC.pngg
mathjax: true
---



系列文章
---

* [MoBot : LINE Bot 開發紀錄 ( LINE / Heroku )](https://allen108108.github.io/blog/2020/03/26/MoBot%20_%20LINE%20Bot%20%E9%96%8B%E7%99%BC%E7%B4%80%E9%8C%84%20%28%20LINE%20_%20Heroku%20%29)
* [芋香冬瓜「查」 : 專為自家需求的機器人查詢系統 App ( MoBot ver.2 )](https://allen108108.github.io/blog/2020/06/26/%E8%8A%8B%E9%A6%99%E5%86%AC%E7%93%9C%E3%80%8C%E6%9F%A5%E3%80%8D%20_%20%E5%B0%88%E7%82%BA%E8%87%AA%E5%AE%B6%E9%9C%80%E6%B1%82%E7%9A%84%E6%A9%9F%E5%99%A8%E4%BA%BA%E6%9F%A5%E8%A9%A2%E7%B3%BB%E7%B5%B1%20App%20%28%20MoBot%20ver.2%20%29/)
* [利用 Line Notify 於 Line Bot ( Django + Heroku ) 進行群組推播提醒](https://allen108108.github.io/blog/2022/03/01/%E5%88%A9%E7%94%A8%20Line%20Notify%20%E6%96%BC%20Line%20Bot%20(%20Django%20+%20Heroku%20)%20%E9%80%B2%E8%A1%8C%E7%BE%A4%E7%B5%84%E6%8E%A8%E6%92%AD%E6%8F%90%E9%86%92/)
* [Line Bot 在群組中監控成員的加入與退出](https://allen108108.github.io/blog/2022/03/02/Line%20Bot%20%E5%9C%A8%E7%BE%A4%E7%B5%84%E4%B8%AD%E7%9B%A3%E6%8E%A7%E6%88%90%E5%93%A1%E7%9A%84%E5%8A%A0%E5%85%A5%E8%88%87%E9%80%80%E5%87%BA/)

<!-- more -->

What's Line Notify ?
---
Line Bot 的應用中，除了「被動回覆」使用者外，「主動推播」的功能在 Line Bot 中亦是不可或缺的一環。在商業應用上，主動推播的需求甚至高過於被動回覆，主動推播可以讓業主進行最即時的通知，無論是警示通知或是行銷企劃的宣傳，都會讓使用者對於 Line Bot 的依賴性更強，當然，也可以讓業主的行銷變得更精準，進而帶來後續的收益。

一定有人會問，主動推播功能不就利用帳號直接傳送給使用者就好了嗎 ? 的確，不管是以前的 Line@ 還是 Line Bot，直接利用帳號進行訊息的傳送最簡單，但當使用者人數增加到一定的程度後，訊息的費用也會是一筆可觀的費用，以下是 Line 官方帳號針對訊息的計費方式 : 

![](https://i.imgur.com/X815ciJ.png)

官方帳號的計費方式以用量做區分，分為低、中、高用量，每月的免費訊息則數個別是 500、4000、25000 則，對於精打細算的業主來說，雖然費用不算太高，但也總是期望是否能有更好的選擇。也因此，Line Notify 便是 CP 值相對高的另一種主動推播方案。

Line Notify 我們可以視為是 Line 生態系中的一個小工具，可以讓我們利用其進行免費主動推播，在概念上，其實就相當於我們將要發送的訊息先傳至 Line Notufy 中，再由 Line Notify 來進行推播。


Line Notify 登錄步驟
---

Line Notify 並非 Line 的主體服務，所以我們必須前往 Line Notify 的網站 ( [https://notify-bot.line.me/zh_TW/](https://notify-bot.line.me/zh_TW/) ) 進行相關設定。

利用 Line 的帳號密碼進行登入後，網站右上角會顯示登入的 Line 帳號暱稱，點擊後會出現下拉選單 : 
* 個人頁面 : 顯示目前 Line Notify 的連動狀況，簡單來說就是顯示目前有哪些服務可以使用 Line Notify，另外，此頁面也提供了個人 ( 開發人員 ) 的連動權杖發行。
* 管理登錄服務 : 此頁面針對提供服務的用戶進行 Line Notify 的連動，我們要使用 Line Bot 連棟則是透過這部分來進行。
* 登出

本篇文章主要旨在介紹 Line Bot 如何與 Line Notify 進行連動，因此筆者接下來介紹的會以【管理登錄服務】來進行 Notify 的相關設定。

![](https://i.imgur.com/G4Baw0J.png)

從登入的下拉選單中我們選擇【管理登錄服務】，之後點選【登錄服務】後會出現下圖的欄位需要填寫，每一個欄位都是必填，但除了最後的 Callback 欄位外，其餘的部分其實都簡單填寫即可，不會影響後續的操作。

![](https://i.imgur.com/h5t0jco.png)

Callback 網址的設定主要是參照 Line Bot 的 webhook url 再加上 `/notufy`，以筆者例子來說，將 Line Bot 部署於 heroku 的話 Callback 網址就會是如下的型態 : 
```
https://<LineBot App name>.herokuapp.com/notify
```
填妥上述資訊後，按下 【同意並前往下一步】後，我們可以得到一組 Client ID 以及 Client Secret，連同上面的 Callback url，之後都會在 後續的步驟以及 Django 的設置中使用到。


Access Token
---
當我們要利用 Line Notify 進行訊息推播前，須取得對方的 Access Token 才能進行推播，接下來的部分較為複雜，這也是筆者認為利用 Notify 中最麻煩的部分。

要取得 Access Token 我們需要以下幾個步驟來進行：

1. 先進入連動用網址，利用此網址進行 Line Notify 綁定
2. 取得 User Code
3. 利用 user Code 來取得 User Access Token

#### 連動網址取得

根據 [Line Notify API Document](https://notify-bot.line.me/doc/en/https://notify-bot.line.me/doc/en/)的說明，我們可以利用 `create_auth_link` 函式來進行連動網址的生成。

```python=
def create_auth_link(user_id, client_id=client_id, redirect_uri=redirect_uri):
    
    data = {
        'response_type': 'code', 
        'client_id': client_id, 
        'redirect_uri': redirect_uri, 
        'scope': 'notify', 
        'state': user_id,
        'response_mode': 'from_post'
    }
    query_str = urllib.parse.urlencode(data)
    
    return f'https://notify-bot.line.me/oauth/authorize?{query_str}'
```

其中參數說明如下：

* response_type : 輸入字串`code`
* client_id : 輸入 Line Notify 的 Client ID
* redirect_url : 輸入 Line Notify 的 Callback URL
* scope : 輸入字串 `notify`
* state : 輸入任意亂碼字串，目的是避免網路攻擊。
* response_mode : 輸入字串 `from_post`


#### User Code 取得

將生成出來的網址放入瀏覽器中進行連結，會出現一個頁面，可以選擇要連結 Notify 服務的群組

![](https://i.imgur.com/Tsiojr2.png)

選取要連結之群組後，按下【同意並連動】之按鈕後，網址會轉換成下列型態：

```
<Callback URL>?code=<CODE>&state=<STATE>
```

並且在 Line Notify 中會即時傳送一個群組連動成功之訊息

![](https://i.imgur.com/2L8CrL9.png)

此時我們可以將 Notufy 帳號加至欲連動之群組中。

#### User Access Token 取得

利用 Line Notify API 來進行 User Access Token 的取得

```python=
token_get_url = 'https://notify-bot.line.me/oauth/token'
params = {
    'grant_type':'authorization_code',
    'code':<CODE>,
    'redirect_uri': redirect_uri,
    'client_id': client_id,
    'client_secret': client_secret,

}
get_token = requests.post(token_get_url,params=params)
```

數數說明 ：

* grant_type : 輸入字串`authorization_code`
* code ： 擷取出 `<CODE>` 的部分填入
* redirect_url : 輸入 Line Notify 的 Callback URL
* client_id : 輸入 Line Notify 的 Client ID
* client_secret : 輸入 Line Notify 的 Client Secret


利用這樣的方式我們便可以取得群組的 Access Token，後續 Line Notify 便會根據這個token 來針對特定群組進行訊息的推播。

Django 的設置
---

在 Line Bot 側，我們需要監控 Notify 是否有推播訊息，因此在 Line Bot 程式中要加入以下程式碼 ：

`views.py`

```python=
def notify(request):
    #取得 code
    pattern = 'code=.*&'
    raw_uri = request.get_raw_uri()
    codes = re.findall(pattern,raw_uri)
    for code in codes:
        code = code[5:-1]
        print(code)

    #取得 token
    user_notify_token_get_url = 'https://notify-bot.line.me/oauth/token'
    params = {
        'grant_type':'authorization_code',
        'code':code,
        'redirect_uri': <Callback>
        'client_id':<Client ID>
        'client_secret': <Client Secret>

    }
    get_token = requests.post(user_notify_token_get_url,params=params)
    #print(get_token.json())
    token = get_token.json()['access_token']
    #print(token)

    #抓取user的info
    user_info_url = 'https://notify-api.line.me/api/status'
    headers = {'Authorization':'Bearer '+token}
    get_user_info = requests.get(user_info_url,headers=headers)
    #print(get_user_info.json())
    notify_user_info = get_user_info.json()
    return HttpResponse()
```

至此，我們已經完成了所有 Line Notify 連動的相關工作。

訊息推播
---

我們目前已經可以取得要推播的群組之 Access Token，就可以來進行訊息的推播了，透過下面的 script 我們可以將想要傳送的訊息透過 API 進行推播 ：


```python=
api = 'https://notify-api.line.me/api/notify'
message = '【LineBot】測試通知 : Notify 訊息測試 !'
headers = {
  "Authorization": "Bearer " + access_token, 
  "Content-Type" : "application/x-www-form-urlencoded"
}

payload = {'message': message}
r = requests.post("https://notify-api.line.me/api/notify", headers = headers, params = payload)
```

之後我們便可以在該群組中收到訊息如下 ：

![](https://i.imgur.com/ik379lP.png)


後記
---

這篇文章主要是要讓讀者了解整個 Notify 推播的主要流程，熟悉這樣的流程後，其實可以有更多的應用面，例如，我們可以利用最後的 script 搭配 Django 資料庫系統將各個群組的 Access Token 進行儲存，也可以將這段 Script 寫進 API 中，在特定狀況下直接 trigger API 達到自動推播的效果。

不過正如同本文最前面提到的，提供服務的 Line Bot 在訊息中其實並沒有很明顯的辨識度，這是 Notify 在商業應用中較為可惜的一點，倘若讀者可以接受這樣的呈現方式，那麼 Line Notify 確是一個蠻不錯的推播工具的。