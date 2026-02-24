---
title: 將已有的專案加入 TotoiseSVN 進行版本控制
date: 2020-09-05 18:53:40
categories:
- 後端開發 Backend Development
image: https://i.imgur.com/pZhNbwz.jpg
mathjax: true
---

好一段時間沒有更新文章，最主要的原因是筆者換了一份新工作，被指派到負責 IAQ (Indoor Air Quality) 專案中 Gateway (閘道器) 的後端開發，雖然跟深度學習目前較無相關，但後端、底層的部分也一直是我想要接觸的工作項目，只是對我來說都是嶄新的事物，便少了許多時間自己進行研究、學習，也相對的少了部落格文章的更新。

<!-- more -->

以上雖說都是題外話，但也是這篇文章的起點。

在資安控管嚴密的大公司中，其實並不太方便使用現在許多現有的工具進行開發，尤其針對一些較大的專案，少了 Github 的支援確實也難隨時進行版本控制及協作，這也因此必須要利用另外的工具，在資安的前提下可以順利的推動開發進程且隨時可以進行版控，TotoiseSVN 便是一個好工具之一。


版本控制系統 Version Control Ststem
---

### Concurrent Versions System (CVS)

作為跨平台開源版本控制的開山始祖，基本的版本控制目標都能實現，由於採用的是集中式版控系統 ( Centralized Version Control System )，CVS 僅對不同版本進行備份，所有程式碼集中統一管理。

開發人員進行開發時都必須從伺服器上下載，解決衝突部分最後進行 commit，所有的更動及版本資訊都儲存在 CVS server 中，這也表示，一但無法連線，便失去了協作及版本控制的功能。

