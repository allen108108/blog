---
title: "Android 初探 (一) ： 從 Hello  World ! 認識 Android 專案開發"
date: 2020-03-02 15:35:09
categories:
- 系統相關 OS
- Android
image: https://i.imgur.com/7YeW7Qb.png
mathjax: true
---

前言
---

在電腦視覺 ( 或是整個 AI 領域 ) 中，針對問題來進行建模，經過不斷的調整參數及模型結構來找出最能解決任務的模型，這些就是工作的全部 ？ 

其實並不然。

<!-- more -->

真正工作之後，發現建模並不是最困難及麻煩的，真正的大挑戰會是在模型訓練完成的部屬工作，依照不同的設備需求，將模型載入系統之中，還能順利進行每一次預測，這才會是真正算完成一個完整的專案。但若在部署上發生了無法解決的問題，再好的模型、再優異的性能表現，最後都只能淪為空談。因此，根據不同的設備、作業系統，都必須要有初步的了解，才能順利的將模型正確部署。

這篇文章也不是要分享什麼多專業的部分，畢竟筆者對於 Android 基本上就是 0 基礎    ，先從工作的專案邊做邊學習，然後單純想記錄一下自己工作上面遇到的狀況做個簡單的紀錄，如此而已。


Java 開發工具組 Java Development Kit
---

Android 是一個奠基於 Java 語言上的作業系統，當我們要進行 Android 開發時，必然需要 Java 的整體技術以及各種支援，因此，在進行開發之前，我們必須要先準備好這些工具，以便於後續的開發作業。

JDK，全名為 Java Development Kit，就是一整個 Java 開發工具組，整組的內容物如下圖

