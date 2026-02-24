---
title: "Unsupervised Learning (4) -- Neighbor Embedding"
date: 2019-10-08 00:44:38
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning
image: https://i.imgur.com/0J9l8ZP.png
mathjax: true
---

有了前面 Word Embedding 的概念後，我們可以把問題想得更 generalization 一些 : 「面對流形 ( Manifold )或其他 非線性 ( Non-Linear ) 的狀況，我們是否能嵌入 (Embedding) 到更低維度的空間中並且保留其特性呢 ?」

<!-- more -->

<img width=500 src="https://i.imgur.com/0J9l8ZP.png" >

## Locally Linear Embedding ( LLE )

LLE 的核心概念是這樣 ，原本的資料中，任一筆資料 $x_i$ 假設可由周圍的 資料點 $x_j$ 線性組合而來，其係數 $w_{ij}$ 即為 $x_i$ 與 $x_j$ 之間的關係 (relation)。而我希望 Embedding 後的 $z_i$ 與 $z_j$ 仍保有 $w_{ij}$ 這樣的關係，藉此來找出 $z_i$ 與 $z_j$。

![](https://i.imgur.com/F6ko75K.png)

要解決這樣的問題，可以經由兩段最優化過程而得 : 

Step 1 : 周圍 $k$ 個資料點線性組合後的向量與 $x_i$ 的差距 (2-norm) 最小化找出 $w_{ij}$

Step 2 : 利用周圍 $k$ 個資料點線性組合 ( 共用相同的 $w_{ij}$ ) 後的向量與 $z_i$ 的差距 (2-norm) 最小化找出 $z$-set

轉換的優劣其實取決於 $k$ 值。

![](https://i.imgur.com/Iwp78WR.png)

當我們 $k$ 取的太大、太小時，都很容易造成轉換後的資料有不同形態的重疊。$k$ 值的選取仍要視資料的分布狀況而定。



![](https://i.imgur.com/hFBtzaq.jpg =250x)



## Laplacain Eigenmap

我們也可以利用於 " [Semi-supervised Learning](http://bit.ly/2MoSVHB) " 中所提到的 Graph Theory 來對非線性資料做轉換。

![](https://i.imgur.com/hw07j2V.jpg)

一但我們將資料利用 Graph Theory 的方式連結成為 Graph 後，我們便可以計算其 $Smoothness$ and $Laplacian\ matrix$。

但這裡是 Unsupervised Learning ，所以我們要將原本的 Loss Function 

$$
L=\sum\limits_{x_r}C(y_r,\hat{y_r})+\lambda S
$$

去掉 Supervied 項

$$
L=\lambda S=\lambda\sum\limits_{i,j}w_{ij}\|z_i-z_j\|_2
$$

但又因為沒有了 Supervised 項，這會導致我們在進行最小化的同時，或使得最後的解必然會是 $z_i=z_j=0$ ，因此我們必須要另外給予一個 Constrain : 

$$
Span\left\{z_1,z_2,\cdots,z_N\right\}=R^M,\ where\ dim(Z)=M
$$

這其實在講說，當我們投射到 Z 空間時，這個 Z 空間必然是「最小」的空間。

## T-Distributed Stochastic Neighbor Embedding (t-SNE)

從 LLE 的演算過程可以看出來，它要做的事情是，把距離近的資料點，經過轉換後可以依然保持得很近。但是它卻沒有確保原本距離遠的資料點經過轉換後會一樣離的很遠。

因此，很多時候利用 LLE 轉換後，容易有大量重疊的現象發生。

![](https://i.imgur.com/g2nZn5j.png)

而 t-SNE 則是試圖解決這樣的問題的一種方法。
它的概念簡單來說就是，原本的分布，經過轉換後，分布狀況要盡可能的相同。

首先我們要先對轉換前後的資料空間定義出距離 ( 在這裡，我們將距離定義成為相似度 ) : 


$$
S(x_i,x_j)=e^{-\|x_i-x_j\|_2} \\
S'(z_i,z_j)=\displaystyle{\frac{1}{1+\|z_i-z_j\|_2}}
$$

利用轉換前後空間不同的距離定義，來使資料間遠近能在轉換後更加明顯。

![](https://i.imgur.com/xmUSJVK.png)

再來便可以定義轉換前後的分布為

$$
P(x_j|x_i)=\displaystyle{\frac{S(x_i,x_j)}{\sum\limits_{k\neq i}S(x_i,x_k)}}\\
Q(x_j|x_i)=\displaystyle{\frac{S'(z_i,z_j)}{\sum\limits_{k\neq i}S'(z_i,z_k)}}
$$

一切就緒後，Loss function 便為兩個分布之間的差距，在數學上使用 KL 散度 ( 相對熵 )來計算，接著再找最優解即可 : 

$$
L=\sum\limits_{i}KL\Big(P(*|x_i)\|Q(*|z_i)\Big)=\sum\limits_{i}\sum\limits_{j}P(x_j|x_i)\log\displaystyle{\frac{P(x_j|x_i)}{Q(z_j|z_i)}}
$$

經過 t-SNE 處理，很明顯的改善了 LLE 的問題

![](https://i.imgur.com/0urcL31.png)

各個類別，經過轉換後都可以將各群距離拉開。