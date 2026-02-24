---
title: "Generative Adversarial Network (5) --- General Framework" 
date: 2019-11-11 01:09:00
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning and having it Deep and Structured
image: https://i.imgur.com/RrdITkw.png
mathjax: true
---

從上一個部分的推導，我們可以知道如果我們調整 $V(G,D)$，最後的結果可能就會是另外一種計算 Divergence 的方式，而不會是 JS-Divergence。那我們能不能找出一個 divergence 通式，直接藉由調整通式來改變我們想要的 Divergence ?

<!-- more -->

Sebastian Nowozin 等人在 2016年發表的論文 "*$f$-GAN: Training Generative Neural Samplers using Variational Divergence Minimization*" 便是將這樣的概念應用在 GAN 上面。

$f$-divergence
---

$f$-divergence 我們可以視為是這些 divergence 的通式

$$
D_f(P\|Q)=\int\limits_{x}Q(x)\cdot f(\frac{P(x)}{Q(x)})dx\\
\text{where }f\text{ is a convex function and }f(1)=0 
$$

我們常見的 divergence，例如 : $KL$、$Inverse\ KL$、$JD$、...，我們都能找到一個 $f$ 來對應


![](https://i.imgur.com/QzRAaQw.png)

因為 $f$ 的限制是一個凸函數，所以 $\mathbb{E}(f(x))\geq f(\mathbb{E}(x))$ 且 $f(1)=0$

$$
D_f(P\|Q)=\int\limits_{x}Q(x)\cdot f(\frac{P(x)}{Q(x)})dx\\
\geq f\Big(\int\limits_{x}Q(x)\cdot\frac{P(x)}{Q(x)}\Big)dx=f(1)=0
$$

這兩個條件之所以重要是因為可以確保 $f$-divergence 恆正，且也可以導出當 $P=Q$ 時，$f$-divergence 為 0

$$
P(x)=Q(x)\Longrightarrow D_f(P\|Q)=\int\limits_{x}Q(x)\cdot f(\frac{Q(x)}{Q(x)})dx=0
$$

Convex Conjugate  ( Fenchel Conjugate )
---

在最優化問題上，共軛 (conjugate) 的概念十分重要，原空間難以處理的問題可以轉換到對偶空間上處理，在機器學習中 SVM 便是一個利用對偶概念來建立的模型。

在數學上，共軛函數 (Conjugate Function) 的定義如下[^註1]我覺得可 : 

$$
for\ a\ funciton\  f:X\rightarrow\mathbb{R}\cup\{-\infty,+\infty\}\\
its\ conjugate\ function\ f^*:X^*\rightarrow\mathbb{R}\cup\{-\infty,+\infty\}\\
via.\ f^*(x^*)=\sup\limits_{x\in X}(\langle\ x^*,x\rangle-f(x)) 
$$

共軛函數的幾個重要性質就是 : 
* **「所有的凸函數，都必然會存在一個共軛函數」**
* $(f^*)^*=f$

從這兩個性質我們可以將 $f$-divergence 與 conjugate function 做結合

$$
\because (f^*)^*=f\\
\therefore D_f(P\|Q)=\int\limits_{x}Q(x)\cdot f(\frac{P(x)}{Q(x)})dx\\
=\int\limits_{x}Q(x)\cdot\max\limits_{x^*\in dom(f^*)}\Big(\frac{P(x)}{Q(x)}\cdot x^*-f^*(x^*)\Big)dx
$$

現在我們希望學習一個函數 $D:x\rightarrow x^*$ 可以去逼近 $x^*$，進而可以取代 $x^*$

$$
\because D_f(P\|Q)\geq\int\limits_{x}Q(x)\cdot\Big(\frac{P(x)}{Q(x)}\cdot D(x)-f^*(D(x))\Big)dx\\
=\int\limits_{x}Q(x)\frac{P(x)}{Q(x)}\cdot D(x)dx-\int\limits_{x}Q(x)f^*(D(x)dx\\
=\int\limits_{x}P(x)\cdot D(x)dx-\int\limits_{x}Q(x)f^*(D(x)dx\\
\therefore D(P\|Q)\approx\max\limits_{x\in dom(f)}\Big(\int\limits_{x}P(x)\cdot D(x)dx-\int\limits_{x}Q(x)f^*(D(x)dx\Big)
$$

所以 $f$-GAN 在做的其實就是這件事情

$$
D_f(P_{data}\|P_G)=\max\limits_{D}\Big(\mathbb{E}_{x\sim P_{data}}D(x)-\mathbb{\mathbb{E}_{x\sim P_G}}(f^*(D(x)))\Big)\overset{let}{=}\max\limits_{D} V(G,D)\\
G^*=\arg\min\limits_{G} D_f(P_{data}\|P_G)\\
=\arg\min\limits_{G}\max\limits_{D} V(G,D)\\
$$

將 $V(G,D)$ 表達式一般化，可以在各種不同 divergence 之間做變換。

註釋
---
[^註1]: 
凸共軛的圖形我覺得可以參考下列網址
https://www.researchgate.net/figure/Construction-of-the-Legendre-Fenchel-conjugate-of-a-convex-function-The-left-plot_fig6_279825155