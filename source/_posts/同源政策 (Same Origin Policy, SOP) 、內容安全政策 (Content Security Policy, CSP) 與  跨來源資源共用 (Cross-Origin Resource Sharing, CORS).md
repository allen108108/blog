---
title: "同源政策 (Same Origin Policy, SOP)、內容安全政策(Content Security Policy, CSP)與跨來源資源共用 (Cross-Origin Resource Sharing, CORS)"
date: 2023-04-27 07:50:06
categories:
- 前端開發 Frontend Development
image: https://i.imgur.com/3dvbi5F.png
mathjax: true

---

前言
---

這兩三年的 Web APP 前端開發經驗，讓我對前端的一些概念也逐漸地變得清晰，最近剛好有點時間，也想把遇到過的一些問題記錄下來。

有一些問題在本機端開發時沒有發生，一直到網站要部屬到 Server 的時候，就會發現出現了一連串的錯誤。這錯誤在本地端開發時完全沒有遇到。

為了究其原因，花了一些時間與同事們進行研究與處理，才將這個問題解決，也想趁著最近把這些問題整理起來。

<!-- more -->

Same Origin Policy (SOP)
---

同源政策  (Same Origin Policy, SOP)，是一種瀏覽器安全功能，在這樣的政策下，限制了不同來源的文件與 script 相互存取的機制。

簡單來說，當我的網站有同源政策的保護，外界就不能隨意存取我網站的任何資源，也可以避免有心人士針對網站的攻擊。當然，這也會同時限制我們去存取其他網站資源。

從資訊安全的角度來說固然是一項好的政策，但從開發者的角度來說，變多了這一層顧慮要去考量，這是一體兩面的事情。


Content Security Policy (CSP)
---

那麼，什麼是 Content Security Policy ( 簡稱 CSP ) ? 簡單來說，就是寫在你網站上的一個規則，主要是給瀏覽器看的。這個規則，告訴瀏覽器說本網站可以引用的圖片來源、Javascript 來源、....。有做這樣的限制，可以使網站避免一些外部的攻擊，為資訊安全加上一層保護。

### 語法 Syntax


在實作上，通常我們會在 html 網站的 head 部分，加上 CSP 的設定指令，瀏覽器便會在顯示網站時使用我們所要求的指令來進行顯示 ： 

```htmlembedded=
<!DOCTYPE html>
<html lang=en>
    <head>
        <meta http-equiv="Content-Security-Polict" content="<policy-directive>; <policy-directive>">
        <meta charset=utf-8>
        <meta http-equiv=X-UA-Compatible content="IE=edge">
        <meta name=viewport content="width=device-width,initial-scale=1">
    </head>
</html>

```

當然，也可以在 web.config 中進行設定 ：

```htmlembedded=
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <httpProtocol>
            <customHeaders>
                <remove name="Content-Security-Policy" />
                <add name="Content-Security-Policy" value="<policy-directive>; <policy-directive>" />
            </customHeaders>
        </httpProtocol>
        <httpErrors errorMode="Detailed" />
    </system.webServer>
</configuration>
```

其中 `<policy-directive>` 便是將我們想要設定的指令寫進去，每一個指令都包含了 `<directive>` 以及 `<value>`，而不同的指令之間用 `;` 來區隔。


### 指令 Directives


詳細的指令種類，讀者們可以從 [MDN Web Docs: Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy) 中查閱，在本文中僅簡單介紹幾個比較常使用到的指令。

#### default-src 
所有指令的預設值，當其他指令未進行任何設定時，會使用 `default-src ` 的設定，`-src` 指的來源，從這邊我們也可以知道，CSP 中，有很大一部分會針對網頁中的「來源」有較多的限制。

#### style-src / script-src
針對 stylesheets / JavaScript 的來源進行限制，一般來說，CSP 是禁止我們利用 inline 程式碼的，原因是因為很多的駭客也會利用這樣的方式進行網頁的入侵。

如果要開發就必須要加上設定值 `unsafe-inline` 才可確保 inline 的部分可以正常使用，但相對的也會容易有資安的問題。


#### img-src
針對 image 的來源進行限制。在網頁的設計上，我們也往往會使用到非常多的圖片來進行妝點或是增加訊息的吸睛度，但一旦使用外部圖片也容易會有資安的問題。




### 設定值 Values


