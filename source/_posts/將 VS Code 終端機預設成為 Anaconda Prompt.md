---
title: 將 VS Code 終端機預設成為 Anaconda Prompt
date: 2019-10-13 04:41:28
categories:
- 軟體 Software
image: https://pic2.zhimg.com/v2-269ee3c9fa04c0a4705c251309c75260_1200x500.jpg
mathjax: true
---

前幾天搞壞了一個 Anaconda Environment，只能利用其他的 Env. 來繼續手邊的工作。但由於這些其他的 Env. 大多為特定狀況下使用，也並不是常用的環境，所以很多的套件、設定多少都會要重新處理，這也不算太意外。

<!-- more -->

但卻遇到了一個挺匪夷所思的狀況，在原先舊的環境中，使用 VS Code 終端機執行都沒問題的小程式，換了一個環境卻出現問題。 

在該環境下的 Anaconda Prompt 或是 Pycharm 上執行都不會有問題，但同一個環境下的 VS Code 終端機執行卻屢屢發生錯誤。

目前大概猜想是預設終端機 PowerShell 上面的問題，既然如此乾脆把 VS Code 終端機設定為 Anaconda Prompt ，也解決一些整合上的問題。

Step 1. 開啟設定檔
---
檔案 > 喜好設定 > 設定 > 功能 > 終端機 > Integrated › Shell: Windows 點選 " 在 settings.json 內編輯 "

![](https://i.imgur.com/UwZMBZH.png)

Step 2. 修改設定檔
---

### 修改 "terminal.integrated.shell.windows"

將其值改為 "C:\\WINDOWS\\System32\\cmd.exe"

### 加上 "terminal.integrated.shellArgs.windows"

在 { } 內部最後加上一段

```
"terminal.integrated.shellArgs.windows": [
        "/K", "C:\Users\allen\Anaconda3\Scripts\activate.bat C:\Users\allen\Anaconda3"
    ]
```

![](https://i.imgur.com/MQxGYwj.png)

[ ] 內部的值請將 Anaconda Prompt 點選滑鼠右鍵選擇 內容 > 目標 內的值填入即可


<img width=500 src="https://i.imgur.com/Zph52Dx.png" >


