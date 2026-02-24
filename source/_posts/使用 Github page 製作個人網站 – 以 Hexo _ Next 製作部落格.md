---
title: 使用 Github page 製作個人網站 -- 以 Hexo / Next 製作部落格
date: 2019-10-07 03:27:11
categories:
- 前端開發 Frontend Development
image: https://i.imgur.com/lQ97x53.jpg
mathjax: true
---


以 Wordpress 作為部落格的主要平台也已經大半年了，文章也已經到了 70+ 的數量，但 Wordpress 商業化的實在太嚴重，許多為人稱羨的外掛都必須要升級為商務版才能夠使用，每個月要六、七百元的花費實在是太誇張，畢竟不是每一個人都是專業收費寫手。
<!-- more -->
沒有了外掛的 Wordpress 就完全不是 Wordpress 了。

最近剛好碰到身邊有人在做 github page，就來了解看看這樣的方式是否能達到自己想要呈現的方式。事實上在寫這篇文章的時候，我已經做完一個 Blog 了，但在過程中實在碰到太多的坑，每一次都讓人想直接放棄...，那乾脆來寫一篇文統整一下整個建構的過程吧。

## 部落格需求


要建構一個自己期待的部落格，首先必須要釐清自己對於部落格 ( 網站 ) 的要求，而我的要求大概是如此 : 

1. 排版不要太過鬆散。很多人使用的 Medium 對我來說就是比較鬆散的排版方式，我自己看起來的確是不太習慣。
2. 可以直接使用 Markdown 編輯。我還是都會先用 HackMD 作為編輯平台，再發布到部落格上，因此最好是可以直接沿用其格式，不用進行太多改動。
3. 支援 $\LaTeX$。對我來說，這是最最重要的一點，我的筆記內涵蓋太多數學式，若無法支援 $\LaTeX$，改用圖片方式呈現會讓整個頁面變得很亂，且無法行內插入數學式也會導致整個文章的流暢性會銳減。

從幾個使用 Github page 的部落格來看，是蠻有可能達到我期望的效果，那我們就開始來進行吧 !


## Github repo 設定 & Hexo 搭建


設置 Github page 專案一般有兩個方式 : 
1. 以 `<user_name>.github.io` 為專案名稱，利用 master branch 建構，這樣生成的網址將會是 `https://<user_name>.github.io`
2. 以自訂專案名稱 `<repo_name>` 利用 gh-pages banch 建構，，這樣生成的網址將會是 `https://<user_name>.github.io/<repo_name>`

我自己想要做的是一個比較完整個個人網站，因此想保留 `https://<user_name>.github.io` 作為首頁網址，在這邊我就使用第二種方式來做部落格部分的建構。

### STEP 1 : 新增一個 github repository

在個人的 github 頁面上新增一個 repository (此處我就使用 blog 作為專案名稱)，不需建立 README.md

