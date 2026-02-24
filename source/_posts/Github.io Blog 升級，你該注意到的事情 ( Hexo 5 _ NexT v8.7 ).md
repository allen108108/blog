---
title: Github.io Blog 升級，你該注意到的事情 ( Hexo 5 / NexT v8.7 )
date: 2021-09-01 16:18:45
categories:
- 前端開發 Frontend Development
image: https://i.imgur.com/lQ97x53.jpg
mathjax: true
---


前言
---

自從開始工作以來，專案及案件如雪片般飛來 (?)，導致部落格的更新幾乎呈現停擺的狀態，在這一年中，幾乎都著重在前端開發 ( Vue.js )、API串接 ( PHP )，從原本一無所知的領域，現在也總算稍微做出點成果，這一年中邊做邊學習，也累積了不少經驗，一直想要花點時間記錄下來，想歸想，也一直拖到了現在。

但正當我寫完今年的第一篇文章後才發現，我自己電腦中的 Node JS 已經被我拿來處理工作上的事情，版本已經與之前撰寫部落格時不同，既然都已走到這步田地，乾脆狠下心來花一些時間來做一個部落格的大升級，至少，這樣的狀態可以不用一直擔心版本的問題，也可以感受一下新版本帶來的差異性。

<!-- more -->


NVM for windows
---

首先，為了避免這次的蠢事再度發生，筆者安裝了 NVM ( Node JS Version Manager )，這是一個專門為了 Node JS 而生的版本管理器，利用簡單的指令，可以迅速切換各種不同 Node JS 版本n的環境。

