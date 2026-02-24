---
title: "Deep Reinforcement Learning (8) --- Imitation Learning" 
date: 2020-04-22 17:39:48
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning and having it Deep and Structured
image: https://i.imgur.com/MdVC5JZ.png
mathjax: true
---


前一個部分，我們了解到如果遇到 Sparse Reward 時，可以怎麼樣做學習。但，如果今天可能連 Reward 都沒有，又或者是 Reward 很難定義的狀況下，可以怎麼做學習呢 ? Imitation Learning ( 或稱 Learning by Demonstration / Apprenticeship Learning ) 就是在解決這樣的情況。

<!-- more -->

Behavior Clone
---

Behavior Clone 其實就是一種 Supervised Learning，我們無法得到 Reward，就收集許多專家 (Expert) 的資料

$$
(s_1, \hat{a}_1)\\
(s_2, \hat{a}_2)\\
(s_3, \hat{a}_3)\\
(s_4, \hat{a}_4)\\
\vdots
$$

然後設計一個神經網路來進行訓練。


<img width=500 src="https://i.imgur.com/wLFDAac.png" >



這樣的訓練過程看似簡單，但真正在實作上仍會存在一些問題  : 

(1)
我們在收集資料的時候往往都收集的是有限的資料，難以讓機器學習到各種可能情況的應對方式，舉例來說，收集駕駛人的駕駛行為給自駕車學習，但它可能會學習不到當即將遇到車禍時應該怎麼處理，因為我們的資料裡面可能收集不到即將出車禍時駕駛的應對方式。又或者，當我們收集玩家遊戲的資料給機器做學習時，也可能很難學習到發生一些突發狀況時應該怎麼反應。

這些狀況，只能夠靠著 Data Aggregation 收集更多樣化的資料來避免。但在許多真實情況下，Data Aggregation 的資料收集其實是不切實際的，拿上面自駕車的例子來看，我們不可能一直讓車子持續往牆壁衝過去然後一直收集駕駛人的行為吧 ? 

(2)
其次，收集專家的資料時，往往也會同時收集到專家行為中某些與任務較為無關的行為，這樣子的行為可能出於個人習慣等等因素。如果機器可以百分之百完全複製專家行為倒也還好，但我們設計的 Neural Network Capacity 是有限的，在學習過程中不可能百分百完全學習，如果機器只學習到這些無關的行為，便會出現問題。

(3)
Supervised Learning 的每一筆資料都是獨立的，上一筆資料有所誤差，並不會影響到下一筆的預測。然而 RL 中，每一筆資料都是有序列關係的，一個 State 往往決定於上一個 Action，除非 Data Aggregation 可以收集到足夠多樣的資料且機器可以完全做到訓練 0 誤差，否則，在 Bahavior Clone 中，訓練資料與測試資料的分布基本上會完全不同。




Inverse Reinforcement Learning
---

Behavior Clone 的問題衍伸出另外一種方法 -- Inverse Reinforcement Learning。在講解 IRL 的訓練過程之前我們先來看看 RL 與 IRL 的不同 : 

### Reinforcement Learning

給定一個 Reward Function ，利用不斷的互動來找出可以使 Reward 最大化的 Actors。

### Inverse Reinforcement Learning

給定一些專家行為資料，試圖去找出一個 Reward Function ，在從這個 Reward Function 中找出最佳行為模式。找出最佳模式後再重新找出一個新的 Reward Funciton ，如此反覆訓練讓最佳行為模式越來越好。


其實，從上面的敘述就可以了解到 IRL 的整個訓練框架如下

![](https://i.imgur.com/aZDubVK.png)

* 隨機初始一個 Actor $\pi$
* 讓 $\pi$ 與環境互動後試圖找出一個 Reward Function，可以讓專家的 Action $\hat{\pi}$ 得到的 Reward 總是高過於 $\pi$ 所的到的 Reward。
* 利用找出來的 Reward Function 來訓練 $\pi$ 可以得到更好的分數
* 再利用訓練完成的 $\pi$，重新找一個 Reward Function，可以讓專家的 Action $\hat{\pi}$ 得到的 Reward 總是高過於 $\pi$ 所的到的 Reward。
* 重複上列步驟

仔細想想，這樣的框架不就是 GAN 的概念嗎 ?