![](https://i.imgur.com/70P8ltx.png)

### STEP 2 : 安裝 Node.js 套件管理工具

還有很多重要的事情要解說，請原諒我，接下來的部分我就簡單介紹一下，
中間可能會有一些坑，上網找一下應該都會有很多解決方式。
(EX: git 過程中 Permission denied (publickey) problem...)

Hexo 是一個用 Node.js 所寫成的部落格框架，因此在安裝前必須先確定我們是否有 Node.js 套件管理工具 npm ( 本文使用 ) 或 yarn。

我們可以從 [Node.js 官網](https://nodejs.org/en/?source=post_page-----317beefdf182----------------------) 下載安裝，安裝完後至終端機輸入

```
$ npm -v
```
輸出版本後便表示安裝已完成


### STEP 3 : 安裝 Hexo

```
$ npm install -g hexo-cli
# 安裝 hexo

$ hexo init <repo_name>
# 初始化安裝 hexo 的資料夾

$ git clone https://github.com/<user_name>/<repo_name>.git
# 將安裝 hexo 資料夾內的檔案全部複製到這裡
```

這樣我們就可以從這邊修改部落格內容，並且推上 github 專案中。

這裡有幾個常用指令要記住，之後再進行˙部落格修改的時候會很常使用到 : 

```
$ hexo clean
# 清除舊 publish設定

$ hexo g
# 建立新 publish

$ hexo deploy
# deploy 到伺服器

$ hexo server
# 在本地瀏覽網站內容 (預設 http://localhost:4000)
```

### STEP 4 : 安裝 Theme ( Next )

Next 大概是目前 Hexo 主題中最受歡迎的一個，事實上仍有許多不同的主題深受各取向部落客喜歡，大家可以先看過一輪之後再做決定，不然改到後來實在真的會懶得重來一次。

```
$ cd <repo_name>
# 切換到要部屬至 github 的資料夾路徑

$ git clone https://github.com/iissnan/hexo-theme-next themes/next
# clone Next 主題專案到站點裡的themes資料夾中
```
這樣並不算完成，接下來要到設定檔中將主題改為 next ( 預設是 landscape )

## 基本設定


我們先做一些網站的基本設定，主要的設定檔有兩個

```
\_config.yml
\themes\next\_config.yml
```

### `\_config.yml`

網站的基本資訊設置，比較要注意的地方是 `language` 的部分，之後要配合 Next 的語言統一做設定。
```
# Site
title: #網站/部落格名稱
subtitle: #網站/部落格副標題
description: #網站/部落格基本描述 
keywords:
author: #作者
language: #語言 zh-TW
timezone: #時區 Asia/Taipei
```

設定文章網址型態，這裡主要是讓某些地方會需要顯示文章網址 (例如版權宣告區塊)的部分可以正確顯示。
```
# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: https://<user_name>.github.io/<repo_name>
root: /<repo_name>/
permalink: :year/:month/:day/:title/
permalink_defaults:
```

將主題改為 next

```
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next
```

最後這裡最重要，之後我們要利用 hexo 指令進行部屬，就會調用這部份來進行部屬，如果這部分沒有正確便無法部屬上去。

```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: https://github.com/<user_name>/<repo_name>.git
  branch: gh-pages
```
如果我們並不是利用 gh-pages 來進行網站設置，那麼 branch 的部分就要改成 master。


### `\themes\next\_config.yml`

版權頁設定，這裡可以設定文末 (或 sidebar) 版權宣告的型態，這裡我選用的是 `by-nc-sa` ( 姓名標示-非商業性-相同方式分享 )。

![](https://i.imgur.com/S1n6Red.png)

`language` 部分則是設定 [cc 版權說明網頁](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh_TW)的語言內容，剛好台灣有人處理這部分，所以有繁體中文可以使用。

```
# Creative Commons 4.0 International License.
# See: https://creativecommons.org/share-your-work/licensing-types-examples
# Available values of license: by | by-nc | by-nc-nd | by-nc-sa | by-nd | by-sa | zero
# You can set a language value if you prefer a translated version of CC license, e.g. deed.zh
# CC licenses are available in 39 languages, you can find the specific and correct abbreviation you need on https://creativecommons.org
creative_commons:
  license: by-nc-sa
  sidebar: false
  post: true
  language: deed.zh_TW
```


Next 一共有四種版型可以選擇，大家可以自行選擇版型，將不要用的版型用 `#` 註釋掉即可。( 我自己使用的是 Gemini 版型 )
```
# Schemes
#scheme: Muse
#scheme: Mist
#scheme: Pisces
scheme: Gemini
```

側邊 menu 區塊設定，我自己文章不會使用標籤tag，所以只留分類categories部分。之後如果要使用 SEO 優化，還會開啟 sitemap 的部分。

```
menu:
  home: / || home
  #about: /about/ || user
  #tags: /tags/ || tags
  categories: /categories/ || th
  archives: /archives/ || archive
  #schedule: /schedule/ || calendar
  #sitemap: /sitemap.xml || sitemap
  #commonweal: /404/ || heartbeat
```


大頭照設定，url 部分可以給圖片網址，也可以給資料路徑。
```
avatar:
  # In theme directory (source/images): /images/avatar.gif
  # In site directory (source/uploads): /uploads/avatar.gif
  # You can also use other linking images.
  url: /images/圖片.jpg
  # If true, the avatar would be dispalyed in circle.
  rounded: true
  # If true, the avatar would be rotated with the cursor.
  rotated: false
```

sidebar 部分可以顯示我們自己想提供給讀者的社交平台連結，底下這段便是要進行設定。
第一行是我自己加進去的，因為我需要從部落格連回去個人主頁的部分，就利用這裡來進行連結設定。

`||` 後面後面是進行圖示的設定，可以從 [Font Awesome](https://fontawesome.com/icons?d=gallery) 進行選擇，從網站上將圖示名稱加進去即可。

![](https://i.imgur.com/CqCLW3R.png)


```
social:
  #Github page: https://username.github.io/ || github-alt  
  #GitHub: https://github.com/username || github
  #E-Mail: mailto:username@mail.com || envelope
  #Google: https://plus.google.com/yourname || google
  #Twitter: https://twitter.com/yourname || twitter
  #FB Page: https://www.facebook.com/username|| facebook
  #Wordpress: https://mathpy655905385.wordpress.com/ || wordpress
  #VK Group: https://vk.com/yourname || vk
  #StackOverflow: https://stackoverflow.com/yourname || stack-overflow
  #YouTube: https://youtube.com/yourname || youtube
  #Instagram: https://instagram.com/yourname || instagram
  #Skype: skype:yourname?call|chat || skype
```

開啟瀏覽人次設定。`busuanzi_count` 在比較新版的 Next 主題內都已經包含在其中了，不需另外安裝或是從 css 部分調整原始碼，我們只要在 `enable` 的設定上改為 true 即可。

```
busuanzi_count:
  enable: true
  total_visitors: true
  total_visitors_icon: user
  total_views: true
  total_views_icon: eye
  post_views: true
  post_views_icon: eye
```
其實裡面還有幾個可以調整的地方，但由於跟接下來我們要解說的部分有關，就留待各部分分開解釋。

## 語言設定


在 Hexo 的語言設定中，幾乎都是用參照的方式來去對整個網站中的欄位做語言上的轉換。

在站點的設定檔 (\_config.yml) 中語言部份設定為 zh-TW，那麼我們就必須要有一個 zh-TW.yml 檔，裡面就是針對每一個欄位名稱給予其繁體中文翻譯。

這樣的檔案位在主題 next 中的 language 資料夾中 ( \themes\next\languages )，如果裡面沒有 zh-TW.yml 檔案，那我們就必須自己寫一個檔案放在裡面。

檔案內容如下 : 
```
---
title:
  archive: 歸檔
  category: 分類
  tag: 標籤
  schedule: 時間表
menu:
  home: 首頁
  archives: 歸檔
  categories: 分類
  tags: 標籤
  about: 關於
  search: 搜尋
  schedule: 時間表
  sitemap: 網站地圖
  commonweal: 公益 404
sidebar:
  overview: 本站概要
  toc: 文章目錄
post:
  posted: 發表於
  edited: 更新於
  created: 創建時間
  modified: 修改時間
  edit: 編輯
  in: 分類於
  more: 更多
  read_more: 閱讀全文
  untitled: 未命名
  sticky: 置頂
  views: 閱讀次數
  related_posts: 相關文章
  copy_button: 複製
  copy_success: 複製成功
  copy_failure: 複製失敗
  copyright:
    author: 作者
    link: 文章連結
    license_title: 版權聲明
    license_content: "本網誌所有文章除特別聲明外，均採用 %s 許可協議。轉載請註明出處！"
page:
  totally: 共有
  tags: 標籤
footer:
  powered: "由 %s 強力驅動"
  theme: 主題
  total_views: 總瀏覽次數
  total_visitors: 訪客總數
counter:
  tag_cloud:
    zero: 沒有標籤
    one: 目前共有 1 個標籤
    other: "目前共有 %d 個標籤"
  categories:
    zero: 沒有分類
    one: 目前共有 1 個分類
    other: "目前共有 %d 個分類"
  archive_posts:
    zero: 沒有文章。
    one: 目前共有 1 篇文章。
    other: "目前共有 %d 篇文章。"
state:
  posts: 文章
  pages: 頁面
  tags: 標籤
  categories: 分類
search:
  placeholder: 搜尋...
cheers:
  um: 嗯..
  ok: 還行
  nice: 好
  good: 很好
  great: 非常好
  excellent: 太棒了
keep_on: 繼續努力。
symbol:
  comma: "，"
  period: "。"
  colon: "："
reward:
  donate: 捐贈
  wechatpay: 微信支付
  alipay: 支付寶
  bitcoin: 比特幣
accessibility:
  nav_toggle: 切換導航欄
  prev_page: 上一頁
  next_page: 下一頁
symbols_count_time:
  count: 文章字數
  count_total: 總字數
  time: 所需閱讀時間
  time_total: 所需總閱讀時間
  time_minutes: 分鐘
```

這樣的方式應該可以用在各種主題上面，如果今天我們有看到一個主題想要使用，但卻沒有繁體中文，應該可以使用這樣的方式來做更改。( 但我不確定這樣會不會有侵權疑慮XD )

## $\LaTeX$ renderer


在 hexo 中，預設的 markdown renderer 是 `hexo-renderer-marked`，但這樣的 renderer 並無法使用 $\LaTeX$ 語法，因此要將 renderer 替換成 `hexo-renderer-kramed`

```
$ npm uninstall hexo-renderer-marked --save
$ npm install hexo-renderer-kramed --save
```
但即使 renderer 更換以後仍然有一些地方會在行內插入 $\LaTeX$ 時造成衝突，我們要對這部分進行程式碼更改 ( `\node_modules\kramed\lib\rules\inline.js` )


找到下面兩列，進行相對應的更改
```
//escape: /^\\([\\`*{}\[\]()#$+\-.!_>])/,
escape: /^\\([`*\[\]()#$+\-.!_>])/,
```

```
//em: /^\b_((?:__|[\s\S])+?)_\b|^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
em: /^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
```

正常狀況下，這樣就能開始使用 $\LaTeX$ 了，但是當我實際上在使用時，還是會遇到報錯的狀況 : 

```
Template render error: (unknown path) [Line 37, Column 81]
  expected variable end
```

由於我們使用 $\LaTeX$ 時，會有很多的符號可能會相連著使用，這導致會與 hexo 裡面的一些模板引擎 ( nunjucks ) 相衝突，所以我們要做相應的更改 ( `\node_modules\nunjucks\src\lexer.js` ) : 

```
var VARIABLE_START = '{$';
var VARIABLE_END = '$}';
```
如此修改過後，截至目前為止使用就沒有太大的問題了。


## 色彩配置 ( 背景色、文章方塊顏色、側邊欄顏色、文字顏色、標題欄顏色 )


這幾個部分的顏色必須相互做搭配，不然整體的配色會變得非常奇怪，所以這邊我一起來解釋。

首先背景顏色是整個部落格映入眼簾最大範圍的部分，也可以想成是整個部落格主題的基底，因此要選擇白色底還是深色底都會牽動接下來其他顏色的配置。

這個部分的更動要在 `\themes\next\source\css\_variables\<你選擇的版型>.styl` 中更動，我選擇以深藍色 ( #1d1d30 ) 為底

這裡的部分要使用 Hex Code 還是 RGB Code 都可以。

```
// Settings for some of the most global styles.
// --------------------------------------------------
$body-bg-color           = #1d1d30;
```

接下來我針對文章方塊、側邊欄 ( 包含 menu 跟 sidebar ) 做統一設定，不然顏色通通不一樣會太花，風格容易跑掉。

統一都設定成 `rgba(0, 153, 153, 0.1)` ，這裡的 0.1 指的是透明度。

`\themes\next\source\css\_schemes\Gemini\index.styl` 中設定文章方塊顏色
```
// Post & Comments blocks.
.post-block {
  background: rgba(0, 153, 153, 0.1);
  border-radius: $border-radius-inner;
  box-shadow: $box-shadow-inner;
  padding: $content-desktop-padding;
}
```
`\themes\next\source\css\_schemes\Pisces\_layout.styl` 設定 menu 區塊顏色

```
.header-inner {
  background: rgba(0, 153, 153, 0.1);
  border-radius: $border-radius-inner;
  box-shadow: $box-shadow-inner;
  overflow: hidden;
  padding: 0;
  position: absolute;
  top: 0;
  width: $sidebar-desktop;

  +desktop-large() {
    .container & {
      width: $sidebar-desktop;
    }
  }
```
`\themes\next\source\css\_schemes\Pisces\_sidebar.styl` 設定側邊欄顏色
```
.sidebar-inner {
  background: rgba(0, 153, 153, 0.1);
  border-radius: $border-radius;
  box-shadow: $box-shadow;
  // padding: 20px 10px 0;
  box-sizing: border-box;
  color: $text-color;
  width: $sidebar-desktop;

  if (hexo-config('motion.enable') && hexo-config('motion.transition.sidebar')) {
    opacity: 0;
  }
```

接下來我們做文字顏色的設定 `\themes\next\source\css\_variables\base.styl`
這一個區塊決定了所有文字或分隔線等等的顏色定義，在整個 next 主題裡面大概都會從這裡來抓顏色。

所以大家可以嘗試著更動每一個部分，你就會知道那些顏色對應到那些文字區塊。
這裡的部分我已經改動非常多了，所以文字跟色碼基本上對不太上 XDDD

```
// Colors
// colors for use across theme.
// --------------------------------------------------
$whitesmoke   = #f5f5f5;
$gainsboro    = #000d33;
$gray-lighter = #ddd;
$grey-light   = #eee;
$grey         = #bbb;
$grey-dark    = #999;
$grey-dim     = #eee;
$black-light  = #bbb; 
$black-dim    = #333;
$black-deep   = #eee;
$red          = #ff2a2a;
$blue-bright  = #87daff;
$blue         = #0684bd;
$blue-deep    = #262a30;
$orange       = #fc6423;
```
當然，有些顏色會有很多區塊共用，當你在這裡做更改時，很有可能一次改動很多區塊的顏色，這部分實在很繁雜，我會建議先從這裡做更改，真的必須要切割不同區塊分開處理再個別到該區塊去做處理。

最後，標題欄位的顏色從 `\themes\next\source\css\_schemes\<你選擇的版型>\index.styl` 來做更改

更改的方式只要在最後加上這段即可

```
.site-meta {
  background: #660033; //我使用酒紅色來作為標題欄的背景色
```

我自己部落格的整體顏色配置大概是像這樣 ( 有一種延禧攻略的配色XD )

![](https://i.imgur.com/lQ97x53.jpg)


## 字體大小


字體大小的更改直接到 `\themes\next\source\css\_variables\<所選擇的版型>.styl` 的最後加入

```
// 部落格文字的大小
$font-size-base = 15px

// Code字體的大小
$code-font-size = 13px
```

## 版面比例


設置到這裡，大概會完成個雛型了，這邊我們要調整一下整個版面的比例配置。
在預設的狀況，左右留白會很多，以電腦桌面版本來看，總覺得留白太多。要改變這樣的比例我們可以在 `\themes\next\source\css\_variables\<所選擇的版型>.styl` 最後加入

```
$content-desktop         = 80%
$content-desktop-large   = 80%
$content-desktop-largest = 80%
```
比例可自行調整看看哪一種比較適合自己

## 動態背景


Next 使用者大部分都使用 canvas-nest 這套動態背景，我也不例外，但要注意的是動態背景其實會吃掉一部分資源，當我瀏覽器停在部落格一段時間後，會很明顯聽到筆電風扇狂轉的聲音 @.@，這部分大家可以斟酌使用。

使用方法如下

```
先切換到 next 路徑底下

$ git clone https://github.com/theme-next/theme-next-canvas-nest source/lib/canvas-nest
```

之後可以到 next 設定檔 `\themes\next\_config.yml` 中設置

```
# Canvas-nest
# Dependencies: https://github.com/theme-next/theme-next-canvas-nest
# For more information: https://github.com/hustcc/canvas-nest.js
canvas_nest:
  enable: true #開啟或關閉
  onmobile: false # Display on mobile or not 是否在手機上顯示
  color: "255, 80, 219" # RGB values, use `,` to separate #線條顏色
  opacity: 1 # The opacity of line: 0~1 #線條透明度
  zIndex: -1 # z-index property of the background #位在哪一層
  count: 100 # The number of lines #線條數量
```

## 調整及部屬


在每一個設定更動後，我習慣會做以下動作

```
$ hexo clean
$ hexo g
$ hexo server
```
先在本機上看看是否有什麼問題，持續地進行微調，待最後設定都沒有問題後再進行部屬

```
$ hexo deploy
```
## 文章發布

### Markdown 編輯器
由於 Hexo 支援 markdown 編輯，因此我強烈建議大家使用 VS Code 來做為 markdown 編輯器，原因有幾個 : 
1. 高亮設計，在編輯上會輕鬆很多
2. 可利用 `Markdown Shortcuts`、`Markdown Preview Github Styling` 套件協助編輯
3. `Insert Date String` 套件可以快速加上時間

總歸來說就是 VS Code 有很多好用的套件可以讓文章編輯變得更簡單。

### `Front-matter`

在文章編輯上，每一篇空白 md 檔開頭加上 `Front-matter` 來指定每一篇文章的參數，以下是我自己常用的部分，還有更多參數可以設定，詳情可參閱 [Hexo 中文官方文檔](https://hexo.io/zh-tw/docs/front-matter)

```
---
title: 文章標題 
date: 文章發布時間
categories:
- <category>
image: 首頁呈現的圖片網址
mathjax: true 開啟 LaTeX
---
```

### 文章發布時間

若有使用 VS Code 的 `Insert Date String` 套件，可直接 `Ctrl+Shift+i` 直接插入當前時間。

### 文章分類

在我們的 menu 欄位有一個分類頁面，可利用以下方式設置

```
切換到站點路徑下
$ hexo new page categories #建立分類頁面
```

之後每一篇文章在 `Front-matter` 中設置 `categories` 就會在分類頁面中呈現。

![](https://i.imgur.com/fP8QRop.png)


### 首頁摘要及圖片設置

一般在部落格首頁我們會希望每一篇文章只要呈現一小部分內容以及一張代表性圖片即可，藉由 " 閱讀全文 " 按鈕來閱讀每一篇文章完整內容，在 hexo 官方的建議作法是在文章中插入 `<!-- more -->` 便可以在首頁僅呈現此代碼之前的文章內容。

至於閱讀全文的按鈕樣式，我們可以在 `\themes\next\source\css\_schemes\<選擇的版型>\index.styl` 最後加入以下內容來做調整

```
//閱讀全文樣式設置
.post-button {
    margin-top: 30px;
    text-align: center;
}

.post-button .btn {
    color: white;
    font-size: 15px;
    background: #000d33;
    border-radius: 16px;
    line-height: 2;
    margin: 0 4px 8px 4px;
    padding: 0 20px;
    border:none;
    -webkit-box-shadow: 0px 5px 30px -3px rgba(0,0,0,0.75);
    -moz-box-shadow: 0px 5px 30px -3px rgba(0,0,0,0.75);
}

.post-button .btn:hover {
    color: #000d33;
    background: #ffffff;
}
```

至於圖片部分只要在 `Front-matter` 給定圖片網址，即可在首頁呈現此文章的代表圖片，且不會在文章全文中出現。

但由於圖片大小的不同，可能會導致整個首頁版面會稍嫌雜亂，我們可以在上面的檔案中最後加入以下內容來限制圖片寬度要相同

```
//摘要圖片大小設置 
img.img-topic {
    width: 100%;
}
```

## 後記

其實認真要玩起來，這裡面的坑還真不少，從完全沒有概念到最後弄到有點樣子也足足是花了三天時間在調整。顏色的配置應該會花最多時間調整，如果單純用原白底配色就會簡單很多，但常看電腦的我實在不是太喜歡白色 XDDD

但當這些設置都完成以後，要進行文章發布就會簡單很多，也希望這篇文章可以幫助大家填一些坑，讓建構部落格的過程中可以更順利。