安裝方式非常簡單，基本上就是一鍵安裝。首先至 nvm-windows 的 [release](https://github.com/coreybutler/nvm-windows/releases) 中下載最新版本的 nvm，找到你想要下載的版本就下載其安裝檔直接安裝，過程筆者就不多贅述，反正就是 next / continue / OK 一路按下去就好了。

安裝完成後，可以打開 cmd 輸入 `nvm` 即會顯示版本號，以及相關指令說明。

![](https://i.imgur.com/EOTpkQp.png)

這裡筆者簡單說明幾個最常使用的指令 : 

`nvm ls available` : 顯示可以下載的 Node JS 版本
`nvm ls` : 顯示已經安裝的 Node JS 版本
`nvm install <版本號>` : 下載 Node JS 指定版本，例如 `nvm install 14.17.5`
`nvm use <版本號>` : 直接切換至需要的版本，例如 `nvm use 14.17.5`

基本上簡單的使用，這四個指令就可以打天下了，其他的細節可以參考 [官網](https://github.com/coreybutler/nvm-windows) 。

筆者撰寫本文時，`nvm` 的最新版本為 `v16.8.0`，筆者的確也嘗試過，以最新版本進行安裝，但會有一些問題存在，因此筆者以自身的版本搭配作為本文的建議配置，至少可以確認這樣的配置是可以進行的。

![](https://i.imgur.com/idKv60Q.png)

`NodeJS v14.17.5`，此為 LTS 的最高版本。
`Hexo 5.4.0`、`NexT v8.7.0`，均為筆者遇到當下的最新版本。
`Pandoc v2.14.2`、`hexo-renderer-pandoc v0.3.0`，為了數學公式所安裝的套件。

最主要是這些部分，大致上就可以應付我自己個人技術部落格的需求，也可以應付大多數的數學公式呈現。以下是最後筆者的 `package.json` 內容，供讀者們參考 : 

```
  "dependencies": {
    "hexo": "^5.0.0",
    "hexo-deployer-git": "^3.0.0",
    "hexo-generator-archive": "^1.0.0",
    "hexo-generator-category": "^1.0.0",
    "hexo-generator-index": "^2.0.0",
    "hexo-generator-tag": "^1.0.0",
    "hexo-renderer-ejs": "^1.0.0",
    "hexo-renderer-pandoc": "^0.3.0",
    "hexo-renderer-stylus": "^2.0.0",
    "hexo-server": "^2.0.0",
    "hexo-theme-landscape": "^0.0.3",
    "hexo-theme-next": "^8.7.0"
  }
```

Node JS v14.17.5
---

因為已經確認版本，我們便可以利用指令 `nvm install 14.17.5` 直接進行安裝，安裝完成後再利用 `nvm use 14.17.5` 來使用該環境。最後可以使用 `node -v` 來確認版本是否順利安裝。

Node JS 的環境安裝，因為有了 `nvm` 的輔助變得簡單很多，隨時可以進行環境切換，當然我們也能嘗試各種不同的版本搭配，若讀者有興趣，可以嘗試看看最新版本的 Node JS 跟 Hexo 5 的搭配是否也可以運作。

Hexo 5
---

在進行 Hexo 5 的更新前，有一件非常重要的事情，**如果你的部落格是舊版本要升級成新版本，請務必要記得，不要直接在原本的部落格上進行更新，有非常高的機會會更新失敗。** 在此，筆者的建議是先創建一個新的部落格資料夾，並且建立一篇新文章，確認該文章可以呈現後，再進行後續的搬移動作。

首先，在 cmd 中輸入 `npm install -g hexo-cli` ，這個步驟同時會進行 `hexo-cli` 與 `hexo` 的安裝，安裝完成後，可以利用 `hexo -v` 確認安裝版本。

![](https://i.imgur.com/6Per7Of.png)

先初始化一個新的部落格資料夾，這邊假設取名為 `hexoBlog`，進入此資料夾中進行初始化的動作，指令`hexo init hexoBlog` 之後便會在目錄中增加一個名為 `hexoBlog` 的資料夾。

進入此資料夾後，輸入指令 `npm install` 便會將所需要的套件進行完整安裝。安裝完成後，基本的新版 Hexo 已經安裝完成，在其中讀者們可以發現 `/hexoBlog/_config.yml`、`/hexoBlog/_config.landscape.yml` 兩個檔案，`_config.yml` 為部落格的 site config ( 站台設定檔 )，而 `_config.landscape.yml` 則為部落格的 theme config ( 主題設定檔 )，此處預設主題為 landscape，但因為我們接下來要換成 NexT，所以在此處筆者會先改名為 `_config.next.yml`。

NexT v8.7
---

**安裝 NexT 主題的方式有兩種，請擇一進行，同時安裝兩種方式可能會產生衝突。**

安裝方式一 : npm 安裝，此方式會將主題套件安裝在 `hexoBlog/node_modules/hexo-theme-next` 中
```
npm install hexo-theme-next
```

安裝方式二 : git 安裝，這種方式則會將主題安裝在 `hexoBlog/themes/next` 中

```
git clone https://github.com/next-theme/hexo-theme-next themes/next
```

若讀者發現自己的主題兩者均有安裝，建議移除其中一種，筆者舊版使用的是第二種方式，這次則選擇利用 npm 安裝，將 `hexoBlog/themes/next` 資料夾整個刪除，以免衝突。

安裝完成後，先將主題資料夾 ( 本文例 : `hexoBlog/node_modules/hexo-theme-next` ) 內容整個覆蓋至 `hexoBlog/_config.next.yml` 檔案中。安裝步驟到了此階段，幾乎已經完成了近八成。接下來要做的就是逐步地將舊的部落格設定移至 `hexoBlog`，這裡的轉移不能直接將設定檔覆蓋，因為新舊版本的設定檔內容並不完全相同，有些設定項目並不通用，因此建議的做法是利用程式碼對照工具，將舊版的設定檔內容，逐一在新版設定檔內容作修改。

![](https://i.imgur.com/Zi4x9RU.png)

如上圖，筆者習慣利用 VSCode 的比較功能進行程式碼的比對，當然坊間有許多工具都可以使用，端看讀者習慣來搭配使用。新舊版本的設定項目互有增減，所以建議將重要的部分進行移轉後，再來調整一些細部項目。

調整完站台設定檔 `hexoBlog/_config.yml` 及主題設定檔 `hexoBlog/_config.next.yml` 後，筆者開始會將舊文章進行分批搬移，每一次的搬移都會利用 `hexo server` 做一下確認，因為筆者的文章會有大量的數學公式，因此必須安裝對應的渲染器進行處理。

在新版的 NexT 中，支援的數學公式渲染引擎有 `MathJax` 及 `KaTeX` 兩種，各自需要安裝的套件也不相同，筆者這邊使用的是以 `MathJax` 渲染的方式，安裝 `hexo-renderer-pandoc` 來進行數學公式的渲染。若讀者想要更全面了解其他種方式可以參考 : [让 Hexo Next (v8.0.0) 支持 LaTeX 数学公式](https://dog.wtf/tech/making-hexo-next-theme-latex-math-equation-supported/) 一文。


Pandoc
---

在新版的 NexT 中，有提及以前使用的 `hexo-renderer-kramed` 已經停止維護，因此建議改用 `hexo-renderer-pandoc` 來渲染數學公式。然而在進行 `hexo-renderer-pandoc` 安裝前，幾個重要的關鍵提供讀者們注意 : 
* `hexo-renderer-pandoc` 是基於 Pandoc 的衍生套件，因此必須先確定 Pandoc 有安裝。
* 確認環境中沒有其他的數學公式渲染套件，若有的話務必進行刪除，例如 : `hexo-renderer-kramed`、`hexo-renderer-marked`、.....

Pandoc 的安裝跟 nvm 類似，也是屬一鍵安裝，先至 Pandoc 官網進行最新版本[下載](https://pandoc.org/installing.html)，或是至 [release](https://pandoc.org/releases.html#revision-history-for-pandoc) 尋找特定版本下載。

安裝完成之後便可以進行 `hexo-renderer-pandoc` 的安裝 : 

```
npm install hexo-renderer-pandoc --save
```

安裝過後應該要可以正常顯示嗎 ? 其實正常應該會有一些錯誤發生。



### `Error expected variable end`

這個錯誤訊息在之前的文章中亦有提到，主要是 LaTex 語法放進 Hexo 時會有渲染上面的問題，無法針對 `{{` 或 `}}` 這種連續大括號進行處理，過去的方式是在 `hexoBlog/node_modules/nunjucks/src/lexer.js` 中將 : 

```
var VARIABLE_START = '{{';
var VARIABLE_END = '}}';
```

修改為

```
var VARIABLE_START = '{$';
var VARIABLE_END = '$}';
```

然而，這樣的方式在這次真的版本中完全行不通。

最主要的原因是，新版的 Hexo 中更改了渲染方式，替換掉 `swig` 檔而以 `njk` 檔案取代，而在 `njk` 中便會用大量的雙大括號進行渲染，如果我們依照過去的方式會使整個站台無法渲染成功。

因此，若文章中有大量雙大括號時，建議將雙大括號之間留空白，也就是說 `{{` 改為 `{ {` 且 `}}` 改為 `} }`

如此一來，便可以正常顯示所有的數學符號。

個人化設定
---

接著我們就可以針對網站的美觀度進行個人化的調整。

舊版的 Blog 中，有些 CSS 設定我仍然想要保留，希望這個 Blog 站台可以保留一些個人的特色在其中，在 NexT 8.0.0 版本後，依舊保留了個人化的選項，並且可以統一將版面的 CSS 或樣式調整統整在一處進行處理 ( 推薦 : [Hexo-NexT 版本更新记录](https://tding.top/archives/2bd6d82.html) ) ，但筆者希望在極短時間內進行搬移，因此仍然保留舊版的調整方式。經筆者測試過後，舊版的方式仍然可以調整 CSS 設定，因此便遵從之前的文章進行調整 ( 詳見 : [使用 Github page 製作個人網站 -- 以 Hexo / Next 製作部落格](https://allen108108.github.io/blog/2019/10/07/%E4%BD%BF%E7%94%A8%20Github%20page%20%E8%A3%BD%E4%BD%9C%E5%80%8B%E4%BA%BA%E7%B6%B2%E7%AB%99%20%E2%80%93%20%E4%BB%A5%20Hexo%20_%20Next%20%E8%A3%BD%E4%BD%9C%E9%83%A8%E8%90%BD%E6%A0%BC/) )

基本上，所有的調整我們一樣都可以利用 `hexo server` 在本機端先進行檢查，畢竟舊版的部落格還在，得先確定更新版本沒有問題在部屬上去做更換。


最後，站台設定檔 `hexoBlog/_config.yml` 最後的 `deploy` 項目務必要正確填寫，這個項目完成後，基本上部屬的工作也正式轉移，之後便可以捨棄舊版，利用我們的更新版本直接 push 到原本的部落格站台進行更新。


Personal Access Token
---

但是事情總是沒有這麼順利，因為太久沒更新部落格，在進行部屬的時候才發現，github 已經不是我們認識的那個 github 了.....

```
remote: Support for password authentication was removed on August 13, 2021.
Please use a personal access token instead. remote: 
Please see https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/ for more information. 
fatal: unable to access "..." : The requested URL returned error: 403
```

簡單來說，就是 github 原本是用個人的登入密碼作為憑證進行專案的更新，但在 2021/08/13 便不再支援這樣的密碼憑證，取而代之的，我們必須取得 personal access token 來進行憑證更新。

步驟簡易說明如下 : 

1. 前往 Github 網站並登入。
2. `Settings` > `Developer settings` >`Personal Access Tokens` > `Generate New Token` 進行  personal access token 的新增。
3. 填入 token 的用途、期限以及權限設定後，按下 `Generate token` 便會產出一個新的 token，此時請複製下來，否則離開就不會再出現。
4. 至 Windows 控制列搜尋 `認證管理員`，選擇 `Windows 認證`。
5. 在一般憑證下尋找 `git:https://github.com`，若無，則進行新增一般憑證，填入 Github 帳號名稱，密碼則以前面複製下來的 token 值進行取代後按下儲存。

經過了上述的種種步驟，讀者們應該也可以順利的更新部落格了。


結論
---

太久沒有更新，加上 Node JS 版本管理的失當，導致筆者花了好幾天的時間尋找 Solution，幸虧還可以順利更新，否則前面一百來篇的文章就真的必須要段捨離了。

這件事情讓我們知道，環境的版本管理非常重要，在進行重要專案的規劃及部屬，必須要將各框架、套件的版本都確認下來，最好是安裝檔都留有備份，才能以備不時之需。再不然，就必須要利用各種版本管理工具來協助我們進行專案的開發。

最後，祝福各位讀者，可以順利更新自己的部落格內容囉 !


