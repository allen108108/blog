---
title: Kaggle Case - Santander Customer Transaction Prediction (2)
date: 2019-10-08 02:08:01
categories:
- 案例討論 Case Study
image: https://miro.medium.com/max/1188/0*GPWsCoea-g1npXqH.png
mathjax: true
---

> 本篇為此系列文中第二篇，若還沒有讀過前文的讀者，建議從" [Kaggle Case - Santander Customer Transaction Prediction (1)](https://hackmd.io/s/B1YMZu6wN) "開始閱讀唷 !
> 


<!-- more -->

### Data exploration



我們在上一篇已經進行了初步的 Data Exploration ，本篇開始要對資料再作更進一步的探索，以便後續的特徵工程能夠順利進行。

#### Density plots of feature
首先，這邊先對 target 0,1 中各個欄位/特徵的密度做一個了解

```python=
def plot_feature_distribution(df1, df2, label1, label2, features):
    i = 0
    sns.set_style('whitegrid')
    plt.figure()
    fig, ax = plt.subplots(10,10,figsize=(18,22))

    for feature in features:
        i += 1
        plt.subplot(10,10,i)
        sns.kdeplot(df1[feature], bw=0.5,label=label1) #核密度函數:繪製平滑分布估計圖
        sns.kdeplot(df2[feature], bw=0.5,label=label2)
        plt.xlabel(feature, fontsize=9)
        locs, labels = plt.xticks()
        plt.tick_params(axis='x', which='major', labelsize=6, pad=-6)
        plt.tick_params(axis='y', which='major', labelsize=6)
    plt.show();

```

