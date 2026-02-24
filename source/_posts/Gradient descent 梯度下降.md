---
title: Gradient descent 梯度下降
date: 2019-10-22 16:06:26
categories:
- 深度學習 Deep Learning
image: https://i.imgur.com/i0i3oUA.png
mathjax: true
---


我一直想要針對 Machine Learning / Deep Learning 的基礎優化方式來寫一篇比較詳盡又不會太困難的文章，一方面想介紹這些很「優美」的方法給初學者或是有點興趣的人知道，另一方面也是想重新審視一下自己對於這些基礎算法的理解到底可以到哪裡。

其實也不是第一次寫這些文章，就梯度下降這一篇也是幾個月前就寫好的，但經過這幾個月來論文、課程的淬鍊之後，對於梯度下降也有一些新的認識跟想法，不如重新來寫一下到底什麼是梯度下降法 ( Gradient Descent ) ? 對於梯度下降法我們又該知道些什麼呢 ? 

<!-- more -->

一切都要從斜率說起。
 
## 斜率

### 直線斜率
首先，我們先來定義一下什麼是斜率。

在數學上，一條直線的斜率可以經由線上兩點來決定 : 
假設有兩點 $(x_1,y_1)$ 以及 $(x_2,y_2)$ 在直線 $L$ 上，那麼斜率的定義為


$$
Slope=\displaystyle{\frac{\Delta y}{\Delta x}=\frac{y_1-y_2}{x_1-x_2}}
$$



