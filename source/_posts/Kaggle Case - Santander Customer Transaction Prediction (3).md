---
title: Kaggle Case - Santander Customer Transaction Prediction (3)
date: 2019-10-08 02:08:06
categories:
- 案例討論 Case Study
image: https://miro.medium.com/max/1188/0*GPWsCoea-g1npXqH.png
mathjax: true
---

> 本篇為此系列文中第三篇，若還沒有讀過前文的讀者，建議從" [Kaggle Case - Santander Customer Transaction Prediction (1)](https://hackmd.io/s/B1YMZu6wN) "開始閱讀唷 !


<!-- more -->

## Feature Correlation

雖說Feature Correlation 以及下一部分的 Duplicate values 在Kernel作者的觀點都還算是 Data Exploration的一部分，但是畢竟它已經跨到特徵處理了，我就把這兩部分併到這邊一起來看。

```python=
%%time
correlations = train_df[features].corr().abs().unstack().sort_values(kind="quicksort").reset_index()
# DataFrame.倆倆算相關係數.取絕對值.排列(?).排序.重新編號
correlations = correlations[correlations['level_0'] != correlations['level_1']]
correlations.head(10)
```
在這裡，我嘗試過中間的`.unstack()`替換成`.stack()`似乎不會影響這部分特徵相關係數的排序，但是不管何者，都不能省略，它是將`train_df[features].corr().abs()`這個DataFrame變成可排序的一個重要的媒介。[^註1]

```python=
correlations.tail(10)
```

```python=
correlations.head(10)
```

上面輸出的結果(略)，可以發現到除了特徵自己本身的相關係數為1外，其他不同的特徵配對的相關性都非常低 ( 最高也差不多千分之9.9左右 )

> 一樣的，我還是想問，如若有一些特徵的配對呈現高度相關性，那麼這些特徵代表的意義是什麼?
> 




## Duplicate values

```python=
%%time
features = train_df.columns.values[2:202]
unique_max_train = []
unique_max_test = []
for feature in features:
    values = train_df[feature].value_counts()  
    #在每一個feature中找出重複的次數，return 重複的數值及重複的次數
    unique_max_train.append([feature, values.max(), values.idxmax()])
    #value.max()重複最多的次數 ； value.idmax()重複最多次的數值
    values = test_df[feature].value_counts()
    unique_max_test.append([feature, values.max(), values.idxmax()])
```