![](https://i.imgur.com/fY50pFL.png)


### Subversion (SVN)

SVN 與 CVS 均為集中式版控系統，但不管是 SVN 還是後面會介紹的 Git，原則上都是基於改良 CVS 而產生的，SVN 有一個很大的改良是在於專案資料庫的格式改採用二進制格式 ( Binary format ) 而非 CVS 使用的 RFC 格式 ( Request for Comments format )。

這樣的改良大大改進了版控系統可讀檔案型態的限制 ( CVS僅可接受文件檔 )、讀寫平行、共享檔案...等優點，不過雖說如此，也造成了資料存儲變得不是這麼友善，必須有適當的工具輔助才能讓使用者更好上手 ( 正如本文的 TotoiseSVN )。

速度上由於整體架構的不同， SVN 的確比 CVS 快速很多，也支援更多的離線功能，但也因此必須付出巨大的儲存成本。

### Git

Git 應該可以算是目前最熱門的開源版控系統了，有別於 CVS 已逐漸退流行因此不再提供新功能，Git 的開發與維護都還不斷的在持續更新中。

Git 跟 SVN 與 CVS 最大不同點是其為分散式版本控制系統 ( Distributed Version Control System )，Git 的開發者必須在開發時將整個專案複製 (clone) 至 local，達成去中心化的概念。從下圖可以發現，開發者可以在 local 直接離線 commit，後續再藉由連線 puch 到 server 中進行同步。

![](https://i.imgur.com/NPYYC4C.png)



前面有提到 SVN 與 Git 都是基於改良 CVS 而發展出來的系統，Git 當初在開發時，Linus Torvalds 開發者提出了 " WWCVSND " ( What Would CVS Not Do ) 開發原則，至今看來，Git 目前的發展也的確達到這樣的原則，Git 發展至今已經不單單是一個版本控制系統，還是一個檔案管理系統。

### CVS, SVN, Git 比較

下面筆者簡單的列出這三者的比較 : 

![](https://i.imgur.com/y02SzCH.png)

CVS 已經幾乎被 SVN 與 Git 取代，然而 SVN 在某種層面上來說，仍然有著 Git 無法完全取代的優勢，舉例來說，因為 SVN 集中式版本控制的特性，在權限控制上比 Git 來的嚴密，因此在公司企業上，仍然以 SVN 的版控系統為首選，以確保核心程式的資訊安全。

TotoisSVN
---

![](https://i.imgur.com/pZhNbwz.jpg)

前面有提到 SVN 必須要有專門的工具軟體才能讓使用者方便進行版本控制，而 TotoiseSVN 便是基於這樣的需求而產生的工具。

安裝跟使用方面可以至官網( https://tortoisesvn.net/ ) 進行下載並且安裝，安裝過程本文不贅述，反正就是下一步連發就可以無痛安裝完成。

本文的重點放在如果我已經有一些專案，應該怎麼將其加入 SVN 進行專案協作及版本控制，也就是下一部份即將要介紹的。


將現有專案加入 TotoisSVN
---

### 備份專案

為了下面介紹方便，筆者假設已經擁有一個專案 ( 路徑為`C:/projects/project_1` )，內含三份文字檔，現在希望將其加入 SVN 中進行版本控制。因為是要處理現有的專案，為了以防萬一防止過程中操作不慎導致檔案受損或遺失，筆者強烈建議進行下面步驟以前必須先進行備份。

![](https://i.imgur.com/pN0YGid.png)


### 建立 SVN 及 temp 資料夾

我們建立需要的 `temp` ( 路徑為`C:/temp` )  與 `SVN` 資料夾 ( 路徑為`C:/SVN` ) ，並在 SVN 資料夾中建立建立版本庫的 Repository ( 路徑為`C:/SVN/test` )，在資料夾上按右鍵選擇 `TotoiseSVN > Create repository here` 

![](https://i.imgur.com/A7FNxl0.png)

按下 `OK` 後便完成 Repository 的建立，我們進入 `test` 資料夾中可以發現以下資料夾結構，這樣表示 Repository 已經建立完成

![](https://i.imgur.com/3QOnUx0.png)

接下來，我們在 `temp` 資料夾中建立子資料夾 `new`，並於底下建立三個子資料夾 : 

* `C:/temp/new/branches`
* `C:/temp/new/tags`
* `C:/temp/new/trunk`

![](https://i.imgur.com/qeFkNE4.png)

這樣的結構在未來如果使用到高階的專案管理功能上會有很大的作用，即使現在可能不見得會遇到，但預先建立這些結構並不是一件壞事，也不影響目前的版控功能。

建立完成後，將我們原有的專案底下「**需要版控的資料及資料夾**」( 假設需要版控的資料為三個資料中的`textA.txt`及 `textB.txt` )「**移動**」到 `./temp/new/trunk` 底下，剩下不需要版控的資料便先找地方暫存著，**注意 !!一定要保持專案資料夾 `C:/projects/project_1` 內全空 。**



![](https://i.imgur.com/n8bXUcz.png)


### 將專案導入 TotoiseSVN

於 `new` 資料夾上按滑鼠右鍵選擇 `TotoiseSVN > Import` ，於網址列中輸入 `file:///C:\SVN\test` 將這個資料夾一併導入到 SVN 中，


![](https://i.imgur.com/mZhp1VE.png)


OK，我們回到 `C:\SVN\test` 按下右鍵選擇 `TotoiseSVN > Repo-browser` 便可以確定我們已經將 `new` 底下的三個資料夾導入 SVN 中了

![](https://i.imgur.com/E6TlIfz.png)

利用 Repo-browser 也可以確定我們選擇要進行版控的兩個檔案也在 `trunk` 資料夾中。此時 `temp` 資料夾已經不需要了，可以逕行刪除。

### 導回原專案位置

不過別忘了，當初我們是將原本專案中要版控的檔案 **移動** 到 `temp/new/trunk` 中，我們還是會希望在原本的專案位址中進行檔案的修改，而非在 `temp/new/trunk` 進行更動，所以最後一步便是要將其導回原本的專案位址中。

這步很簡單，我們在原本專案資料夾中，按下滑鼠右鍵選擇 `SVN Checkout` ， URL 的部分填入 `file:///C:/SVN/test/trunk` ，而 Checkout Directory 部分填入 `C:\projects\project_1` 

![](https://i.imgur.com/q9gx0rM.png)

按下 OK 後，便可以發現到需要版控的檔案都回到原本資料夾中，且前面都會多了一個小綠勾，表示目前版本為最新版本。[^1]

![](https://i.imgur.com/cF3SUoD.png)

別忘了，我們還有不需要版控的資料再將其移回至此資料夾中即可。


專案版本控制
---

現在，我們已經可以進行版本控制了，日後只要在現有專案文件上進行更改，前面的小綠勾就會轉變成紅色的驚嘆號，那麼我們只需要對此檔案按下滑鼠右鍵選擇 `SVN commit` 即可提交新版本。

如果要增加新的檔案，那我們必須先按滑鼠右鍵選擇 `TortoiseSVN > Add` 將其添加至 SVN 中，再進行 `SVN commit`  即可。


參考資料
---
1. [SVN與CVS兩者間的比較](https://codertw.com/%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80/459584/)
2. [版本控制軟體比較](https://zh.wikipedia.org/zh-tw/%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6%E8%BD%AF%E4%BB%B6%E6%AF%94%E8%BE%83)
3. [Git 前时代：使用 CVS 进行版本控制](https://zhuanlan.zhihu.com/p/51792519)
4. [Git 、CVS、SVN比较](https://blog.csdn.net/whatday/article/details/84135984)
5. [Difference between Concurrent Versions System (CVS) and Subversion (SVN)](https://www.geeksforgeeks.org/difference-between-concurrent-versions-system-cvs-and-subversion-svn/)


註釋
---

[^1]: 如果我們的電腦曾經下載過一些軟體，例如趨勢防毒、onedrive...等，這些都會使用到圖示的疊加功能，Windows有限制最多只能有 15 個圖示疊加，如果未出現 TotoiseSVN 沒有出現小綠勾 很有可能是前面安裝過的軟體已經佔掉這 15個名額，詳細調整方法可以參考 : [TortoiseGit 的綠色勾勾消失了](http://mycodetub.logdown.com/posts/253390-tortoisegit-green-hook-is-gone)、[SVN圖標突然消失了怎麼辦](https://www.taodabai.com/how/885112511.html)..等文章