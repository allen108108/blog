---
title: "Line Bot 在群組中監控成員的加入與退出"
date: 2022-03-02 18:53:35
categories:
- 專案 Project
- Line Bot
image: https://i.imgur.com/OLINzCe.png
mathjax: true
---

系列文章
---

* [MoBot : LINE Bot 開發紀錄 ( LINE / Heroku )](https://allen108108.github.io/blog/2020/03/26/MoBot%20_%20LINE%20Bot%20%E9%96%8B%E7%99%BC%E7%B4%80%E9%8C%84%20%28%20LINE%20_%20Heroku%20%29)
* [芋香冬瓜「查」 : 專為自家需求的機器人查詢系統 App ( MoBot ver.2 )](https://allen108108.github.io/blog/2020/06/26/%E8%8A%8B%E9%A6%99%E5%86%AC%E7%93%9C%E3%80%8C%E6%9F%A5%E3%80%8D%20_%20%E5%B0%88%E7%82%BA%E8%87%AA%E5%AE%B6%E9%9C%80%E6%B1%82%E7%9A%84%E6%A9%9F%E5%99%A8%E4%BA%BA%E6%9F%A5%E8%A9%A2%E7%B3%BB%E7%B5%B1%20App%20%28%20MoBot%20ver.2%20%29/)
* [Line Bot 在群組中的喚醒/休眠機制 (Django Database 之應用)](https://allen108108.github.io/blog/2021/08/30/Line%20Bot%20%E5%9C%A8%E7%BE%A4%E7%B5%84%E4%B8%AD%E7%9A%84%E5%96%9A%E9%86%92_%E4%BC%91%E7%9C%A0%E6%A9%9F%E5%88%B6%20(Django%20Database%20%E4%B9%8B%E6%87%89%E7%94%A8)/#more)
* [利用 Line Notify 於 Line Bot ( Django + Heroku ) 進行群組推播提醒](https://allen108108.github.io/blog/2022/03/01/%E5%88%A9%E7%94%A8%20Line%20Notify%20%E6%96%BC%20Line%20Bot%20(%20Django%20+%20Heroku%20)%20%E9%80%B2%E8%A1%8C%E7%BE%A4%E7%B5%84%E6%8E%A8%E6%92%AD%E6%8F%90%E9%86%92/)

<!-- more -->

前言
---

Line Bot 加入群組後，可以跟使用者進行互動外，還可以協助我們來監控成員的加入以及退出。這樣的動作對於群組成員的管理至關重要，在商業應用上，也可以將不同群組的使用者建立在同一個資料庫中一併進行管理，我們可以從資料庫中了解我們所有的使用者以及其所屬的群組，未來在進行訊息的推播、資料的收集以及相關應用。

因此，筆者希望藉由本文帶領讀者了解如何利用 Line Bot 來監控成員的加入/退出，以及後續的延伸應用。


事件 Event
---

從系列文章 [MoBot : LINE Bot 開發紀錄 ( LINE / Heroku )](https://allen108108.github.io/blog/2020/03/26/MoBot%20_%20LINE%20Bot%20%E9%96%8B%E7%99%BC%E7%B4%80%E9%8C%84%20%28%20LINE%20_%20Heroku%20%29) 我們曾經介紹過，曾經介紹過，Line Bot 的回應觸發是以事件 (Event) 為主，先前文章中提到的訊息回覆，則是利用 `isinstance(event, MessageEvent)` 判斷是否為訊息事件 (MessageEvent)，若是，則根據訊息內容來進行動作判斷。

```python=
def callback(request):
    if request.method == 'POST':
        signature = request.META['HTTP_X_LINE_SIGNATURE']
        body = request.body.decode('utf-8')

        try:
            events = parser.parse(body, signature)
        except InvalidSignatureError:
            return HttpResponseForbidden()
        except LineBotApiError:
            return HttpResponseBadRequest()
        for event in events:
            if isinstance(event, MessageEvent):
                user_id = event.source.user_id
                mtext = event.message.text
                
                """
                將收到的文字 mtext 進行後續的回應判斷
                """
                
        return HttpResponse()
    else:
        return HttpResponseBadRequest()
```

#### MemberJoinedEvent / MemberLeftEvent

正如同訊息事件一樣， Line Bot 針對成員的加入或退出，也是利用事件 ( MemberJoinedEvent / MemberLeftEvent ) 來進行動作的觸發，我們可以由 [Line Messageing API](https://developers.line.biz/en/reference/messaging-api/) 來查看 `MemberJoinedEvent` 事件的範例格式 ( json ) : 


```json=
{
  "destination": "xxxxxxxxxx",
  "events": [
    {
      "replyToken": "0f3779fba3b349968c5d07db31eabf65",
      "type": "memberJoined",
      "mode": "active",
      "timestamp": 1462629479859,
      "source": {
        "type": "group",
        "groupId": "C4af4980629..."
      },
      "joined": {
        "members": [
          {
            "type": "user",
            "userId": "U4af4980629..."
          },
          {
            "type": "user",
            "userId": "U91eeaf62d9..."
          }
        ]
      }
    }
  ]
}
```

從上述範例中，我們可以了解事件為 MemberJoinEvent，那麼 event.joined.member 會輸出加入群組使用者的陣列，其中包含了使用者 ID，除此之外，我們也可以同時知道群組的 ID。

當使用者加入時，我們可以利用 Line Bot 來進行歡迎語句的回覆

```python=
if isinstance(event, MemberJoinedEvent):
        line_bot_api.reply_message(event.reply_token, TextSendMessage(text=f'Hi ! 歡迎加入 ~'))
```

當然成員離開時也可以有類似的動作

```python=
if isinstance(event, MemberLeftEvent):
        line_bot_api.reply_message(event.reply_token, TextSendMessage(text=f'掰掰 ~'))
```

成員加入/離開與資料庫處理
---

前面有提到，我們可以知道加入者們以及群組的 ID，利用 API 我們亦可以藉由 ID 把使用者暱稱找出來，知道了這些訊息後，我們便可以將這些資訊寫入資料庫中。

首先，我們在 `models.py` 中定義使用者的資料庫及新增/刪除使用者的方法 ( `createByUserId` / `deleteByUserId` )，加入使用者的時候同時加入使用者 ID 及暱稱

`models.py`
```python=
class MemberTbl(models.Model):
    UserID = models.CharField(max_length=35)
    UserName = models.CharField(max_length=20)
    BPJID = models.CharField(max_length=3)
    GWID = models.CharField(max_length=15)
    def createByUserId(userId, userName, bpjid, gwid):
        MemberTbl.objects.filter(UserID=userId).delete()       
        memberTbl = MemberTbl(
            UserID = userId,
            UserName = userName,
        )
        memberTbl.save()
    def deleteByUserId(userId):
        MemberTbl.objects.filter(UserID=userId).delete() 
```

並且在 `admin.py` 中建立資料庫

`admin.py`
```python=
class MemberTbl_Admin(admin.ModelAdmin):
    list_display = ('UserID','UserName')

admin.site.register(MemberTbl,MemberTbl_Admin)
```

資料庫準備好後，我們便可以在成員加入群組的事件中，同時將使用者加入資料庫

```python=
if isinstance(event, MemberJoinedEvent):
    for member in event.joined.members:
        # 針對每一位使用者取得群組ID及使用者ID
        user_id = member.user_id
        group_id = event.source.group_id
        # 利用 API 取得使用者暱稱
        profile = line_bot_api.get_profile(user_id)
        user_name = profile.display_name
        # 新增使用者至資料庫中
        MemberTbl.createByUserId(user_id, user_name, bpjid, GWID) 
        # 加入歡迎語句
    line_bot_api.reply_message(event.reply_token, TextSendMessage(text=f'Hi ! 歡迎加入 ~')) 
```


當使用者退出群組時，我們亦可以將使用者自資料庫中刪除

```python=
if isinstance(event, MemberLeftEvent):
    for member in event.joined.members:        
        for member in event.left.members:
            # 針對每一位使用者自資料庫中進行刪除
            user_id = member.user_id
            MemberTbl.deleteByUserId(user_id)  
```