![](https://i.imgur.com/TDLWHOq.jpg)

其中比較多人會提到的大概就是 JVM 以及 JRE。

JVM ( Java Visual Machine )，的工作在於將 Java 語言可以在各個作業系統中運作，地位就像是一個翻譯人員，將 Java 這個外國人翻譯給每一個不同國籍的人聽。而 JRE ( Java Runtime Environment ) 的角色便是提供一個環境來執行 Java。

這些通通包在 JDK 裡面。

專屬於你的第一個 Android 專案
---
![](https://i.imgur.com/7YeW7Qb.png)

### 下載並安裝 JDK 及 Android Studio

上面簡單解釋了 JDK ，不難了解到 JDK 便是在開發以 Java 為基礎的程式所必需要的工具，當然也包括 Android，在下載 Android Studio 以前，我們必須要先下載 JDK。

JDK 下載直接至 Oracle [官網](https://www.oracle.com/java/technologies/javase-jdk13-downloads.html) 依照目前的作業系統選擇適當版本下載，安裝也非常簡單，照著步驟一步一步進行即可。

接下來進入到重點的 Android Studio 下載及安裝，一樣至官網[下載](https://developer.android.com/studio)，安裝一樣跟著步驟走就好。

### 新增 Android 專案

按照步驟安裝完成以後，系統會問你是要開新的專案還是選擇一個已經有的舊專案，這邊我們選擇 `Start a new Android Studio project`

![](https://i.imgur.com/CIysWiq.png)

選擇界面，第一次開發我們就選擇 `Empty Activity` 或是 `Basic Activity`

![](https://i.imgur.com/6maQAdp.png)

可以簡單設定一下專案的內容 : 專案名稱、安裝位址、要用 `Kotlin` 或是 `Java` 語言

![](https://i.imgur.com/GkGvS7j.png)

最後，系統會進行專案的建置 (build)，左下角紅框處會顯示建置的狀況，等到都打上綠色勾勾後就表示專案建置已經完成，我們可以開始著手進行專案內容的建構。

當然，這是一個完全新建立的專案，在 build 的過程不太可能會報錯。但若我們是拿別人的專案來用，就會有很高的機會會碰到 build 專案過程發生錯誤，這個時候就要根據報錯的內容進行調整，這又會是 case by case 的狀況，只能遇到一個坑就想辦法填一個坑來進行。


![](https://i.imgur.com/0tJcVIg.png)


### 專案架構

當我們完成了專案的建置後，一個專案的架構就會如下圖呈現在 Android Studio 之中，以下我們會依序介紹一下整體架構的內容為何

![](https://i.imgur.com/HwooGxD.png)

1. `app > java > com.example.myfirstapp > MainActivity`

當我們點開 APP 時，整個 APP 會根據這一個檔案的內容啟動 [Activity](https://developer.android.com/guide/components/activities?hl=zh-tw)，並且載入整體版面配置 (layout) 。

2. `app > res > layout > activity_main.xml`

這個檔案便清楚描述了整個使用者介面 (UI, Users Interface) 的[版面配置 (layout)](https://developer.android.com/guide/topics/ui/declaring-layout?hl=zh-tw)，

3. `app > manifests > AndroidManifest.xml`

此檔案描述了整個 APP 的基本特徵，並定義每一個組成。

4. `Gradle Scripts > build.gradle`

`Gradle Scripts`內會有兩個相同名稱的 `build.gradle` ，一個用於此專案，另外一個則是用於 APP 模組。這個檔案則是告訴系統應該如何進行建置這個專案，詳細內容可以參閱 " *[Configure your build](https://developer.android.com/studio/build/index#module-level)* "


### 利用 emulator 檢視執行成果

在 `Tools > AVD Manager` 中我們可以設置用來跑 APP 的系統模擬器 ( 當然，也可以直接接上有 Android 系統的移動設備 )。

先選擇我們想要的移動設備尺寸及型號

![](https://i.imgur.com/6SOTHql.png)

再來安裝我們想要使用的系統

![](https://i.imgur.com/UNlnWxK.png)

最後作一些基本設定就完成了可以用來跑 APP 的模擬器，這邊的設置通常可以完全不用調整，但如果像筆者本人的專案會希望利用筆電的 Webcam 模擬手機前後鏡頭，在這邊就必須要進行設定，不然會無法使用鏡頭。

![](https://i.imgur.com/Z5iIez0.png)

我們不是只能設置一台模擬器，我們可以加入多種不同尺寸、系統的模擬器來針對專案進行測試，但也要注意，這樣其實會占掉蠻多電腦內部的硬碟容量。若電腦本身的硬碟容量不大，則在這個部分必須要多注意。

我們也可以從紅框處看看現有的虛擬設備，最後選定要用來執行專案的模擬器後按下箭頭進行執行

![](https://i.imgur.com/zhIrxvx.png)

完全新增的專案中，若我們沒有調整專案內容，執行後則會出現 "Hello World!" 字樣。 


<img width=500 src="https://i.imgur.com/YjGHycp.png" >






結語 
---

很多的技術文章會帶著大家一步一步地進行專案開發，這樣很好，應該說非常好。因為經過幾篇文章之後，便可能可以做出一個像樣的 Android APP。

然而，這一篇文章中，筆者並沒有介紹太多專案開發的詳細過程，反而花一些時間把重點放在 Android 的基礎概念以及整個 Android 專案的架構。正如同文章開頭所說的，在完全 0 基礎的前提下，筆者希望在進行專案開發的過程中，也能了解基礎的專業知識，而不是單純的只有按照步驟來做 APP 開發而已。


參考資料
---
1. [Android Developers Documentation](https://developer.android.com/docs)
2. [2019 iT 邦幫忙鐵人賽 - [Andriod] Andriod Studio 從入門到進入狀況 系列](https://ithelp.ithome.com.tw/users/20105694/ironman/1642)
3. [JDK 安裝與認識](https://openhome.cc/Gossip/Java/JDK.html)



系列文章
---

* [Android 初探 (一) ： Android 初探 (一) ： 從 Hello  World ! 認識 Android 專案開發](https://bit.ly/3bouPaB)
* [Android 初探 (二) ： 從 "Hi, Android !" 了解 Activity 與 UI](https://bit.ly/2vOoFlb)