---
title: "Unsupervised Learning (1) -- Principle Component Analysis ( PCA )"
date: 2019-10-08 00:35:10
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning
image: https://i.imgur.com/i5jt7QR.png
mathjax: true
---

## 化繁為簡

在 " [Semi-supervised Learning](http://bit.ly/2MoSVHB) " 一文中曾經提到過化繁為簡的概念，簡單來說就是從我們手邊的資料去找出背後的規則、最精要的部分。

<!-- more -->


<img width=500 src="https://i.imgur.com/4LVhZdz.png" >


我們要問的是，是不是能從複雜、高維度的資料中，找到低維度的決定性因素可以決定這些高維度的表現 ?

<img width=500 src="https://i.imgur.com/mBUx0Ja.png" >
<img width=500 src="https://i.imgur.com/wFvzbbQ.png" >


上圖上，從這3-D流形中我們是否可找出2-D的規則 ? 
上圖下，在 MNIST 中，資料都是$28\times 28$ 的灰階圖。但我們知道，在 $28\times 28$ 的灰階圖中絕大多數的圖都會是亂碼圖，真正能形成數字的比例是極少的。是否代表我們可以縮減其維度呢 ?


## Algorithm 1 (Clustering) : K-means 


<img width=500 src="https://i.imgur.com/z0mHNSn.png" >



Step 1 : 決定我們要將資料分為 K 類 [^註1]


Step 2 : 在資料中隨機任選 K 筆做為 cluster center

Step 3 : 對所有資料點，計算他們與 cluster center 的距離，依照距離遠近進行分類

Step 4 : 針對已經分好的類別進行 cluster center 的重新分配

重複 Step 3 -- 4 直至收斂。

## Algorithm 2 (Clustering) : Hierarchical Agglomerative Clustering ( HAC )


<img width=500 src="https://i.imgur.com/Obv52sn.png" >


Step 1 : 將所有資料視為個別的cluster，再由底部倆倆合併，開始往上建立成一棵樹結構

Step 2 : 決定一個 threshold ，看要從什麼部分切開，就可以決定他們會被分做幾類。

## Algorithm 3 (Distributed Representation) : Principle Component Analysis

* Clustering : 一筆資料必須隸屬於一個 cluster
* Distributed Representation : 給出這筆資料隸屬於各分類的機率

Distributed Representation 有兩個主要的方法 : 

1. Feature Selection : 當資料會集中在某些特徵上時，我們便可以將所有維度降到只剩下這些資料集中的特徵上面。

2. Principle Component analysis : 基本的概念是將資料 ( $X$ ) 在原本的空間中，利用線性變換 ( $W$ ) 將資料變換到另外一個坐標系。在原本的空間 ( $Z$ ) 中，資料或許並不是這麼容易找出方式可以降維，因此我們利用線性變換創造出一組維度較小的空間，然後將資料轉換至其中，藉此達到降維的目的。但這裡要注意的是，經過變換後的維度，各自代表什麼意思必須經過分析後才有可能知道。

## 從最大 Variance 角度來看 PCA

### Reduce to 1-dimension


<img width=500 src="https://i.imgur.com/i5jt7QR.png" >

如果我們只考慮降到一維，直覺得我們可以知道要選擇的方向應該是 variance 最大的方向 $w^1$ (為求計算方便我們假設這是單位向量)

$$
Z=WX\Longrightarrow z_1=w^1X
$$

由於我們降到只有一維，所以 $z_1$ 為$1\times n$ 的矩陣， $w^1$ 是 $1\times d$ 的矩陣 ( $d$ 為 $X$ 的維度，$n$ 為 $X$ 的行(資料)數 )。

其實我們也可以把問題轉化成 : 找出使 $Var(z_1)$ 有最大值，且 $\|w\|_2=1$ 的 $w$

$$
w^1=arg\max_{\|w\|=1}Var(z_1)=arg\max_{\|w\|=1}\displaystyle{\frac{1}{N}}\sum\limits_{z_1}(z_1-\bar{z_1})^2
$$

### Reduce to K-dimension 

延續上面的作法找出 $w^1$ 後，我們可以找出 $w^2$，但是會多一個條件 : $w^1\cdot w^2=0$ ( 因為我們要找出的是新空間的基底，必須互相垂直 )

$$
w^2=arg\max_{\|w\|=1\\ w^1\cdot w=0}Var(z_2)=arg\max_{\|w\|=1\\ w^1\cdot w=0}\displaystyle{\frac{1}{N}}\sum\limits_{z_2}(z_2-\bar{z_2})^2
$$
$$
w^3=arg\max_{\|w\|=1\\ w^1\cdot w=0\\w^2\cdot w=0}Var(z_3)=arg\max_{\|w\|=1\\ w^1\cdot w=0\\w^2\cdot w=0}\displaystyle{\frac{1}{N}}\sum\limits_{z_3}(z_3-\bar{z_3})^2
$$
$$
\vdots\\\Downarrow
$$
$$
W=\begin{bmatrix}(w^1)^T\\(w^2)^T\\\vdots\end{bmatrix}
$$