```python=
t0 = train_df.loc[train_df['target'] == 0]
t1 = train_df.loc[train_df['target'] == 1]
features = train_df.columns.values[2:102]
plot_feature_distribution(t0, t1, '0', '1', features)
```
![](https://i.imgur.com/Cpl6q3z.png)

```python=
features = train_df.columns.values[102:202]
plot_feature_distribution(t0, t1, '0', '1', features)
```
![](https://i.imgur.com/p92Y4qV.png)

OK，200個欄位/特徵在目標為0及1的表現都呈現在上面，看得真讓人眼花撩亂~~然後想立刻關螢幕~~。
雖然說這些圖實在讓人覺得很阿砸，但是這些圖其實沒有這麼困難，仔細看一下不管 target是0還是1，大部分特徵的分布都是幾乎一樣的。不過就是有這麼幾個( **var_0, var_1, var_2, var_5, var_9, var_13, var_106, var_109, var_139 ...** )在target不同下的分布明顯的不一樣，甚至其中少數幾個( **var_2, var_13, var_26, var_55, var_175, var_184, var_196** )更呈現了類似二元隨機變量的分布圖 (?) 。而這些資訊都可提供為接下來的特徵選擇上的參考指標。

在訓練資料中，target 0或是1的特徵分布我們看完了，再來要對照一下訓練資料跟測試資料的特徵分部有沒有什麼不同。

```python=
features = train_df.columns.values[2:102]
plot_feature_distribution(train_df, test_df, 'train', 'test', features)
```
![](https://i.imgur.com/1dpxjZq.png)

```python=
features = train_df.columns.values[102:202]
plot_feature_distribution(train_df, test_df, 'train', 'test', features)
```
![](https://i.imgur.com/jEq3j6f.png)

我們可以發現到，儘管在上面我們提到的幾個"特別"的特徵分布上，訓練資料與測試資料上的表現幾乎呈現相同的狀況。


#### Distribution of mean and std

首先，先來看看均值(mean)的分布
```python=
plt.figure(figsize=(16,6))
features = train_df.columns.values[2:202]
plt.title("Distribution of mean values per row in the train and test set")
sns.distplot(train_df[features].mean(axis=1),color="green", kde=True,bins=120, label='train')
sns.distplot(test_df[features].mean(axis=1),color="blue", kde=True,bins=120, label='test')
plt.legend()
plt.show()
```
![](https://i.imgur.com/7v0Vu7i.png)

```python=
plt.figure(figsize=(16,6))
plt.title("Distribution of mean values per column in the train and test set")
sns.distplot(train_df[features].mean(axis=0),color="magenta",kde=True,bins=120, label='train')
sns.distplot(test_df[features].mean(axis=0),color="darkblue", kde=True,bins=120, label='test')
plt.legend()
plt.show()
```
![](https://i.imgur.com/155SSAg.png)

```python=
plt.figure(figsize=(16,6))
plt.title("Distribution of std values per row in the train and test set")
sns.distplot(train_df[features].std(axis=1),color="black", kde=True,bins=120, label='train')
sns.distplot(test_df[features].std(axis=1),color="red", kde=True,bins=120, label='test')
plt.legend();plt.show()
```
![](https://i.imgur.com/6YPVZIX.png)


```python=
plt.figure(figsize=(16,6))
plt.title("Distribution of std values per column in the train and test set")
sns.distplot(train_df[features].std(axis=0),color="blue",kde=True,bins=120, label='train')
sns.distplot(test_df[features].std(axis=0),color="green", kde=True,bins=120, label='test')
plt.legend(); plt.show()
```
![](https://i.imgur.com/ydSQ9Zi.png)

很好，上面四張圖大概又要讓人崩潰了。
其實只要看懂上面兩張，下面兩張只是統計量不同罷了。

(為了排版及整體性考量，我把解釋放在註解[^1])

[^1]:一樣都是均值(標準差)的分布，這兩張有什麼不同?
想要知道差別，我們要先了解code裡面`train_df[features]`以及`train_df[features].mean(axis=1)`各代表什麼意思。
    ![](https://i.imgur.com/cvleG1R.png)
    `train_df[features]`就是把原先的表格拿掉ID跟target單純以特徵呈現


    ![](https://i.imgur.com/dp00T6D.png)

    ![](https://i.imgur.com/aPOZIyq.png)
    上面兩張，可以知道axis參數代表的就是計算均值(標準差)的"方向"。當axis=0時，計算的方向便是以行(垂直)的方式計算，而當axis=1時，便是以列(水平)做計算方向。
    
    * axis=0 : 呈現各別特徵的平均(標準差)
    * axis=1 : 表示每一筆資料，所有特徵值的平均(標準差)

除了拿Training Data 與 Test Data來比較外，我們也可以比較Training Data內不同Target的均值(標準差)有什麼不同。

```python=
t0 = train_df.loc[train_df['target'] == 0]
t1 = train_df.loc[train_df['target'] == 1]
plt.figure(figsize=(16,6))
plt.title("Distribution of mean values per row in the train set")
sns.distplot(t0[features].mean(axis=1),color="red", kde=True,bins=120, label='target = 0')
sns.distplot(t1[features].mean(axis=1),color="blue", kde=True,bins=120, label='target = 1')
plt.legend(); plt.show()
```
![](https://i.imgur.com/t7qPPi3.png)

```python=
plt.figure(figsize=(16,6))
plt.title("Distribution of mean values per column in the train set")
sns.distplot(t0[features].mean(axis=0),color="green", kde=True,bins=120, label='target = 0')
sns.distplot(t1[features].mean(axis=0),color="darkblue", kde=True,bins=120, label='target = 1')
plt.legend(); plt.show()
```
![](https://i.imgur.com/dlOuPNI.png)

以上面這幾張圖我們可以將各種統計量( **min,Max,偏態skewness ,峰度Kurtosis** )依樣畫葫蘆般的通通做過一遍，在此請容許小弟直接省略。

> Gabriel Preda在這一個部分上面，除了最小值的圖呈現比較明顯的右偏，基本上並沒有對這一連串的視覺化做出解釋及處理，那我們可以思考的是，這一連串的觀察中，圖形可能會出現哪些狀況?而我們又有哪些必要處理的部分?怎麼處理?
> 

做到這邊，冗長的Data Exploration總算告一段落，下一部分要面對的就是Feature Engineering以及 run Model 惹~

### To be Continue....