```python=
np.transpose((pd.DataFrame(unique_max_train, columns=['Feature', 'Max duplicates', 'Value'])).\
            sort_values(by = 'Max duplicates', ascending=False).head(15))
```
![](https://i.imgur.com/eq96nyK.png)
```python=
np.transpose((pd.DataFrame(unique_max_test, columns=['Feature', 'Max duplicates', 'Value'])).\
            sort_values(by = 'Max duplicates', ascending=False).head(15))
```
![](https://i.imgur.com/fGgEufQ.png)

從這裡結果我們可以觀察到，在訓練資料及測試資料中，在許多特徵中，某些特定 ( 相近 ) 的數值  重複的次數也非常接近，這是個蠻有趣的現象，也或許在後面的特徵工程中會有所用 ( ? )

## Feature Engineering

```python=
%%time
idx = features = train_df.columns.values[2:202]
for df in [test_df, train_df]:
    df['sum'] = df[idx].sum(axis=1)  
    df['min'] = df[idx].min(axis=1)
    df['max'] = df[idx].max(axis=1)
    df['mean'] = df[idx].mean(axis=1)
    df['std'] = df[idx].std(axis=1)
    df['skew'] = df[idx].skew(axis=1)
    df['kurt'] = df[idx].kurtosis(axis=1)
    df['med'] = df[idx].median(axis=1)
```

```python=
train_df[train_df.columns[202:]].head()
```

```python=
test_df[test_df.columns[201:]].head()
```

Gabriel Preda在特徵的處理上面，選擇的是加進統計量當作新的特徵，而不對就有特徵進行處理，有關於 Feature Engineering 的處理選擇，往往來自於前面對於資料的探索加上後面要用的模型來決定，此處，Gabriel Preda並沒有對於為何要這樣處理特徵多做解釋，或許有一部分也是取決 lightGBM 的特性 (有待查證) ?

```python=
def plot_new_feature_distribution(df1, df2, label1, label2, features):
    i = 0
    sns.set_style('whitegrid')
    plt.figure()
    fig, ax = plt.subplots(2,4,figsize=(18,8))

    for feature in features:
        i += 1
        plt.subplot(2,4,i)
        sns.kdeplot(df1[feature], bw=0.5,label=label1)
        sns.kdeplot(df2[feature], bw=0.5,label=label2)
        plt.xlabel(feature, fontsize=11)
        locs, labels = plt.xticks()
        plt.tick_params(axis='x', which='major', labelsize=8)
        plt.tick_params(axis='y', which='major', labelsize=8)
    plt.show();
```

```python=
t0 = train_df.loc[train_df['target'] == 0]
t1 = train_df.loc[train_df['target'] == 1]
features = train_df.columns.values[202:]
plot_new_feature_distribution(t0, t1, 'target: 0', 'target: 1', features)
```

![](https://i.imgur.com/buKRzve.png)


```python=
features = train_df.columns.values[202:]
plot_new_feature_distribution(train_df, test_df, 'train', 'test', features)
```
![](https://i.imgur.com/tlA6vNr.png)



```python=
print('Train and test columns: {} {}'.format(len(train_df.columns), len(test_df.columns)))
```
![](https://i.imgur.com/AA3nNNN.png)

上面這些圖看到不用太驚恐，其實只是把上一篇的概念用在新的特徵上面，看看這些新特徵的分布是否有其一致性。而最後我們的測試資料及訓練資料都各增加了八個新特徵。


## Model

首先，將原本的資料做一個修改 : 拿掉ID，並將 target獨立出來

```python=
features = [c for c in train_df.columns if c not in ['ID_code', 'target']]
target = train_df['target']
```

設置模型參數
```python=
param = {
    'bagging_freq': 5,
    'bagging_fraction': 0.4,
    'boost_from_average':'false',
    'boost': 'gbdt',
    'feature_fraction': 0.05,
    'learning_rate': 0.01,
    'max_depth': -1,  
    'metric':'auc',
    'min_data_in_leaf': 80,
    'min_sum_hessian_in_leaf': 10.0,
    'num_leaves': 13,
    'num_threads': 8,
    'tree_learner': 'serial',
    'objective': 'binary', 
    'verbosity': 1
}
```

最重要的就是 Run Model

```python=
%%time
folds = StratifiedKFold(n_splits=10, shuffle=False, random_state=44000)
oof = np.zeros(len(train_df))
predictions = np.zeros(len(test_df))
feature_importance_df = pd.DataFrame()

for fold_, (trn_idx, val_idx) in enumerate(folds.split(train_df.values, target.values)):
    print("Fold {}".format(fold_))
    trn_data = lgb.Dataset(train_df.iloc[trn_idx][features], label=target.iloc[trn_idx])
    val_data = lgb.Dataset(train_df.iloc[val_idx][features], label=target.iloc[val_idx])

    num_round = 1000000
    clf = lgb.train(param, trn_data, num_round, valid_sets = [trn_data, val_data], verbose_eval=1000, early_stopping_rounds = 3000)
    oof[val_idx] = clf.predict(train_df.iloc[val_idx][features], num_iteration=clf.best_iteration)
    
    fold_importance_df = pd.DataFrame()
    fold_importance_df["Feature"] = features
    fold_importance_df["importance"] = clf.feature_importance()
    fold_importance_df["fold"] = fold_ + 1
    feature_importance_df = pd.concat([feature_importance_df, fold_importance_df], axis=0)
    
    predictions += clf.predict(test_df[features], num_iteration=clf.best_iteration) / folds.n_splits

print("CV score: {:<8.5f}".format(roc_auc_score(target, oof)))
```

![](https://i.imgur.com/yy03rsH.png)

真的不誇張，我也是花了30幾分鐘才跑完這10個folds的驗證過程，經過這一系列的處理，這一次的CV Score可以達到90%，著實不低。

> 不過其實說真的，這個Case其實並沒有太多的處理，一方面是資料本身還算蠻乾淨，二來我想也是模型本身的原因。

最後，把特徵的重要性視覺化

```python=
cols = (feature_importance_df[["Feature", "importance"]]
        .groupby("Feature")
        .mean()
        .sort_values(by="importance", ascending=False)[:150].index)
best_features = feature_importance_df.loc[feature_importance_df.Feature.isin(cols)]

plt.figure(figsize=(14,28))
sns.barplot(x="importance", y="Feature", data=best_features.sort_values(by="importance",ascending=False))
plt.title('Features importance (averaged/folds)')
plt.tight_layout()
plt.savefig('FI.png')
```
![](https://i.imgur.com/fd7I6lH.png)

最後，將結果輸出成CSV檔上傳就可以即時獲得名次

```python=
sub_df = pd.DataFrame({"ID_code":test_df["ID_code"].values})
sub_df["target"] = predictions
sub_df.to_csv("submission.csv", index=False)
```


## 後記

這一篇 kernel 在kaggle上面獲得了500多的按讚，應該會是在這題所有分享kernel裡面的最高分，說真的，的確實至名歸，在講解以及整個處理的流程都非常的順暢。

如果要拿這一篇當作第一篇kaggle Study的題材也算是蠻適合的，可以了解到流程，也可以知道Kaggler到底都在幹些什麼事情。

不過，簡單易懂也有相對來說的可惜，可惜並沒有遇到比較接近現實的資料狀況 ( EX: 缺失值、異常值、特徵的選擇及縮放.... )，不過對於第一次接觸的新手來說，如果一開始就這麼刺激，~~可能看完就放棄了~~XDDD


註釋
---
[^註1]:
這個部份我想了有一段時間，若將`.unstack()`整個拿掉則會出現Traceback,主要還是跟`train_df[features].corr().abs()`的型態有關

![](https://i.imgur.com/pv7OUwJ.png)

這樣的表格其實無法排序，所以`.unstack()`可以說是將整個表格換做可以排序的方式

![](https://i.imgur.com/hjnxrvN.png)