知道了方向，我們就要去試著把這些 $\left\{w^i\right\}_{i=1}^{K}$ 給找出來。

經過一些證明[^註2]可以得到以下結論 : 
1. $w^i\ is\ the\ eigenvector\ of\ S=Cov(x)$$\ correspond\ to\ the\ i-th\ largest\ eiganvalue$
" $w^i$ 是 $S=Cov(x)$ 中第 $i$ 大特徵值對應的特徵向量 "

2. $Cov(Z)\overset{let}{=}D\ is\ a\ Diagonal\ Matrix$
" $Cov(Z)$ 是一個對角矩陣"



## 從 Sinular Value Decompposition ( SVD，奇異值分解 ) 角度來看PCA

了解了整個演算法是如何做空間轉換之後，我們其實可以從另外一個角度來理解 PCA 



![](https://i.imgur.com/kcQ0DNi.png =500x)



任何 objetct ，都可以被解釋成由很多個部分線性組合而成

$$
x\approx c_1u^1+c_2u^2+\cdots+c_Ku^K+\bar{x}\\
x-\bar{x}\approx c_1u^1+c_2u^2+\cdots+c_Ku^K 
$$
$$
x^1-\bar{x}\approx c_1^1u^1+c_2^1u^2+\cdots\\
x^2-\bar{x}\approx c_1^2u^1+c_2^2u^2+\cdots\\
\vdots
$$

如果我們將 $x-\bar{x}$ 各分量以行向量的方式組成一個 $M\times N$ 矩陣 $A$，那我們可以對矩陣 $A^TA$ 進行 SVD

$$
A^TA=U\Sigma V^T\overset{(let\ \Sigma V^T=C)}{=}UC
$$

從這個角度我們可以證明這一組 $UC$ 與 $A$ 之間有 minimize Error，而 $U$ 之行向量即為 $\left\{u^1,u^2,\cdots,u^K\right\}$，而所有的 $c_i^j$ 則構成了矩陣 $C$

![](https://i.imgur.com/Z0LN3SJ.png)

## 從 Neural Network 來看 PCA

這邊我們對參數做一點改變 ，一來是利用 Neural Network 得出來的結果並不必然會與 PCA 的結果相當，因此換個參數比較不會混淆。其次是利用我們在 NN 上面常用的參數比較容易對 NN 有概念上的連結。

$$
x\approx c_1u^1+c_2u^2+\cdots+c_Ku^K+\bar{x}\\
x-\bar{x}\approx c_1u^1+c_2u^2+\cdots+c_Ku^K\overset{let}{=}\hat{x}
$$

且令 $u^i=w^i$ ，則我們可以把上式改寫成 $\hat{x}=\sum\limits_{k=1}^{K}c_kw^k$ 而我們的目的就是希望它與 $x-\bar{x}$ 之間的 error 越來越小。因此我們可以利用 NN 進行 Gradient Descent 來縮小 error。

![](https://i.imgur.com/zhJXhTD.png)


## Weakness of PCA

* PCA 依照 Variance 來進行資料的投影，因此很可能將原本不同類別的資料 merge 在一起，從低維度的空間中來看，這些不同類別的資料將會混再一起無法分別。
* PCA 是 Linear algorithm，無法針對 non-Linear problem 進行很好的處理

## How many Principle Components ?

這跟 NN 要多少 Layers 、多少 neurons ，K-means 要取什麼 K 值一樣，都是一個 Open Question。

現今實務上常用的方式是計算特徵值的占比來決定我們要用多少的 Principle Component。

![](https://i.imgur.com/lZUQHJx.png)

我們知道，每一個 Principle Component 都是對應著一個特徵值的特徵向量，我們可以將每一個特徵值排列出來計算他們的比例。從上圖來看，我們可以依照各自的比例決定我們要的 Principle Component 個數可以取 4。



我們可以用ㄧ些實例來看看 Principle Component Analysis 的運作情況

<img width=500 src="https://i.imgur.com/lS5LdOg.png" >
<img width=500 src="https://i.imgur.com/dBKe71w.png" >



我們從這兩個實例會發現一些令人困惑的地方 : 為什麼 Principle Components 並不是一個一個的局部組成 ? 看起來仍是一個全局圖像 ?!

其實我們從 Principle Component Analysis 的演算過程可以發現這並不那麼令人意外

![](https://i.imgur.com/onZEuxo.png)

以上圖為例，參數 $a_1,a_2,\cdots$ 並不限定正實數，也有可能是負數。如果為負數，那麼各 components 也有可能有相減狀況出現，這個 「9」 也可能是 「8」 再減去一個下半圓而產生的，因此，在沒有條件限制的狀況下，這樣子的 Principle Components 的確是可以理解的。

<img width=500 src="https://i.imgur.com/QlHekOn.png" >
<img width=500 src="https://i.imgur.com/TbeLiAT.png" >


上圖我們針對參數加了非負的條件進去 (Non-Negative Matrix Factorization , NMF)，果然就有點 components 的樣子了呢 !

## Matrix Facorization

有了上面 SVD 的概念，我們可以知道任何矩陣必然可以拆成兩個矩陣相乘，這樣的分解方式，我們可以從中了解到什麼呢 ?

![](https://i.imgur.com/w5obDbK.png)

這是一個宅男收藏各動漫角色公仔的統計表，我們把這表格視為一個矩陣的話，可以將這矩陣拆成兩個矩陣的乘積

![](https://i.imgur.com/moVk5oQ.png)

這裡有趣的地方是，第一個矩陣的列向量與第二個矩陣的行向量相乘便可以得到「宅男 A」 收藏「人物 1」的公仔個數。換一個角度來看，我們可以想成「宅男 A」 與「人物 1」的潛在特徵 ( $r^A$ 與 $r^1$ ) 經過內積後可以得到收藏個數。

再來舉一個例子 

![](https://i.imgur.com/K8qQK0T.png)

林軒田在機器學習技法第十五講中提到的例子，許多的使用者對於每一部電影的評分統計，我們也可以視為一個矩陣，那當然也可以進行矩陣分解。

$$
R\approx V^TW
$$

一樣的，我們也可以將每一個評分視為每一個使用者與每一部電影的潛在特徵( 喜劇類 ? 動作片 ? 百視達 ?.....有阿湯哥 ? )內積後的結果。

問題來了，我們分解後的這些所謂的潛在特徵，其實是不能被解釋的。

![](https://i.imgur.com/iynowoB.png)

當然，我們也可以為矩陣分解設置 threshold 與 regularization

![](https://i.imgur.com/NFgfxDp.png)

矩陣分解的方式在許多的分析上面都會用到，也都有很多的變形用法，在資料分析的領域裡面這是一個非常重要的分析方式。

![](https://i.imgur.com/eXm4CfW.png)


註釋
---


[^註1]: 
如何決定 K 值?雖然說這是一個 Open Question，但也是一個 Data Driven 的問題。我們還是可以從資料的觀察中看看是否有自然的分類 ? 或是從我們要研究的問題中來決定如何取 K 值。

[^註2]: 
證明如下 :
(1)
$\because Var(z_1)=\displaystyle{\frac{1}{N}}\sum\limits_{z_1}(z_1-\bar{z_1})^2$
$=\displaystyle{\frac{1}{N}}\sum\limits_{x}(w\cdot x-w\cdot \bar{x})^2$
$=\displaystyle{\frac{1}{N}}\sum\limits_{x}(w\cdot (x- \bar{x}))^2$
$=\displaystyle{\frac{1}{N}}\sum\limits_{x}(w)^T(x- \bar{x})(x- \bar{x})^Tw$
$\Big(\because (a\cdot b)^2=(a^Tb)^2=a^Tba^Tb\ \overset{(a^Tb=scalar)}{=}\ a^Tb(a^Tb)^T=a^Tbb^Ta\Big)$
$=(w)^T=\displaystyle{\frac{1}{N}}\sum\limits_{x}(x-\bar{x})(x-\bar{x})^Tw$
$=(w)^TCov(X)w$

We want to find $w^1=arg\max\limits_{\|w\|=1}(w)^TCov(X)w$
Let $g_1(w)=(w)^TSw-\alpha\Big((w)^Tw-1\Big)$
$\Big(By\ Lagrange\ Multiplier\ Method\Big)$
$\Longrightarrow\nabla g_1(w)\overset{let}{=}0$
$\Longrightarrow Sw-\alpha w=0$
$\Longrightarrow Sw=\alpha w$
$\Longrightarrow (w)^TSw=\alpha (w)^Tw=\alpha$
$\therefore$ choose thw largest $\alpha$ we can obtain the largest $(w)^TSw$
$\Longrightarrow w^1=$ the eiganvector correspond to the largest eiganvalue
We want to find $w^2=arg\max\limits_{\|w\|=1\\ w^1\cdot w=0}(w)^TCov(X)w$
$\therefore$ we can create a function $g_2(w)=(w)^TSw-\alpha\Big((w)^Tw-1\Big)-\beta\Big((w)^Tw^1\Big)$
$\Longrightarrow\nabla g_2(w)\overset{let}{=}0$
$\Longrightarrow Sw-\alpha w-\beta w^1=0$
$\Longrightarrow (w^1)^TSw-\alpha (w^1)^Tw-\beta (w^1)^Tw^1=0$
$\therefore\beta=0$
$\Longrightarrow Sw=\alpha w$
$\Longrightarrow w^2=$ the eiganvector correspond to the largest eiganvalue and $w^2\neq w^1$
$\therefore w^2=$ the eiganvector correspond to the 2-nd largest eiganvalue
Similarly,we can proof that $w^i$ is the eigenvector of $S=Cov(X)$ correspond to the i-th largest eiganvalue

(2)
$Cov(Z)=\displaystyle{\frac{1}{N}}\sum\limits_{z}(z-\bar{z})(z-\bar{z})^T=WSW^T$ ( where $S=Cov(X)$ )
$=WS[w^1\cdots w^K]=W[Sw^1\cdots Sw^K]=W[\lambda_1w^1\cdots \lambda_Kw^K]$
$[\lambda_1Ww^1\cdots \lambda_KWw^K]=[\lambda_1e_1\cdots \lambda_Ke^K]=$Diagonal Matrix