指令的意義大家大致上了解了，接下來筆者就簡單介紹幾個比較常用的設定值。

#### `http://*.example.com`

允許使用以 example.com 的任何子網域為來源，這其實就是一個白名單的概念。

#### `self`

允許使用以相同的網域跟 port 為來源。

#### `unsafe-inline`

允許使用 inline 程式碼。

#### `none`

禁止任何的來源。


Cross-Origin Resource Sharing (CORS)
---

筆者在這幾年的開發過程中，會碰到 Cross-Origin Resource Sharing ( 簡稱 CORS )的狀況大約會是在網站開發過程中，若瀏覽器需要針對外部 API 進行 Request 的時候，就可能會遇到。

### Http Header

正規的做法應該是在 API 開發的時候，在 header 加上通行證，讓瀏覽器可以進行 API Request，常見的 http header 便是直接在 API header 中設定白名單

```
Access-Control-Allow-Origin: <http://web.com.tw>
```

若要無條件開放則把網址部分以 `*` 來取代即可。


### CSP ? CORS ?

上面提到了 CSP 以及 CORS，乍聽之下這兩者實在很相近，那這兩者在開發場景上到底有什麼不同呢 ?

簡單來說，當我們的網站要去存取別人的資源 ( 或是別人的網站要來存取我們的資源 )，那就會跟 CORS 政策相關，但倘若我們要載入別人的資源到我們的網站，這牽扯的部分就會是 CSP 政策。


### CORS 代理伺服器

要解決 CORS 的問題，過往有 CORS Anywhere 這樣的 CORS 代理伺服器的服務，因為 CORS 的源頭是由瀏覽器產生，而且有時候 API 伺服器因為較嚴謹的資安限制無法在API 上進行鬆綁，所以當瀏覽器要去跟 API 要資料的時候就會被阻擋而造成錯誤的發生。 

類似 CORS Anywhere 這樣的代理伺服器服務便是為了解決這樣的兩難狀況，瀏覽器將請求發送至 CORS 代理伺服器後再轉送至 API 伺服器，因為 CORS 代理伺服器並非瀏覽器，因此便能藉此避開 CORS 的限制。

但，隨著現在 Heroku 的免費方案於去年底終結，CORS Anywhere 這樣的服務也逐漸凋零，但前端開發不可能因此而停止，我們還是得找一個解決方法來處理 CORS。


### Cloudflare

Cloudflare 利用單一介面為大大小小的企業甚至於個人提供穩定且安全的網路服務，用較不準確的說法，或許我們可以將 Cloudflare 感覺就是服務功能在多一些的 Heroku，而我們會利用一個 JavaScript 程式碼部署在這邊，利用 Cloudflare 來作為 CORS 的代理伺服器。

首先註冊並登入 Cloudflare

![](https://i.imgur.com/lnksDeG.png)

* `服務名稱` : 原則上可以任取，最後會變成代理伺服器網址的一部分。
* `選擇啟動器` : 這部分只是系統會給你一個樣板讓我們便於部署我們的服務程式碼，不管選擇哪一個，最終還是依照我們放上去的 script 來執行。

建立服務後就會出現下列資訊，我們可以點擊快速編輯按鈕將我們的 Script 放入服務中

![](https://i.imgur.com/W7PIaaw.png)

我們接著利用 [cloudflare-cors-anywhere](https://github.com/andy922200/cloudflare-cors-anywhere/blob/master/index.js) 這個專案來進行服務的部署。

將專案中 `index.js` 檔中的所有內容複製貼上至下圖左側的欄位中

![](https://i.imgur.com/ZqqnciN.png)

之後按下 `儲存並部署` 的按鈕後，我們就成功的建立了一個 CORS 的代理伺服器服務。

回到 Dashboard 中，下圖紅框處就是我們代理伺服器的網址
![](https://i.imgur.com/PcCGU7z.png)

使用方式 : `<CORS代理伺服器網址>?<API網址>`  這樣就可以在開發過程中進行跨域的 API 存取了。


後記
---

前端網頁開發上，勢必會遇到 CSP/CORS 的狀況，若公司對於資安比較嚴格一點，建議在開發前先確定允許的設定有哪些再進行後續開發。不要像筆者一樣，開發了一個大型 Web App 後才發現公司伺服器規範與開發設計有出入，屆時要進行大規模的修正反而得不償失。