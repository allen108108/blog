---
title: Kaggle Case - Santander Customer Transaction Prediction (1)
date: 2019-10-08 02:06:34
categories:
- 案例討論 Case Study
image: https://miro.medium.com/max/1188/0*GPWsCoea-g1npXqH.png
mathjax: true
---

在這一個 Kaggle Challenge中，Santander提供了20萬筆的Train Data以及20萬筆的Test Data，希望Kagglers可以建構出一個模型來預測他們客戶是否能夠跟Santander進行交易。

在這邊，我選擇以Gabriel Preda的[Kernel](https://github.com/allen108108/Kaggle/blob/master/Santander_Customer_Transaction_Prediction/Santander%20EDA%20and%20Prediction.ipynb)來進行第一次的Kaggle Case研讀，也藉此了解一般在進行EDA以及Feature Engineering時可能會遇到的方法及解決的方式。


<!-- more -->

在這邊他大概以幾個大方向來依序做處理:
1. Prepare the data analysis
2. Data exploration
    * Check the data
    * Density plots of features
    * Distribution of mean and std
    * Distribution of min and max
    * Distribution of skew and kurtosis
    * Features correlations
    * Duplicate values
3. Feature engineering
4. Model



---

### Prepare the data analysis

``` python
import gc
import os
import logging
import datetime
import warnings
import numpy as np
import pandas as pd
import seaborn as sns
import lightgbm as lgb
from tqdm import tqdm_notebook
import matplotlib.pyplot as plt
from sklearn.metrics import mean_squared_error
from sklearn.metrics import roc_auc_score, roc_curve
from sklearn.model_selection import StratifiedKFold
warnings.filterwarnings('ignore')
```
從這邊我們大概可以看到在後面的Code裡面大概會用到那些模組、工具 ( 不過 Gabriel Preda的這一連串匯入中，有些本次似乎用不到。 )

會用 **os** 模組做一些系統操作 ( EX: 讀取檔案...)、**numpy**及**pandas**則是數據處理的好幫手、**seaborn**及**matplotlib**處理視覺化、**roc_auc_score**用來評估、**StratifiedKFold**用來驗證，最後此次用的 Model 則是 **lightgbm**

而這一整份的模組或許也可以當作我們的一份必備清單，日後是否有需要增減再做調整。
```python=
%%time
train_df = pd.read_csv("train.csv")
test_df = pd.read_csv("test.csv")
```
這裡，我將kaggle提供的檔案跟這個ipynb檔放在一起，所以在讀取的時候就很簡單，直接讀取就可以。若不同資料夾存放則可使用os模組進行讀取 [^1]

[^1]:存放於不同資料夾的檔案讀取
    ```python=
    dpath=os.path.abspath("待讀取資料的絕對路徑(不含檔案本身)") #資料夾位置
    fpath=os.path.join(dpath,"檔名(含副檔名)")                #檔案位置
    a_train=pd.read_csv(fpath)                               #讀檔
    ```


### Data exploration
#### Check the data


```python=
train_df.shape, test_df.shape
```

![](https://i.imgur.com/zkwUouv.png)



```python=    
train_df.head()
```
![](https://i.imgur.com/FyHhV8E.png)



```python= 
test_df.head()
```
![](https://i.imgur.com/acnLzJE.png) 

以上這三個算是很基本的資料檢視過程，觀察筆數、看看Train & Test 的各個欄位。也可以在此時簡單的看看資料數據是否有很嚴重的缺失值。

從這個例子來看，我們可以瞭解幾件事情 : 
* Train data :
    有20萬列，即為資料筆數
    有202行，包含了ID_Code、target、以及200行的未知欄位資料
* Test data : 
    有20萬列，即為資料筆數
    有201行，包含了ID_Code、以及200行的未知欄位資料

Train & Test資料差在 target，這的確可以理解，我們要利用訓練資料訓練出一套模型，再來對測試資料進行測試，看看準確度到多少，這便是我們要做的事情，而 Test 資料的 target 只有 Kaggle 才有，當我們最終答案上傳，他們會立刻進行比對，可以即時的告知我們結果。

```python=
def missing_data(data):
    total = data.isnull().sum() 
    #Pandas通常使用.isnull()來找缺失值，.sum()則是計算缺失的數量
    percent = (data.isnull().sum()/data.isnull().count()*100)
    tt = pd.concat([total, percent], axis=1, keys=['Total', 'Percent'])  
    # pd.concat(objs合併的東西,axis合併的方向,keys索引)
    types = []
    for col in data.columns:
        dtype = str(data[col].dtype)
        types.append(dtype)
    tt['Types'] = types
    return(np.transpose(tt))
```

```python=
missing_data(train_df)
```
![](https://i.imgur.com/mrThAya.png)

```python=
missing_data(test_df)
```
![](https://i.imgur.com/kpSqOi9.png)

檢查缺失值在整個資料處理的過程中，往往會是一個非常重要的步驟，缺失值數量過多，則跑出來的模型的擬合狀況會非常差，在這個Case裡面，很幸運地並沒有任何的缺失值在其中。
*(這蠻可惜的，畢竟如何填補、修改、刪除缺失值也是需要很多的技巧，只能等之後的Case再來學習了)*

> Gabriel Preda的這個缺失值計算函數 (後面很多自訂函數亦然) 我認為是一個可以共通的方式，可以記下來以便未來處理不同Case可以使用
> 

```python=
%%time
train_df.describe()
```
![](https://i.imgur.com/Q5vr23v.png)
```python=
%time
test_df.describe()
```
![](https://i.imgur.com/XPORwqT.png)

`DataFrame.describe()` 是一個非常好用的工具之一，可以一次將整個表格的統計量呈現出來，而這些統計量可以很清楚的呈現這些資料的分布及特性。

在這邊我們可以知道幾個狀況 : 
* 無論 Training 還是 Test data，標準差都不小
* Training 及 Test data 的極值、均值、四分位數、標準差都很接近
* 各欄位平均值差異極大

```python=
def plot_feature_scatter(df1, df2, features):
    i = 0
    sns.set_style('whitegrid')
    plt.figure()
    fig, ax = plt.subplots(4,4,figsize=(14,14))

    for feature in features:
        i += 1
        plt.subplot(4,4,i)
        plt.scatter(df1[feature], df2[feature], marker='+')
        plt.xlabel(feature, fontsize=9)
    plt.show();
```

```python=
features = ['var_0', 'var_1','var_2','var_3', 'var_4', 'var_5', 'var_6', 'var_7', 
           'var_8', 'var_9', 'var_10','var_11','var_12', 'var_13', 'var_14', 'var_15', 
           ]
plot_feature_scatter(train_df[::20],test_df[::20], features)
```
![](https://i.imgur.com/yfaIE8z.png)

這裡的部分，主要是把剛剛觀察的現象做一個視覺化的處理來確認我們的「觀察」是有根據的。從這裏的圖片看來，不管哪一個欄位，在兩個部分的資料的分布都算集中。

( 意即 : 在這40萬筆資料中，每一筆的資料的各個欄位/特徵大概都在某些範圍內不會有太大的異常值 )

> 有兩個問題大家可以思考一下 :
    1. 圖片中每一個點代表的意思是什麼?[^2]
    2. 假如出現完全零散不集中、或是呈現特殊集中方式代表的意義是什麼 ?
> 
[^2]:這個問題我其實思考了一陣子，我們稍微更改一下code你就能知道這代表什麼了:
    
    ```python=
    features = ['var_0', 'var_1','var_2','var_3', 'var_4', 'var_5', 'var_6', 
               'var_7', 'var_8', 'var_9', 'var_10','var_11','var_12', 'var_13', 'var_14', 'var_15', 
               ]
    plot_feature_scatter(train_df[0:1],test_df[0:1], features)

    ```
    我們就只取一個點來看看會是什麼
    ![](https://i.imgur.com/Wmyfy8P.png)
    你看出來了嗎 ? 
    圖中的每一個點，代表的就是Training Data跟Test Data相對應index資料的特徵值。
    拿第一個子圖來說，那一點代表的就是 
    index=0 時 Training Data var_0=8.9255 (X軸) ； 
    index=0 時 Test Data var_0= 11.0656 (Y軸)

```python=
sns.countplot(train_df['target'])
```
![](https://i.imgur.com/f4booTk.png)

```python=
print("There are {}% target values with 1".format(100 * train_df["target"].value_counts()[1]/train_df.shape[0]))

#train_df["target"].value_counts()[1] return "1" 有多少個
#train_df.shape[0] return 列數
```
![](https://i.imgur.com/yiEXtdI.png)

遇到了一個大難題，這是一個 unbalence 的情況。
之所以會困難是因為，如果我們訓練出一個模型，只會把所有資料無差別分類成 target 0 (Dummy classifier)，這樣的準確率也會接近90%，這會影響我們評估這個模型的好壞。(++這也是為什麼在最前面我們發現他選擇使用roc_auc_score, roc_curve來做為評估模型的指標++)

嗯，很順利的現在進行了1/3.....

### To be Continue....