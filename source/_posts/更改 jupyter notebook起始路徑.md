---
title: 更改 jupyter notebook起始路徑
date: 2019-10-08 02:15:42
categories:
- 軟體 Software
image: https://junyou.tw/wp-content/uploads/2018/08/jupyter-1024x509.png
mathjax: true
---

正常 jupyter notebook 的開啟路徑都在 C:\users 內，但相信很多人真正在處理資料的地方都在另外一個資料夾，也當然希望 jupyter notebook 開啟後可以直接處理檔案，不需要一直切換路徑。

在這裡，提供一些小撇步，讓路徑可以隨自己的喜好來設定。

<!-- more -->

1. 開啟 Anaconda Prompt 
2. 輸入`jupyter notebook --generate-config  `，此時出現的會是一串路徑如下

    ![](https://i.imgur.com/dsPa0E4.png)
    
>值得注意的地方是，如果從未改過路徑，return的會是單純一個路徑，但由於我已經先改過路徑因此這邊變成會問你是否要用預設值覆蓋 ( Yes代表路徑會恢復成預設值，No 表示我要維持更改過後的設定)
>

3. 依照return的路徑尋找文件 "jupyter_notebook_config.py"
4. 使用記事本打開此文件，尋找下列文字

        # The directory to use for notebooks and kernels.  
        #c.NotebookApp.notebook_dir = u'' 

5. 將第二行開頭的 # 刪除，並將希望 jupyter notebook起始的資料夾路徑填在兩個單冒號中間，並將原本單斜線改為雙斜線，舉例如下 :
                
        c.NotebookApp.notebook_dir = u'C:\\python'

6. 結束後儲存，並重啟 jupyter notebook 即可。