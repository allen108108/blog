---
title: "[文章] Building a Content Based Recommender System for Hotels in Seattle" 
date: 2019-10-08 01:44:48
categories:
- 文章 Article
image: https://miro.medium.com/max/1600/1*o2BAwSt9etiRqO1nZyMDHg.jpeg
mathjax: true
---

[Susan Li](https://towardsdatascience.com/@actsusanli) .(2019 , Apr 1). *Building a Content Based Recommender System for Hotels in Seattle* .Retrieved June 27,2019 ,from https://towardsdatascience.com/building-a-content-based-recommender-system-for-hotels-in-seattle-d724f0a32070

<!-- more -->

推薦系統 ( Recommendation system )有一個眾所周知的問題，便是在 Cold Start Problem 上推薦系統無法有效地將推薦項目推送給使用者。在新用戶、新產品或是新網站、平台上，由於沒有足夠多的資料，推薦系統很難建構一個適用的 model 來進行推薦。

此篇文章，作者 Susan Li 使用了 Content-Based Filter[^註1] 來做為解決 Cold Start Problem 的方法，Content-Based Recommendation System 可以適用於各種不同的領域，而且沒有 Cold Start Problem，在產品、網站一開始上線後就可以做出有效的推薦。



## Scenario

作者現在模擬一個情境，我們是一個新的 Online Travel Agency ( OTA ，類似於台灣易遊網、Hotels.com...)，且有數千家飯店旅館會在我們的平台銷售。由於我們是新的平台，並沒有太多用戶資料，我們要建立一套 Content-Based Recommendation System ，利用飯店本身的商品陳述來判斷是否符合使用者需求而進行精準的推薦投放。

如何判斷用戶的需求 ? 作者使用了 Cosine Similarity[^註2] 來將我們的資料與用戶所預定、瀏覽的酒店旅館進行 Cosine Similarity 計算，針對其值進行推薦酒店的排序及投放。


## The Data

由於是虛擬情境，作者本身並沒有這些公共酒店旅館的資料，因此作者自行從西雅圖各飯店網站進行資料收集，一共收集了150多家的飯店資料，其中包含名稱、地址以及飯店描述。這些資料都可以在作者 **[Github](https://github.com/susanli2016/Machine-Learning-with-Python/blob/master/Seattle_Hotels.csv)** <i class="fa fa-github"></i> 上取得。



```python=
import pandas as pd
import numpy as np

from nltk.corpus import stopwords
import nltk
nltk.download('stopwords')

from sklearn.metrics.pairwise import linear_kernel
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.decomposition import LatentDirichletAllocation

import re
import random

import plotly.graph_objs as go
import plotly.plotly as py
import cufflinks
pd.options.display.max_columns = 30
from IPython.core.interactiveshell import InteractiveShell
import plotly.figure_factory as ff
InteractiveShell.ast_node_interactivity = 'all'
from plotly.offline import iplot
cufflinks.go_offline()
cufflinks.set_config_file(world_readable=True, theme='solar')
```

```python=
df = pd.read_csv('Seattle_Hotels.csv', encoding="latin-1")
df.head()
print('We have ', len(df), 'hotels in the data')
```

![](https://i.imgur.com/DdkI2Id.png)
![](https://i.imgur.com/WR6TKVh.png =190x)

```python=
def print_description(index):
    example = df[df.index == index][['desc', 'name']].values[0]
    if len(example) > 0:
        print(example[0])
        print('Name:', example[1])
```

```python=
print_description(10)
```

![](https://i.imgur.com/uHtBp2Z.png)

```python=
print_description(100)
```

![](https://i.imgur.com/IciTP1E.png)


## EDA

首先主要針對各個描述中的共同字彙進行統計，並對照 `stop_words` 刪去前後的字彙進行比較。

sklearn 中的 `CountVecorizer` 可以協助我們對於文本中的字彙進行分詞以及計算詞頻。但這樣的方法，有很大的侷限性，因為我們並不考慮上下文的關係，僅針對字彙進行詞頻計算，因此容易失去文本的語意。

### Visualize Token (vocabulary) Frequency Distribution Before Removing Stop Words

```python=
def get_top_n_words(corpus, n=None):
    vec = CountVectorizer().fit(corpus)
    # bag_of_words 傳回的是一個稀疏矩陣
    bag_of_words = vec.transform(corpus)
    # 字彙在所有文本內出現的總次數，sum_words傳回的是一個一列的矩陣
    sum_words = bag_of_words.sum(axis=0)  
    # vec.vocabulary_.items()傳回的是 tuple (word,index)
    words_freq = [(word, sum_words[0, idx]) for word, idx in vec.vocabulary_.items()]
    words_freq =sorted(words_freq, key = lambda x: x[1], reverse=True)
    return words_freq[:n]
    
common_words = get_top_n_words(df['desc'], 20)
df1 = pd.DataFrame(common_words, columns = ['desc' , 'count'])
df1.groupby('desc').sum()['count'].sort_values().iplot(kind='barh', yTitle='Count', linecolor='black', title='Top 20 words in hotel description before removing stop words')
```

![](https://i.imgur.com/KOhNIKV.png)



### Visualize Token (vocabulary) Frequency Distribution After Removing Stop Words

停用詞 ( Stop words ) 指的其實就是很頻繁會用到的詞彙，對於文本分析而言並沒有太大的幫助，因此在處理文本分析的過程中，通常會將其過濾掉。

```python=
def get_top_n_words(corpus, n=None):
    # 濾掉英文的停用詞
    vec = CountVectorizer(stop_words='english').fit(corpus)
    bag_of_words = vec.transform(corpus)
    sum_words = bag_of_words.sum(axis=0) 
    words_freq = [(word, sum_words[0, idx]) for word, idx in vec.vocabulary_.items()]
    words_freq =sorted(words_freq, key = lambda x: x[1], reverse=True)
    return words_freq[:n]
    
common_words = get_top_n_words(df['desc'], 20)
df2 = pd.DataFrame(common_words, columns = ['desc' , 'count'])
df2.groupby('desc').sum()['count'].sort_values().iplot(kind='barh', yTitle='Count', linecolor='black', title='Top 20 words in hotel description after removing stop words')
```

![](https://i.imgur.com/h2rvA3O.png)


### Bigrams Frequency Distribution Before Removing Stop Word

N-Gram 是一種以統計為基礎語言模型的演算法，基本上就是以長度為 N 的滑窗對文本進行掃描，形成一個個長度為 N 的字段。而最常用的就是二元的 bigram 以及三元的 trigram。

在`CountVectorizer` 裡面的參數 `ngram_rang=(min_n,max_n)` 指的就是我們要使用的 n-gram 的最小、最大 n 值。`ngram_rang=(2,2)` 就代表 n=2 ，亦即我們要使用的是 bigram。

```python=
def get_top_n_bigram(corpus, n=None):
    vec = CountVectorizer(ngram_range=(2, 2)).fit(corpus)
    bag_of_words = vec.transform(corpus)
    sum_words = bag_of_words.sum(axis=0) 
    words_freq = [(word, sum_words[0, idx]) for word, idx in vec.vocabulary_.items()]
    words_freq =sorted(words_freq, key = lambda x: x[1], reverse=True)
    return words_freq[:n]
    
common_words = get_top_n_bigram(df['desc'], 20)
df3 = pd.DataFrame(common_words, columns = ['desc' , 'count'])
df3.groupby('desc').sum()['count'].sort_values(ascending=False).iplot(kind='bar', yTitle='Count', linecolor='black', title='Top 20 bigrams in hotel description before removing stop words')
```

![](https://i.imgur.com/2bNKk53.png)

### Bigrams Frequency Distribution After Removing Stop Word


```python=
def get_top_n_bigram(corpus, n=None):
    vec = CountVectorizer(ngram_range=(2, 2), stop_words='english').fit(corpus)
    bag_of_words = vec.transform(corpus)
    sum_words = bag_of_words.sum(axis=0) 
    words_freq = [(word, sum_words[0, idx]) for word, idx in vec.vocabulary_.items()]
    words_freq =sorted(words_freq, key = lambda x: x[1], reverse=True)
    return words_freq[:n]
    
common_words = get_top_n_bigram(df['desc'], 20)
df4 = pd.DataFrame(common_words, columns = ['desc' , 'count'])
df4.groupby('desc').sum()['count'].sort_values(ascending=False).iplot(kind='bar', yTitle='Count', linecolor='black', title='Top 20 bigrams in hotel description After removing stop words')
```

![](https://i.imgur.com/zMKCOL7.png)


### Trigrams Frequency Distribution Before Removing Stop Word

```python=
def get_top_n_trigram(corpus, n=None):
    vec = CountVectorizer(ngram_range=(3, 3)).fit(corpus)
    bag_of_words = vec.transform(corpus)
    sum_words = bag_of_words.sum(axis=0) 
    words_freq = [(word, sum_words[0, idx]) for word, idx in vec.vocabulary_.items()]
    words_freq =sorted(words_freq, key = lambda x: x[1], reverse=True)
    return words_freq[:n]
    
common_words = get_top_n_trigram(df['desc'], 20)
df5 = pd.DataFrame(common_words, columns = ['desc' , 'count'])
df5.groupby('desc').sum()['count'].sort_values(ascending=False).iplot(kind='bar', yTitle='Count', linecolor='black', title='Top 20 trigrams in hotel description before removing stop words')
```

![](https://i.imgur.com/7jzxs8I.png)


### Trigrams Frequency Distribution After Removing Stop Word

```python=
def get_top_n_trigram(corpus, n=None):
    vec = CountVectorizer(ngram_range=(3, 3), stop_words='english').fit(corpus)
    bag_of_words = vec.transform(corpus)
    sum_words = bag_of_words.sum(axis=0) 
    words_freq = [(word, sum_words[0, idx]) for word, idx in vec.vocabulary_.items()]
    words_freq =sorted(words_freq, key = lambda x: x[1], reverse=True)
    return words_freq[:n]
    
common_words = get_top_n_trigram(df['desc'], 20)
df6 = pd.DataFrame(common_words, columns = ['desc' , 'count'])
df6.groupby('desc').sum()['count'].sort_values(ascending=False).iplot(kind='bar', yTitle='Count', linecolor='black', title='Top 20 trigrams in hotel description after removing stop words')
```

![](https://i.imgur.com/c2qAdZs.png)

從上圖可以看出 `pike place market` 幾乎一半以上的飯店都會提到，因為這是一個公共的農產貿易市場，也是熱門的觀光景點，因此也會成為各個旅館的重點宣傳之一。

### Hotel Description Length Distribution

```python=
df['word_count'] = df['desc'].apply(lambda x: len(str(x).split()))

desc_lengths = list(df['word_count'])

print("Number of descriptions:",len(desc_lengths),
      "\nAverage word count", np.average(desc_lengths),
      "\nMinimum word count", min(desc_lengths),
      "\nMaximum word count", max(desc_lengths))
```
![](https://i.imgur.com/jce2019.png)


```python=
df['word_count'].iplot(
    kind='hist',
    bins = 50,
    linecolor='black',
    xTitle='word count',
    yTitle='count',
    title='Word Count Distribution in Hotel Description')
```

![](https://i.imgur.com/LvaXxit.png)


### Preprocessing hotel description text

這一個部分，由於所有的資料都是作者特別去收集的，所以資料本身要做的 cleaning 動作並不多，資料本身不會有太多的 outlier 。

```python=
REPLACE_BY_SPACE_RE = re.compile('[/(){}\[\]\|@,;]')
BAD_SYMBOLS_RE = re.compile('[^0-9a-z #+_]')
STOPWORDS = set(stopwords.words('english'))

def clean_text(text):
    """
        text: a string
        
        return: modified initial string
    """
    text = text.lower() # lowercase text
    text = REPLACE_BY_SPACE_RE.sub(' ', text) # replace REPLACE_BY_SPACE_RE symbols by space in text. substitute the matched string in REPLACE_BY_SPACE_RE with space.
    text = BAD_SYMBOLS_RE.sub('', text) # remove symbols which are in BAD_SYMBOLS_RE from text. substitute the matched string in BAD_SYMBOLS_RE with nothing. 
    text = ' '.join(word for word in text.split() if word not in STOPWORDS) # remove stopwors from text
    return text
    
df['desc_clean'] = df['desc'].apply(clean_text)
```

```python=
def print_description(index):
    example = df[df.index == index][['desc_clean', 'name']].values[0]
    if len(example) > 0:
        print(example[0])
        print('Name:', example[1])

print_description(10)
```

![](https://i.imgur.com/MtUA4ZT.png)

```python=
print_description(100)
```

![](https://i.imgur.com/EtUqVEQ.png)

## Modeling

* 對每一間飯店，創造 unigram, bigram, and trigram 的 TF-IDF matrix
* 計算所有飯店的相似性
* 定義一個函數，當我們輸入一間飯店名稱時，可以輸出前十名推薦飯店名單。

這裡要特別提到的就是 TF-IDF (Term Frequency - InverseDocument Frequency),這是一個常用在資料探勘的加權技術。

假設某一個字(詞)彙 $W_i$ 在某一篇文章 $A_j$ 中出現了 $n_{ij}$ 次，那麼

$$
TF_{ij}=\displaystyle{\frac{n_{ij}}{\sum\limits_{k}n_{kj}}}
$$  
又假設我們一共有 $\mid D\mid$ 個文本資料，此字(詞)彙 $W_i$ 在其中 $D_i$ 個文本都有出現過
$$
IDF_i=\log\displaystyle{\frac{\mid D\mid}{\mid D_i\mid}}
$$

從上面的定義我們可以發現如果有一個詞彙在某個特定文本出現比例極高 ( 高 $TF_{ij}$ )，但卻在所有文本中其他文本出現的比例很少 ( 高 $IDF_i$ )，則 $TF_{ij}\times IDF_i$ 即可產出很高的權重。所以利用 $TF_{ij}\times IDF_i$ 可篩選出足以進行判別的詞彙。



```python=
# 將index改為飯店名稱
df.set_index('name', inplace = True)
```

```python=
tf = TfidfVectorizer(analyzer='word', ngram_range=(1, 3), min_df=0, stop_words='english')
# tfidf_matrix列出各文本內所有字的TFIDF值，且同時進行 normalized
# 第 i 列第 j 行的元素就是代表 詞彙 j 在 文本 i 的 TFIDF value 
tfidf_matrix = tf.fit_transform(df['desc_clean'])
# 第 i 列第 j 行的元素就是代表 詞彙 i 與 詞彙 j 的 cosine value
cosine_similarities = linear_kernel(tfidf_matrix, tfidf_matrix)
```
這裡值得注意的是，`cosine Similarity` 在此處等同於 `liner_kernel`

$$
\cos(x,y)=\displaystyle{\frac{x^Ty}{\|x\|\|y\|}}
$$

$$
linear\_kernel (x,y)=x^Ty
$$

因為 `TfidfVectorizer` 有做 normalization ，所以長度均為 1，也因此兩者其實是一樣的東西 ( 就運算速度來說，linear_kernel 會快一些 )。

```python=
indices = pd.Series(df.index)

indices[:50]
```

![](https://i.imgur.com/XUIiSfQ.png)



```python=
def recommendations(name, cosine_similarities = cosine_similarities):
    
    recommended_hotels = []
    
    # gettin the index of the hotel that matches the name
    # 找出輸入飯店名稱的 index
    idx = indices[indices == name].index[0]

    # creating a Series with the similarity scores in descending order
    # 利用輸入飯店index，從 cosine similarity 矩陣找出這間飯店與其他飯店的 cosine similarity value 並且排序
    score_series = pd.Series(cosine_similarities[idx]).sort_values(ascending = False)

    # getting the indexes of the 10 most similar hotels except itself
    # 從這些排序後的 cosine similarity 取前10
    top_10_indexes = list(score_series.iloc[1:11].index)
    
    # populating the list with the names of the top 10 matching hotels
    for i in top_10_indexes:
        recommended_hotels.append(list(df.index)[i])
        
    return recommended_hotels
```


## Recommendations

試著輸入 " Hilton Seattle Airport & Conference Center " 來看看推薦名單。

```python=
recommendations('Hilton Seattle Airport & Conference Center')
```

![](https://i.imgur.com/7SpTy3m.png)

從我們輸出的推薦名單對比我們從 google 搜尋得到的推薦，有 3/4 的重疊。

![](https://i.imgur.com/L9q3u5h.png)

以下是 tripadvisor 的推薦，與我們的推薦名單也相似。

![](https://i.imgur.com/9aLzrUf.png)

換一個輸入來搜尋看看 " The Bacon Mansion Bed and Breakfast "

```python=
recommendations("The Bacon Mansion Bed and Breakfast")
```
![](https://i.imgur.com/hOzOTEl.jpg)

比對 google 搜尋得到的推薦，仍然有高度的重疊

![](https://i.imgur.com/5RfElvE.png)

但從 tripadvisor 的推薦名單中，就與我們的推薦名單重疊度不高。

![](https://i.imgur.com/QUpPYV5.png)

註釋
---

[^註1]: 
詳細介紹可參閱 : http://recommender-systems.org/content-based-filtering/

[^註2]:
在文本分析中，我們會將所有的文字向量化，使用兩個向量夾角的 cos 值進行相似性的判別。當兩個文字向量夾角越小， cos 值越大，此兩文字相似性越高。