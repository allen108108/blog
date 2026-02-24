---
title: Gateway 在 IOT 中扮演的角色
date: 2020-09-06 17:36:59
categories:
- 後端開發 Backend Development
image: https://i.imgur.com/FTlugXY.png
mathjax: true
---

IOT 的應用在現今的社會上已經十分普遍，智慧家庭、智慧工廠、....等 IOT 應用已經多不勝數，然而其中最關鍵的部件便是 Gateway (閘道器)，一台小小的黑色盒子，除了接收感應器接收到的資料外，還能接收外部指令對設備進行調整，甚至，我們還能在其中佈署 AI 模型進行邊緣運算，直接進行模型的推論。

然而，大多數的人對於 Gateway 的運作幾乎不甚了解，本文將會著重在 B2B 智慧空調的角度上，依筆者最近工作上的接觸，來試圖解釋 Gateway 在 IOT 中扮演的角色是什麼。

<!-- more -->

物聯網 Internet of Things ( IOT )
---

物聯網這樣的概念已經在全世界流行了好幾年，實際上，第一個提出這個名詞的人大概已不可考，但物聯網概念早在 1980 年代就已經有類似的應用。物聯網，顧名思義，利用網路連線將物品進行連接，這樣的方式無論是在一般家庭抑或是工業應用上，都有非常優秀的應用，可以使得生活更加舒適便利，也可以讓工業產線效率上更加提升。

舉例來說，將 IOT 概念應用於家庭之中，我們可以藉由溫度的感知讓空調自動進行調節，藉由自然光線的明暗來調整室內光線；應用於工業之中，可以將產線上的機器進行連結，隨時掌控機器狀態、進行瑕疵檢測...等；應用於交通之上，便可以結合路口監視器來掌握車流，進行適當的交通號誌調節。

近幾年結合電腦視覺及深度學習的發展，將 AI 人工智慧結合 IOT 形成的 AIOT 更是炙手可熱，前一陣子中研院開發的 YOLOv4 在交通上的應用便是一個非常好的 AIOT 應用。

IOT 的整個生態系，我們可以大致上做下面的描繪 : 


<img width=500 src="https://i.imgur.com/L0GWCnF.png" >
(圖片來源 :　" *[IoT Ecosystem: A Survey on Devices, Gateways, Operating Systems, Middleware and Communication](https://link.springer.com/article/10.1007%2Fs10776-020-00483-7)* ")

上圖是一個比較完整的生態系，其中的 Middleware 及 Controller 可能都是包在一個 Cloud Server 之中，如果今天設備簡化，僅有一台 Gateway 甚至我們可以利用另一台 Gateway 直接與其對接 ( 也就是說我們可以不用 Cloud Server 來承接 )，這樣的狀況，整個 Web server, database server...等 Middleware 部分可能直接在 Gateway 中進行。


閘道器 Gateway
---

從上面的生態系中我們可以發現，Gateway 的角色在整個 IOT 中扮演著無法替代的角色，甚至我們可以省略 Cloud Server，來讓 Gateway 兼任其角色，進行 request 解析任務分發等工作。

除此之外，Gateway 的機構也相對於整個 IOT 中其他設備來的複雜，要處理無線訊號 ( wifi,Zigbee,... )、有線訊號 ( VRF,RS-485... )、資料存儲( SQL, MySQL, SQLite... )甚至是底層程序 ( process ) 的運行，幾乎可以說是集 IOT 大成於一身的重要設備。

在很多的資料中，大概都會提及 Gateway 在 IOT 中的重要地位，也會介紹一下 Gateway 就是承接客戶端及設備端的中介者，但其實幾乎很少人會提及 Gateway 的內部運作流程。

![](https://i.imgur.com/FTlugXY.png)

筆者依照自己的經驗簡單繪製了一下 Gateway 的 workflow (如上圖)。**這是一個無 Cloud Server 的情況**，當使用者利用 APP 或是控制台遠端對 Gateway 下達指令時，Gateway 內部會先經由 Web Server 解析 HTML，確認使用者意向及參數取得，之後利用編寫好的 API 看是要對設備做設定還是要查詢設備狀態。

如果只是要單純查詢設備狀況，我們直接利用 API 從 Database 撈資料即可，如果要對設備做設定，則會進入底層程序，利用 RS-485 或是 wifi 針對目標設備進行設定。與此同時，Gateway 連接的 Sensor 或是 wifi 也會定時對現場狀況或是設備狀態進行更新，寫入資料庫中。

當然，Gateway 不比 Cloud Server ，所保存的資料及時限都會有所限制。


後記 Postscript
---

大概在前兩天，筆者其實還搞不太懂 Gateway 到底在幹什麼，直到同事的講解以及自己這兩天的融會貫通跟大量的資料查詢後才好不容易稍微理解這中間 Gateway 實際扮演的角色到底是什麼。

不過這其實也只是一篇粗淺的概略介紹，從某一個層面來看，也是寫給自己看的，裡面還有許多的細節都尚未釐清楚，或許等日後對 IOT 更加熟悉後，可以再做補充。

參考資料
---

1. Bansal, S., Kumar, D. *IoT Ecosystem: A Survey on Devices, Gateways, Operating Systems, Middleware and Communication.* Int J Wireless Inf Networks 27, 340–364 (2020). https://doi.org/10.1007/s10776-020-00483-7
2. July Huang.(2018).*Gateway vs Router: What’s the Difference?* Web site : https://medium.com/@july.huang666/gateway-vs-router-whats-the-difference-fb010ee3b5cc