![](https://i.imgur.com/YuKOsXB.jpg =400x)



直觀上來看斜率，可以代表直線的陡峭程度以及傾斜方向。也就是說，給我們一條直線 $L$ 的斜率，原則上我們大概就可以在腦中形塑出這條直線的模樣 ( 或許可能會有一些平移上的差距，但原則上大概就是長那個樣子 )

這是我們國高中、大學基本上對於斜率的了解及認識。
那我們是否可以換一個角度來看「斜率」? 

那如果我們站在這條直線上想要往上爬，應該要往哪個方向 ? 斜率能不能告訴我們這件事情 ?

答案是 : 可以。

從定義上來看，斜率的正負號，決定了這條直線該往哪裡傾斜。但其實也間接地告訴我們，想要往上爬應該往哪個方向走。

![](https://i.imgur.com/2gfrexA.jpg)


簡單來說，若斜率為正，表示只要你在這條直線上，往 $x$ 軸的正向走，便會越走越高，反之，若斜率為負，那麼在直線上往 $x$ 軸的負向走，便會越走越高。

### 切線斜率

清楚了直線定義後，對於任一個曲線 (曲面)上任一點，我們如果能夠找到這一點在某方向上的切線斜率，那我們不也可以知道在這一點上的傾斜程度及方向嗎 ? 那不也就可以知道如果我們要往上爬，在這一個方向上，我們該往哪裡走嗎 ?



![](https://i.imgur.com/gqHKVoB.jpg =400x)



## 導數

現在的問題是，如果我們知道了一條曲線方程式，我們該怎麼找到某一點的切線斜率 ?

這時候微積分中導數的概念便可輕鬆解決這樣的問題 


<img width=500 src="https://i.imgur.com/jxToNR7.jpg" >
(紫色線為通過該點的割線，求切線斜率的概念便是將割線斜率中的 $\Delta x$ 逼近於 $0$)


假設 $f(x)$ 為平面上一曲線方程式，$f'(a)=\lim_{x\to\ a}\frac{f(x)-f(a)}{x-a}=f(x)$ 在 a點的切線斜率 $=f(x)$在a點的陡峭程度

(我們可以將導數泛化出一個導函數 $f'(x)=\lim_{h\to 0}\frac{f(x+h)-f(x)}{h}$ ，只要帶入我們想要的點，就可以求出該點的切線斜率)

承上所述，導數的正負號也表現出整個圖形上升的方向。


## 偏導數

如果我們遇到的是「高維度」的「曲面」，我們又該怎麼描述在這個曲面上某一點的陡峭程度 ? ( 想像一下，在一個高高低低的山丘上，你可能站在不同的點，都能感受到不同的傾斜程度 )

把上述導函數的概念稍微推廣至高維度，則偏導函數便是可求出各點在各方向上的切線斜率 (偏導數) 的工具 : 

假設 $f(x_1,x_2,..,x_n)$ 為 $\mathbb{R}^n$ 空間中一曲面方程式，
我們便可以求出 在$x_i$ 方向上的偏導函數 $\frac{\partial f}{\partial x_i}=f(x_1,x_2,..,x_n)$  中某一點 $x$ 在 $x_i$ 方向的切線斜率 $=f(x_1,x_2,..,x_n)$ 中某一點 $x$ 在 $x_i$ 方向的傾斜程度 
而偏微分的正負號當然也可以表現出 $f(x_1,x_2,..,x_n)$ 中某一點 $x$ 在各軸上面往上升的方向。

## 梯度

如果我們把上述每一個方向的偏導函數收集起來，便是我們的梯度 : 

$\nabla f(x_1,x_2,..,x_n) =(\frac{\partial f}{\partial x_1},\frac{\partial f}{\partial x_2},..,\frac{\partial f}{\partial x_n})$

梯度，便是一個可以表現此高為曲面在某一點上，對於每一個方向的描述，我們不再需要個別去看此點在每一個軸上的傾斜表現，梯度包含了所有的資訊在其中。

那麼，若我們想要找到 $f(x_1,x_2,..,x_n)$ 的極值，最簡單的方式便是令 $\nabla f(x_1,x_2,..,x_n) =0$ 
( $Fermat's\ Extreme\ Value\ Theorem$ )



## 梯度在深度學習上的應用

今天如果我們的  $f(x_1,x_2,..,x_n)$ 是一個 Loss Function，計算著真實值與預測值上的差異，那麼梯度便可以用來找出滿足使Loss Function出現最小值的點。 ( 我們在預測上面當然是希望錯誤越少越好! )

$\nabla f(x_1,x_2,..,x_n) =0$ ?

最簡單的方式在深度學習的使用上面卻是困難重重，原因很簡單，根據設定的神經元個數、層數、連結方式...等，往往整個 Loss function 會是一個上千或上萬個變數的函數，再者，根據 Loss function 的型態，梯度的型態通常到最後都是非線性，這種種因素都會導致及困難的計算。


## 梯度下降法 Gradient Descent

梯度下降法的概念便是根據隨機選擇的初始點，不斷調整步伐以及方向，試圖找出最優解。

如果我們可以初始一個在 Loss function 上隨機的點，我們該如何迭代找出新的點不斷的往 Loss function 上的最小值移動呢 ?

梯度下降法給了我們一個方式來不斷地找出新的點來往最小值處移動 

$$
w^{t+1}=w^{t}+\eta\cdot\big(-\nabla L(w^t)\big)=w^t-\eta\cdot\nabla L(w^t)
$$

這個式子其實很直觀，我們可以這樣解釋 : 新的點 ($w^{t+1}$)，是舊的點 ($w^t$)，根據梯度正負號的相反方向 ($-\nabla L(w^t)$) 以及步伐大小 ($\eta$) 所找出來的。

之所以要取 $-\nabla L(w^t)$，是因為原本梯度代表的是向上的方向，但我們這邊要找的是 Loss 的最小值，所以必然要跟梯度不同號。

### 梯度下降法的注意事項 

**1. 梯度下降法的使用條件**

根據上面的介紹，梯度下降法在使用前必須要確定 Loss function 本身是否可微分 (differentiable)，或者至少局部可微。

倘若真的 Loss function 是一個不可微函數，那麼就必須思考如何轉換或是利用凸優化 (convex optimization) 的方式來處理。

**2. 根據不同的隨機選擇起始點，Gradient Descent過程會非常不一樣**

![](https://i.imgur.com/Fj03Kt3.jpg)


**3. 每一次步調的大小也會影響到是否能找到最優解**

若 $\eta$ 太小，迭代次數會很多且容易落在相對極小值 ( 而非絕對最小值 )
若 $\eta$ 太大，迭代結果震盪大，不夠穩定
因此最好的方式是 $\eta$ 隨著每一次迭代做調整

![](https://i.imgur.com/E1AvmX2.gif)

**4. 每一次的梯度計算都必須要將所有資料納入計算**




## 隨機梯度下降法 Stochastic Gradient Deacent

梯度下降法在實質的應用上，必須耗費非常大的計算成本，因為每一次的更新，都必須要計算一次梯度，而每一次計算梯度都必須將所有資料點通通放進來才能計算出梯度，這樣的操作實在不太符合效益。

$$
L(w^t)=\frac{1}{N}\sum\limits_{n=1}^{N}err(y^n,\hat{y}^n)=\frac{1}{N}\sum\limits_{n=1}^{N}err\big(y(x^n,w^t),\hat{y}^n\big)\\
\nabla L(w^t)=\nabla \Big(\frac{1}{N}\sum\limits_{n=1}^{N}err\big(y(x^n,w^t),\hat{y}^n\big)\Big)
$$

也因此在現在的實際應用上，多以隨機梯度下降法 ( Stochastic Gradient Deacent, SGD ) 作為優化算法來進行權重的更新。

有別於梯度下降法必須將所有資料點通通進行運算產生梯度，SGD 的概念是僅利用單一筆資料 ( 或是一小批次的資料 ) 計算梯度即可，只要我們隨機抽取出夠多次的資料點，那麼它們最終的期望值會近似於梯度下降法的結果。

$$
\nabla L(w^t)=\underset{random\ n}{\mathbb{E}}\Big(\nabla \big( err\big(y(x^n,w^t),\hat{y}^n\big)\big)\Big)\approx \nabla \Big(\frac{1}{N}\sum\limits_{n=1}^{N}err\big(y(x^n,w^t),\hat{y}^n\big)\Big)
$$


使用 SGD 取代 GD 的確會讓整個計算成本大幅下降，但由於隨機性的關係，SGD 也會相對不穩定。

![](https://i.imgur.com/i0i3oUA.png)
(圖片來源 : [Optimization Methods](https://datascience-enthusiast.com/DL/Optimization_methods.html))

## 參考資料

1. [王富祥, 游雪玲 (2017)。《七把刀弄懂微積分》(第三版)。考用出版社。](https://www.books.com.tw/products/0010772464)
2. 3Blue1Brown - [Gradient descent, how neural networks learn | Deep learning, chapter 2](https://youtu.be/IHZwWFHWa-w)
3. Hsuan-Tien Lin - [Logistic Regression :: Gradient of Logistic Regression Error @ Machine Learning Foundations (機器學習基石)](https://youtu.be/IZttt_v5tSw)
