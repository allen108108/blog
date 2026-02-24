---
title: "Line Bot 在群組中的喚醒/休眠機制 (Django Database 之應用) "
date: 2021-08-30 19:06:19
categories:
- 專案 Project
- Line Bot
image: https://i.imgur.com/OLINzCe.png
mathjax: true
---

系列文章
---

本文對於以 Django 為基礎設計的 Line Bot 細節並不會有太多著墨，這些都在先前文章中有提到，建議讀者可以先行閱讀下面兩篇文章 : 

* [MoBot : LINE Bot 開發紀錄 ( LINE / Heroku )](https://allen108108.github.io/blog/2020/03/26/MoBot%20_%20LINE%20Bot%20%E9%96%8B%E7%99%BC%E7%B4%80%E9%8C%84%20%28%20LINE%20_%20Heroku%20%29)
* [芋香冬瓜「查」 : 專為自家需求的機器人查詢系統 App ( MoBot ver.2 )](https://allen108108.github.io/blog/2020/06/26/%E8%8A%8B%E9%A6%99%E5%86%AC%E7%93%9C%E3%80%8C%E6%9F%A5%E3%80%8D%20_%20%E5%B0%88%E7%82%BA%E8%87%AA%E5%AE%B6%E9%9C%80%E6%B1%82%E7%9A%84%E6%A9%9F%E5%99%A8%E4%BA%BA%E6%9F%A5%E8%A9%A2%E7%B3%BB%E7%B5%B1%20App%20%28%20MoBot%20ver.2%20%29/)
* [利用 Line Notify 於 Line Bot ( Django + Heroku ) 進行群組推播提醒](https://allen108108.github.io/blog/2022/03/01/%E5%88%A9%E7%94%A8%20Line%20Notify%20%E6%96%BC%20Line%20Bot%20(%20Django%20+%20Heroku%20)%20%E9%80%B2%E8%A1%8C%E7%BE%A4%E7%B5%84%E6%8E%A8%E6%92%AD%E6%8F%90%E9%86%92/)
* [Line Bot 在群組中監控成員的加入與退出](https://allen108108.github.io/blog/2022/03/02/Line%20Bot%20%E5%9C%A8%E7%BE%A4%E7%B5%84%E4%B8%AD%E7%9B%A3%E6%8E%A7%E6%88%90%E5%93%A1%E7%9A%84%E5%8A%A0%E5%85%A5%E8%88%87%E9%80%80%E5%87%BA/)

<!-- more -->

Line Bot in Group
---

一般來說，Line Bot 在群組中，會將所有人的對話都視為一個 event，既然監聽到 event 的發生，那麼就很容易會執行回覆，最後造成的結果就是 Line Bot 像跳針一樣一直跳出來回應。

通常要解決這種狀況，有兩種方法，一種是在 Line Bot 的設計上，均使用招呼語與 Line Bot 進行對話 : 「小黑 今天天氣如何 ?」、「小白 使用方式 ?」....諸如此類的對話方式，但這種方式通常太過不自然，而且如果今天我們設計的機器人不完全是 for 群組使用，在一般個人與 Line Bot 的應對上還得加上招呼語也顯得太過麻煩。

另外的方法，也是蠻多線上的群組機器人比較常用的方式就是讓機器人本身有休眠的機制存在，也就是說，當我們要使用 Line Bot 時再喚醒他，不需要用的時候可以主動讓它休眠，也可以讓它在一定時間後自動休眠。這，也是我們今天要討論的方式。


設計概念
---

要達成這種喚醒機制，就必須要利用資料庫來記錄 Line Bot 的喚醒時間以及到期時間。當收到每一次的群組 event 時，便會跟資料庫中的資料進行比對，若時間超過了到期時間，便不作任何回應，並且刪除資料庫中的資料，直到下一次接收到喚醒的指令。


概念其實不難，接著就是實作的部分。

筆者沿用之前的 Line Bot 設計，使用 Django python 框架，在這樣的條件下應該怎麼進行資料庫的操作呢 ? 這就是接下來的主題了。


新增資料庫表單
---

要在 Django 中建立資料庫表單並且進行存取、寫入等動作，連結的就是四個檔案 : 

`models.py` : 負責定義資料庫表單的初始狀態，包含欄位資料以及預設資料，除此之外，也可以定義針對這個表單進行各種動作的函數等....

`admin.py` : 此處針對建立的表單進行註冊的動作，註冊後，我們可以於 Django 的管理後台進行表單的管理。

`settings.py` : 這邊我們使用的是 SQLite 資料庫系統，因此在此處進行 QLㄔㄛ設定。

`views.py` : 這裡就不用多說了，使用者與資料庫的互動主要發生在此處，使用者進行什麼樣的動作，而可以對資料庫進行怎麼樣的行為，主要都是定義在這邊。

#### models.py

因為我們希望可以當使用者利用某些方式 ( 例如 : 招呼語 ) 喚醒 Line Bot 時，可以在資料庫中洩入被喚醒的時間及到期的時間 ( 此處預設三分鐘 Line Bot 會休眠 )，因此我們建立以下的表單 : 

```python=
class GroupAliveTbl(models.Model):
    # 定義 GroupAliveTbl 的欄位以及資料型態
    GroupID = models.CharField(max_length=40)
    AliveDate = models.DateTimeField()
    Expired = models.DateTimeField(null=True)
    
    # 定義 GroupAliveTbl 的函數
    # 新增一筆資料
    def creatAlive(id):
        GroupAliveTbl.objects.filter(GroupID=id).delete()
        groupAlive = GroupAliveTbl(
            GroupID = id,
            AliveDate = timezone.now(),
            Expired = timezone.now() + timedelta(minutes = 3)
        )
        groupAlive.save()
    
    # 判斷 Line Bot 是否休眠
    def isAlive(id):
        groupAlive = GroupAliveTbl.objects.filter(GroupID=id)
        if groupAlive.exists() == False:
            return False
        groupAlive = groupAlive[0]
        return timezone.now() < groupAlive.expire
```

資料型態的部分，常見的欄位型態有：



| 欄位型態 | 說明 |
| -------- | -------- |
|CharField | 欄位型態為字串|
|TextField | 欄位型態為多行字串|
|IntegerField | 欄位型態為整數|
|PositiveIntegerField | 欄位型態為正整數|
|floatField | 欄位型態為浮點數|
|BooleanField | 欄位型態為布林值|
|DateField | 欄位型態為日期|
|DateTimeField | 欄位型態為日期時間|

而我們也可以給欄位屬性 ：

| 屬性 | 說明 |
| -------- | -------- |
| null | 是否為空值|
| primary_key | 是否為主鍵|
| unique | 是否唯一值|
| null | 可為空值|

在 `model.py` 中以 `class` 定義表單，而在內部定義其函數方法以供後續利用。在此例中，定義了 `creatAlive` 函式來建立一筆新的 Alive 資料，並且計算出到期時間 (Expired)，另外，利用 `isAlive` 來判斷是否 Line Bot 已休眠。

#### admin.py

```python=
class GroupAliveTbl_Admin(admin.ModelAdmin):
    list_display = ('GroupID','AliveDate','Expired')

admin.site.register(GroupAlive,GroupAliveTbl_Admin)
```

在這個步驟，我們加入了 ModelAdmin 類別來定義要在 admin 後台管理介面顯示的欄位，之後再用 `register` 來進行註冊。不過這部分並非我們此篇文章的重點，因此筆者不會利用太多篇幅來進行解釋。


#### settings.py

這部分應該在初期建構 Line Bot 時就已經寫進 `setting.py` 之中，但有個部分還是要稍微提醒一下。

```python=
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / "db.sqlite3",
    }
}
```

原本上述的設定，在實際應用上會一直報錯，經查詢過後，需要改為 `str(BASE_DIR / "db.sqlite3")` 才可正常運作。


#### views.py

在 Line Bot 的設計上，筆者希望能將 Line Bot 在群組內外的動作進行切割，這樣可以確保，我們在單獨跟機器人進行溝通的時候可以避免掉喚醒/休眠的機制。

為了達成這個目的，我們得先了解到 Line Bot 怎麼判斷現在是群組訊息還是個人訊息


從每一個 Line Bot 接收到的 event，我們其實除了可以知道接收到來自於哪個使用者的訊息外，還可以知道目前的對話是跟個人還是在群組內的對話

`event.source.use_id` : 接收到訊息是來自於哪一個使用者 id
`event.source.group_id` : 接收到訊息是來自於哪一個群組 id
`event.source.text` : 訊息內容
`event.source.type` : 這個對話是來自於客人 `user` 還是群組 `group`

確定了 Line Bot event 是來自於個人或群組後，便可以針對不同的條件進行不同的動作，當 event 來自於群組並接受到招呼語時應該如何、當 event 來自於群組並接受到休眠提示語時應該如何而當 event 來自於個人時又應該如何回應 : 

```python
for event in events:
    if isinstance(event, MessageEvent):
        inGroup = False
        group_id = ''
        user_id = event.source.user_id
        mtext = event.message.text
        if (event.source.type == 'group'):
            #當 event 來自於群組，這邊設置一個參數，以區別後續回覆函式 sendResponse 辨別用
            inGroup = True
            group_id = event.source.group_id
            #當 event 來自於群組，並接受到喚醒提示語
            if event.message.text == '嗨 LineBot':
                GroupAlive.creatAlive(event.source.group_id)
                line_bot_api.reply_message(event.reply_token, TextSendMessage(text='找我幹嘛 ?'))
            #當 event 來自於群組，並接受到休眠提示語
            elif event.message.text == 'LineBot 掰':
                GroupAlive.objects.filter(groupId=event.source.group_id).delete()
                line_bot_api.reply_message(event.reply_token, TextSendMessage(text='OK~掰'))
            #當 event 來自於群組，且未進入休眠狀態
            elif GroupTicket.isAlive(event.source.group_id)):
                sendResponse(event, mtext, user_id, inGroup, group_id)
        #當 event 來自於個人
        elif event.source.type == 'user' :
            sendResponse(event, mtext, user_id, inGroup, group_id)
```

如此一來，Line Bot 便可以在群組對話時，不會一直回覆所有使用者的對話，在需要它時再將其喚醒進行動作即可。當然，Line Bot 在群組間的動作還可以有更複雜的變化，例如當群組有心加入成員時，Line Bot 亦可以針對這樣的事件進行有別於對話的主動回覆，這部份我們就留待下篇文章再來討論吧 !
