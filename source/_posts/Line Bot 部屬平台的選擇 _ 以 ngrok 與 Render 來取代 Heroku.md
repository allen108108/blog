---
title: "Line Bot 部屬平台的選擇 : 以 ngrok 與 Render 來取代 Heroku"
date: 2023-04-20 13:57:45
categories:
- 專案 Project
- Line Bot
image: https://i.imgur.com/Tz6F3DH.png
mathjax: true
---

系列文章
---

* [MoBot : LINE Bot 開發紀錄 ( LINE / Heroku )](https://allen108108.github.io/blog/2020/03/26/MoBot%20_%20LINE%20Bot%20%E9%96%8B%E7%99%BC%E7%B4%80%E9%8C%84%20%28%20LINE%20_%20Heroku%20%29)
* [芋香冬瓜「查」 : 專為自家需求的機器人查詢系統 App ( MoBot ver.2 )](https://allen108108.github.io/blog/2020/06/26/%E8%8A%8B%E9%A6%99%E5%86%AC%E7%93%9C%E3%80%8C%E6%9F%A5%E3%80%8D%20_%20%E5%B0%88%E7%82%BA%E8%87%AA%E5%AE%B6%E9%9C%80%E6%B1%82%E7%9A%84%E6%A9%9F%E5%99%A8%E4%BA%BA%E6%9F%A5%E8%A9%A2%E7%B3%BB%E7%B5%B1%20App%20%28%20MoBot%20ver.2%20%29/)
* [Line Bot 在群組中的喚醒/休眠機制 (Django Database 之應用)](https://allen108108.github.io/blog/2021/08/30/Line%20Bot%20%E5%9C%A8%E7%BE%A4%E7%B5%84%E4%B8%AD%E7%9A%84%E5%96%9A%E9%86%92_%E4%BC%91%E7%9C%A0%E6%A9%9F%E5%88%B6%20(Django%20Database%20%E4%B9%8B%E6%87%89%E7%94%A8)/#more)
* [利用 Line Notify 於 Line Bot ( Django + Heroku ) 進行群組推播提醒](https://allen108108.github.io/blog/2022/03/01/%E5%88%A9%E7%94%A8%20Line%20Notify%20%E6%96%BC%20Line%20Bot%20(%20Django%20+%20Heroku%20)%20%E9%80%B2%E8%A1%8C%E7%BE%A4%E7%B5%84%E6%8E%A8%E6%92%AD%E6%8F%90%E9%86%92/)
* [Line Bot 在群組中監控成員的加入與退出](https://allen108108.github.io/blog/2022/03/02/Line%20Bot%20%E5%9C%A8%E7%BE%A4%E7%B5%84%E4%B8%AD%E7%9B%A3%E6%8E%A7%E6%88%90%E5%93%A1%E7%9A%84%E5%8A%A0%E5%85%A5%E8%88%87%E9%80%80%E5%87%BA/)

<!-- more -->

前言
---

呼~工作之後，整個部落格基本上要變成年更了，還好我不靠這吃飯，反正這部落格本來就是拿來記錄一切技術小事情的地方，就隨意寫，大家也隨意看吧 XDDDDD

好啦，言歸正傳。

Heroku 從去年 11 月底開始，停止了所有的免費方案，這導致像我這種開發 Line Bot 純粹給自己玩的這種開發者頓時失去了依靠，也因為這樣，去年底之後我幾乎都只用 ngrok 來做短暫的開發。


ngrok
---

[ngork](https://ngrok.com/) 簡而言之就是一個代理伺服器，利用簡單的一行指令就可以建立一個 http / https 的伺服器，並與我們本機端的 Localhost 建立一個安全的通道 (tunnel)，所以我們在本機運行的服務就可以經由這個通道安全地方在網路上使用。

![](https://i.imgur.com/ykzH92q.png)

### 註冊 ngrok 並取得 Authtoken

註冊登入的程序基本上不用多說，每個網站都大同小異，
註冊完成登入後進入[ Authtoken](https://dashboard.ngrok.com/get-started/your-authtoken) 頁面便可取得屬於我們個人的 Authtoken。

![](https://i.imgur.com/4oB6Lpn.png)


### 啟動 Django

接著我們進入到我們自己的 Line Bot 專案資料夾中啟動 Django

```python=
$ cd <your line bot project>
$ python manage.py runserver
```
此時應該會在終端機中看到本機伺服器被啟動的相關資訊 (如下圖)

![](https://i.imgur.com/cigQFxD.png)

一般的 Web 專案，若在瀏覽器上輸入 `http://127.0.0.1:8000/` 便可在本機端看到網頁設計的畫面，其中 `8000` 便為 Django 預設的 port，這也與我們的接下來的步驟相關聯。

### 安裝 ngrok agent

本機側的部分都準備好了，接著就是要準備跟 ngrok 進行對接，首先我們進入[ ngrok 下載頁面](https://ngrok.com/download)，在自己的本機上要先安裝 ngrok agent。

![](https://i.imgur.com/9dOEALC.png)

找到本機 OS 與對應版本後即可下載安裝。


### ngrok agent 連結至 ngrok 帳戶，並啟動 ngrok

本機內的 ngrok agent 與帳戶連結則是靠著下面的兩行指令

```pyhton=
#將個人 Authtoken 加入 ngrok agent 中
$ ngrok config add-authtoken <your Authtoken>

#利用 ngrok agent 與本機專案的 port (預設 8000) 開啟安全通道
$ ngrok http 8000
```

執行 ngrok agent 後，終端機會看到下列資訊

![](https://i.imgur.com/Sr7bbkc.png)

其中 `Forwarding` 欄位指的是我們利用 ngrok 進行連接埠轉發的資訊，簡單來說我們利用上圖中的 `https://6f06-210-200-31-130.ngrok.io` (紅框處) 可以連結到我們本機的 8000 port。

### Line Bot Webhook 修改

我們利用 ngrok 取得了一個 https 的網址 ( 非 IP )，可以連接到我們本機的專案，完全符合 Line Bot Webhook 網址的需求，因此最後我們只要將上圖紅框處的網址依下列方式複製貼上到 [Line Developer](https://developers.line.biz/en/) 設定中即可。

```
<Forwarding URL> + '/callback'
```

依本例，我們在 Webhook URL 中應該貼上的是 `https://6f06-210-200-31-130.ngrok.io/callback`

結束這個步驟後，只要連線、安全通道未斷，我們設計的 Line Bot 就可以正常運作。


### ngrok 免費版本限制

當然，ngrok 提供了免費及收費的服務，免費的部分在使用上必然會有一定的限制，詳細情況可見[官網](https://ngrok.com/pricing)。

筆者這邊就講兩個最顯著的使用限制 : 

1. 暫時性 : 也就是說 ngrok 建立的安全通道是短暫可用的，據網路的資料顯示，可用時間應為 8 小時，8小時候就必須重啟 ngrok。 ( 但根據筆者開發經驗，一段時間閒置之後就會自行斷線，必須重新連線，即使未達 8 小時也會如此 )

2. 隨機網域 : 當我們每一次重啟 ngrok 的時候，Forwarding 的網址是隨機取得，因此每一次崇連，我們就必須進入 Line Developer 平台進行 Webhook URL 的更新。


雖然 ngrok 可以直接在本機端進行部屬，但缺少了 Heroku 過去閒置後僅需要 warmup 後即可使用的便利性，若人不在電腦前，一旦服務斷線便無法補救。在開發階段或許 ngrok 仍是一個不錯的選擇，但若進入實際的測試階段或是上線接段，這樣的服務很難滿足需求。


Render
---

Render 筆者認為就是一個與 Heroku 定位幾乎相同的競品，所以主打的客群基本上是相同的，這點 Render 本身也知道，所以在自己的網站上他們也與 Heroku 進行了比較，若有興趣的讀者可以自行去閱讀 ( [Render vs Heroku](https://render.com/render-vs-heroku-comparison) )。

![](https://i.imgur.com/qtStrlI.png)

但本文主要是針對 Line Bot 的部屬，筆者就不針對平台的詳細功能多加著墨，則以 Line Bot 專案的部屬流程來做介紹。

### 將專案 push 到 Github 上

Render 本身可以跟 Github 做連動，因此將傳案 push 到 Github 上就可以直接針對帳號底下的專案進行部署。

### 自 Render 建立服務

初次使用時，在 Render 網站中點擊右上角 Dashboard 按鈕後會進入服務內容選擇，本次要部署 Line Bot 因此選擇 Web Services 這ㄒ項服務功能

![](https://i.imgur.com/sINFm6U.png)


點擊下方的 `New Web Service` 後便會與你的 Github 帳號連結，可以選擇你要部署的專案項目。

![](https://i.imgur.com/a88oLNp.png)

選擇指定專案後按下 `Connect` 按鈕，即會跳轉至設定頁面如下

![](https://i.imgur.com/go7kju4.png)

* Name : 設定該服務名稱
* Build Command : 服務建置時會需要安裝的套件等等，輸入 `pip install -r requirements.txt`
* Start Command : 啟動網路服務，輸入 `gunicorn SharpLineBot.wsgi:application`

### 部署服務

設定完成後，按下右上方的 `Manual Deploy`，選擇 `Deploy latest commit` 即可完成部署。

當然 Render 也會有 Log 可以查詢，一旦部署失敗的話就可以去查詢程式碼部分是否有誤，進行修正後重新部署即可。當然未來 Line Bot 服務有任何的狀況也可以從後台來查詢。


後記
---

最近工作的關係，針對 Line Bot 的開發有比較深入的思考。Line Bot 本身的程式邏輯相對來說簡單很多，反倒是服務本身一旦涉及商品化，所衍生的來的問題才是最難克服的部分。

舉例來說，一旦要將 Line Bot 商品化，Line Bot 所目標部署的 Server，便需要根據每間公司的資安、商業考量來進行選擇，這中間也有使用者權限的問題。

Anyway，在不涉及商業化的前提下，我覺得開發 Line Bot 本身是非常有趣的一個開發專案。