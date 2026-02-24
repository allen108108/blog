---
title: "【Python】 在 Linux 系統中使用 paramiko 實現遠端自動下載功能"
date: 2022-03-01 19:40:23
categories:
- 程式設計 Programming
- Python
image: https://i.imgur.com/E4tkdxs.png
mathjax: true
---

很久沒有更新文章，工作實在忙到一個境界，很多想寫的主題擱著擱著，就漸漸沒有動力寫下去。

最近工作剛好有一點空檔，也剛好前陣子有遇到一些問題，剛好趁這個機會抓一些簡單的主題來寫一些「小品」，當然，也順便再讓自己習慣於記錄自己的學習歷程、維持紀錄的手感。

<!-- more -->

在 Linux 系統中實現自動化 : Crontab
---

以往在進行 Linux 系統固定自動化工作時，最直覺想到的就是寫一個 Linux shell script 在系統執行，但倘若，我們希望在一個固定時間進行固定的任務，那或許我們會有更簡單的選擇 -- Crontab。

Crontab 稱作任務時間表，最小單位以分為單位，可以針對我們指定的任務，進行分、時、日、月、星期的固定排程。

在 Linux 系統中輸入 `corntab -e` ，可以進行任務時間的設定。

進入後，我們可以看到一大串註解的說明事項，最重要的是最後一個段落 : 

```
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
```

此段落旨在說明，我們應該要怎麼設置任務排程，簡單來說，我們要先設置我們需求任務的固定執行時間，再將要執行的任務記錄於後面。

```
* * * * * <execute script>
```

前面五個星號，各代表著 「分(Minute, min)、時(Hour)、日(Day of month, dom)、月(Month, mon)、星期(Day of week, dow)」，每一個位置的格式如下 ：

| 欄位名稱 | 格式 |
| -------- | -------- |
| Minute     | 0-59     |
| Hour     | 0-23     |
| Day of month     | 1-31     |
| Month     | 1-12     |
| Day of week     | 0-6     |

而 crontab 也引入一些特殊符號，讓整個排程設定更具彈性，特殊符號代表之意義如下 ：

| 特殊符號 | 代表意義 |
| -------- | -------- |
| ＊     | 星號代表任意數值     |
| ,     | 利用逗號分格不同時間數值     |
| -     | 減號可表示整個數值區間    |
| /n     | 斜線＋數字 n 可代表每隔 n 單位     |

舉例來說 ：

| 排程設定 | 設定符號 |
| -------- | -------- |
| 每分鐘執行 | `* * * * *`     |
| 每日上午9點15分執行 | `15 9 * * *` |
| 每月1日、15日晚上 12 點執行 | `0 0 1,15 * *` |
| 每週一至五晚間9點執行 | `0 21 * * 1-5` |
| 每隔20分鐘執行 | `/20 * * * *` |



利用 paramiko 進行 ssh 遠端下載
---

`paramiko` 是基於 ssh protocol 的一個 python 套件，我們可以利用 `pip install paramiko` 指令進行該套件的安裝。安裝完成後我們便可以利用 `paramiko` 進行遠端伺服器的訪問及上傳/下載檔案等動作。

```python=
import paramik

#開啟客戶端服務
ssh = paramiko.SSHClient()
#允許連接到沒有 host key 的伺服器上 
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
#利用帳號密碼進行伺服器登入，若有指定 port 亦可利用 port 參數進行設定
ssh.connect(hostname='HOST_NAME', username="USER_NAME", password="PASSWORD")
sftp = ssh.open_sftp()
```

從資訊安全的角度來看 `set_missing_host_key_policy()` 是非常危險的做法，通常不建議這樣使用，若我們已有 host key，則使用 `load_host_keys()` 來利用 host key 連接伺服器才是比較安全的做法。

連接到目標伺服器後，便可以針對特定檔案進行下載

```python=
localpath = '目標檔案位址'
remotepath = '來源檔案位址'
sftp.get(remotepath,localpath)
sftp.close()
ssh.close()
```

當然，我們也可以利用 `sftp.put(remotepath,localpath)` 來進行指定檔案的上傳。

搭配 Crontab 設定排程進行特定檔案下載
---

假設我們將自動下載的 python script 放在 `/home/user/folder/download.py` 中，我們希望每日上午九點從遠端伺服器中下載檔案至本機，且目標檔案位址如下：
```
/folder/srcFile.txt
```
則 `download.py` 的程式碼如下 ：

`download.py`
```python=
import os
import paramiko 
import datetime


def main():
    date = datetime.datetime.now().strftime("%Y-%m-%d")

    ssh = paramiko.SSHClient()
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    ssh.connect(hostname='HOST_NAME', username="USER_NAME", password="PASSWORD")
    sftp = ssh.open_sftp()
    localpath = 'srcFile.txt'
    remotepath = '/folder/srcFile.txt'
    sftp.get(remotepath,localpath)

    sftp.close()
    ssh.close()


if __name__ == '__main__':
    main()
```

再來就是要進行 crontab 的設置 ：

```
0 9 * * * cd /home/user/folder/ && /home/user/Anaconda3/bin/python /home/user/folder/download.py
```
值得注意的部分是，我們必須要先切換至 script 的資料夾，並且利用python執行該 script，而 python 必須直接引用 python 路徑才能正常執行。
