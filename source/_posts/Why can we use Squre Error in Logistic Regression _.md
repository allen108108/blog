---
title: Why can we use Squre Error in Logistic Regression ?
date: 2019-10-05 01:38:52
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning
image: https://i.imgur.com/llhP9yg.png
mathjax: true
---



由林軒田機器學習第十講 Logistic Regression 的基礎配合李宏毅的 Machine Learning Lecture 5 來思考 ，為何我們推薦使用 Cross-Entropy 作為error measure 而不是沿用 Linear Regression 的 Square error ?

<!-- more -->

![](https://i.imgur.com/UauyPgR.png)

假定我們現在使用的是 Square Error : $E_{in}(h)=\frac{1}{N}\sum\limits_{n=0}^{N}(\theta(\mathbb{W}^T\mathbb{X})-y_n)^2$

當我們要進行 Gradient Descent ，要計算 Gradient 而對各變數進行偏微分時會發現一個狀況 : 

$y_n=1\Longrightarrow\begin{cases} 
\theta(\mathbb{W}^T\mathbb{X})=1\ (close\ to\ target)  & \frac{\partial E_{in}}{\partial w_i}=0 \\
\theta(\mathbb{W}^T\mathbb{X})=0\ (far\ from\ target) & \frac{\partial E_{in}}{\partial w_i}=0
\end{cases}$

同理，當 $y_n=0$ 時也會有一樣的情況。

我們可以將整個 Loss 視覺化出來如下圖 : 

![](https://i.imgur.com/llhP9yg.png)

當距離目標值很遠的部分，使用 Squre error 可能會因為整個 Loss 變化量太小，斜率太小而導致 Gradient Descent 跑不出來，但反觀 Cross-Entropy 便可完美的將每一次迭代的步伐依照隨機選取的點的坡度來進行適當的調整，這也是為什麼在 Logistic Regression 中大多會建議使用 Cross-Entropy 的原因了。