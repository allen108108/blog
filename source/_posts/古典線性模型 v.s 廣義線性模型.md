---
title: 古典線性模型 v.s 廣義線性模型
date: 2020-03-25 00:05:15
categories:
- 深度學習 Deep Learning
image: https://i.imgur.com/pu8KKwa.png
mathjax: true
---

前言
---

撰寫此文主要受 "[剖析深度學習 (4)：Sigmoid, Softmax怎麼來？為什麼要用MSE和Cross Entropy？談廣義線性模型](https://www.ycc.idv.tw/deep-dl_4.html)" 一文所激勵。

<!-- more -->

上述文章算是簡繁體中文技術部落格中高度掌握脈絡的優質好文，但筆者想從不同的脈絡進展下來理解古典線性模型與廣義線性模型，因此決定進行此文的編寫。


古典線性模型 CLRM, Classic Linear Regression Model
---

高中的時候，我們都學過如何以最小平方法從資料找出一個線性回歸模型，然而，在現實狀況中，我們無法得知整個母體的所有資料，只能利用母體隨機抽樣出來的資料找出樣本迴歸函數 (SRF, Sample Regression Function)，並且期望找出一個「最佳」的 SRF 來擬合母體回歸函數 (PRF, Population Regression Function)。

為了找出「最佳」 SRF，我們給了一些「假設」，從符合這些假設的可能性中找出最佳的結果，符合這些假設的模型，我們便稱之為 「古典線性模型」(CLRM, Classic Linear Regression Model)。

在古典線性模型中，我們給了以下的假設 : 
1. 模型參數為線性，這並不表示變量必須呈線性。
$$
y_i=w_0+w_1x_1+w_2x_2+\cdots+w_nx_n+\epsilon_n=\boldsymbol{wx^T}+\epsilon_n\\
=\text{The linear combination of }X
$$
2. 變量 $\boldsymbol{x}$ 是固定、非隨機的，意指古典線性模型中是在給定的變量條件下進行。
3. 在給定變量 $\boldsymbol{x}$ 的條件下，誤差項 $\epsilon$ 的期望值為 $0$。
$$
E(\epsilon\mid \boldsymbol{x})=0
$$
4. 在給定變量 $\boldsymbol{x}$ 的條件下，誤差項 $\epsilon$ 的變異數相同為 $\sigma^2$。
$$
Var(\epsilon\mid \boldsymbol{x})=\sigma^2
$$
5. 誤差項分布是均值為 $0$，變異數為 $\sigma^2$ 的常態分佈。
$$
\epsilon\sim N(0,\sigma^2)
$$
6. 不同變量 $\boldsymbol{x}_i,\boldsymbol{x}_j$ 誤差項 $\epsilon_i,\epsilon_j$ 之共變異數為$0$。
$$
Cov(\epsilon_i,\epsilon_j\mid \boldsymbol{x})=0
$$
7. 變量 X 不具共線性。
8. 觀察變量之數量必然不小於於參數數量。
$$
N\geq Rank(\boldsymbol{x})\\
\text{where }N\text{ is number of observations }
$$
9. 分析的回歸模型均符合正確指定 (correctly specified) 假設。也就是說我們均假設模型就是 $y_i=w_0+\Sigma_{j=1}^{n} w_jx_j+\epsilon_i$ 這種型態，不正確的模型指定就會產生參數估計的偏差。


從上述的假設中，我們可以推得 $y_i\sim N(w_0+\Sigma_{j=1}^{n} w_jx_j,\sigma^2)$ 這個結論也就是古典線性模型與廣義線性模型的最大差異。

![](https://i.imgur.com/pu8KKwa.png)


統計量與充分統計量 Statistic & Sufficient statistic
---

在延伸至廣義線性模型之前，我們先來了解一些基本概念。

>**統計量的定義是，假設一組隨機樣本 $\boldsymbol{x}=(x_1, x_2,\cdots,x_n)$，若我們依此設計出一個不依賴其他未知參數的函數 $t:\boldsymbol{x}\rightarrow \mathbb{R}^k$， 則稱此函數為隨機樣本 $x_1, x_2,\cdots,x_n$的一個統計量**。

舉個例子來說，隨機樣本 $x_1, x_2,\cdots,x_n$ 的統計量可以是 : 

* $t(x_1,x_2,\cdots,x_n)=\dfrac{1}{n}\sum_{i=1}^{n}x_i=\bar{\boldsymbol{x}}$
* $t(x_1,x_2,\cdots,x_n)=\dfrac{1}{n}\sum_{i=1}^{n}(x_i-\bar{\boldsymbol{x}})^2=\sigma^2$

但要注意的是，像 $\dfrac{1}{n}\sum_{i=1}^{n}(x_i-E(\boldsymbol{x}))^2$ 這類型的函數並不能稱為統計量，從定義來看，它用到了未知的參數 $E(\boldsymbol{x})$ ，這是關於未知母體的參數。

>**充分統計量的定義則是，假設一組隨機變量 $\boldsymbol{x}=(x_1, x_2,\cdots,x_n)$ 其分布 $P_{\theta}(\boldsymbol{x})$ 依賴給定參數 $\theta$ ， $\boldsymbol{T}=\{t_1,t_2,\cdots,t_m\}$ 為一統計量集，若條件機率 $P(\boldsymbol{x}\mid \boldsymbol{T})$ 與 $\theta$ 無關，則稱 $T$ 為 $\boldsymbol{x}$ 的充分統計量。**

我們以「做實驗」來解釋上面的定義，我們手中如果有 $n$ 筆觀測數據，那麼我們可以確定的是基於某些參數條件 $\theta$ 下的分布 $P_{\theta}$ 資訊勢必都在這些數據資料當中，但光僅有這些資料是無法完整描述整個分布。

若我們可以取得一組充分統計量 $\boldsymbol{T}$，「$P(\boldsymbol{x}\mid \boldsymbol{T})$ 與 $\theta$ 無關」這個條件其實就暗示著我們可以從這個條件分布中抽樣出許多的新資料，其分布與 $P_{\theta}(\boldsymbol{x})$ 相同。換句話說，我們可以得到更多的資料來去描述 $P_{\theta}(\boldsymbol{x})$ 這個分布。

理論上來說是如此，但在現實情況來說，「$P(\boldsymbol{x}\mid \boldsymbol{T})$ 與 $\theta$ 無關」這件事情難以被驗證，因此衍生出一個一個非常重要的定理 (證明詳見 Appendix)

$\text{Fisher–Neyman factorization theorem}$

$$
T\text{ is sufficient statistic}\Longleftrightarrow P_{\theta}(\boldsymbol{x})=h(\boldsymbol{x})g_{\theta}(\boldsymbol{T})\\
$$

這個定理確定了基於 $\theta$ 的分布 $P_{\theta}(\boldsymbol{x})$ 與充分統計量的公式化關係，使得尋找充分統計量這件事情變得容易許多。

關於充分統計量，最明顯的例子便是常態分佈的充分統計量為 $\mu,\sigma^2$。



指數族分布 Exponential Family Distribution
---

我們在古典線性模型當中給了非常多的強假設，最重要的是

$$
\epsilon\sim N(0,\sigma^2)
$$

但在現實環境中，這個 Noise 項並不會這麼完美，它可能並**不服從常態分布或甚至隨著 $X$ 的變化而有所改變**，這樣的前提之下，我們必須有一個更通用的線性模型來描述各種可能的狀況。

那我們能不能以一個統一的形式化來描述這些**非局限於常態分布**的分布 ?

指數族分布便解決了這樣的一個問題，利用指數族分布可以描述除了常態分佈以外的多種分布型態。指數族分布的機率密度函數遵循下列的形式 : 

$$
P(\boldsymbol{x}\mid \boldsymbol{\eta})=h(\boldsymbol{x})\exp\big(\boldsymbol{\eta}^T\boldsymbol{t}(\boldsymbol{x})-a(\boldsymbol{\eta})\big)\\
\text{where }a(\boldsymbol{\eta})=\log\int h(\boldsymbol{x})\exp\big(\boldsymbol{\eta}^Tt(\boldsymbol{x})\big)d\boldsymbol{x}
$$

* $\boldsymbol{\eta}:$ 自然參數 (Natural Parameters)
* $\boldsymbol{t}(\boldsymbol{x}):$ 充分統計量 (Sufficient statistic)
* $h(\boldsymbol{x}):$ 依賴測度 (Underlying mesurement)
* $a(\boldsymbol{\eta}):$ 對數標準化 (Log normalize) 以確保此分布積分值為 $1$

我們常見的分布型態均屬於指數分布族的，意即，均能以上述形式來表述

* **常態分布 Normal Distribution**

$$
P(x\mid \mu,\sigma^2 )=\dfrac{1}{\sqrt{2\pi\sigma^2}}\exp\Big\{\dfrac{-(x-\mu)^2}{2\sigma^2}\Big\}=P(x\mid\eta)=\dfrac{1}{\sqrt{2\pi}}\exp\Big\{\dfrac{\mu}{\sigma^2}x-\dfrac{1}{2\sigma^2}x^2-\dfrac{1}{2\sigma^2}\mu^2-\log(\sigma)\Big\}\\
\Longrightarrow P(x\mid\boldsymbol{\eta})=\dfrac{1}{\sqrt{2\pi}}\exp\Big\{\boldsymbol{\eta}^T\boldsymbol{t}(x)-\dfrac{1}{2\sigma^2}\mu^2-\log(\sigma)\Big\}\\
\text{where } \boldsymbol{\eta}=(\dfrac{\mu}{\sigma^2},\dfrac{1}{2\sigma^2})\text{ and } \boldsymbol{t}(x)=(x,x^2)
$$ 


* **二項式分布 Bernoulli Distribution**

$$
P(x\mid\pi)=\pi^x(1-\pi)^{1-x}=\exp\Big\{x\log\dfrac{\pi}{1-\pi}+\log(1-\pi)\Big\}\\
\Longrightarrow P(x\mid\eta)=\exp\Big\{\eta x-\log(1+e^{\eta})\Big\}
$$


* **多項式分布  Multinomia Distribution**

$$
P(x\mid\boldsymbol{\pi})=\prod_{k=1}^m\pi_k^{\delta(x=k)}\\
\text{where }\sum_{k=1}^m\pi_k=1\\
\Longrightarrow P(x\mid\boldsymbol{\eta})=\exp\Big\{\boldsymbol{\eta^T}\boldsymbol{\delta}+\log(1-\sum_{k=1}^me^{\eta_k}) \Big\}\\
\text{where }\eta_k=\log\Big(\dfrac{\pi_k}{1-\sum_{j=1}^{m-1}\pi_j}\Big)\text{ and }\delta_k=\delta(x=k)
$$


* **卜氏分布 Poisson Distribution**

$$
P(x\mid\lambda)=\dfrac{\lambda^xe^{-\lambda}}{x!}=\dfrac{1}{x!}\exp\Big\{x\log\lambda-\lambda\Big\}\\
\Longrightarrow P(x\mid\eta)=\dfrac{\eta^xe^{-\eta}}{x!}=\dfrac{1}{x!}\exp\Big\{x\log\eta-\eta\Big\}
$$



指數族分布的統一表達式可以推導出充分統計量的均值及變異數 (證明詳見 Appendix)，而這個結論，將會是擬合廣義線性模型的重要工具。

$$
E\big[t(x)\big]=\dfrac{\partial a(\eta)}{\partial\eta}=\mu\overset{t(x)=x}{=}E\big[x\big]\\
Var\big[t(x)\big]=\dfrac{\partial^2 a(\eta)}{\partial\eta^2}
$$

( $t(x)$ 並不見得會等於 $x$，但在許多情況下都會成立，例如 Bernoulli Distribution 或是 Multinomia Distribution  )

此結論也暗示由於 $a(\eta)$ 為凸函數，二階導數恆為正，因此確保其一階導函數為一對一函數，且 $\eta$ 與 $\mu$ 存在一對一關係。

廣義線性模型
---

無論是古典線性模型，或是廣義線性模型，其實中心思想都一樣，就是試圖利用隨機樣本 $\boldsymbol{x}=\{x_1,x_2,\cdots,x_n\}$ 的線性組合來預測結果 $y$。

但廣義線性模型 GLM 擴展了古典線性模型的假設 : 

* $y\sim P(y\mid \eta)=h(y)\exp\big(\eta^T\boldsymbol{t}(y)-a(\eta)\big)$ ，這項假設**讓$y$ 的分布不再限於常態分佈**。
* $\eta=\boldsymbol{w}^T\boldsymbol{x}$，這項強假設將**指數族分布納入了線性預測**。
* 引入了 Link Function $g(\mu)=\eta$，**將 $y$ 的分布平均值 $\mu$  ( 或是充分統計量的平均值 ) 與線性預測 $\boldsymbol{w}^T\boldsymbol{x}$ 連結在一起**

特別說一下 Link Function，這個部分在廣義線性模型中是最難讓人理解的一環，引入這樣的函數關係究竟有什麼意義呢 ?

有一個角度或許可以比較清楚 Link Function 的作用。在進行擬合的過程，線性預測 $\boldsymbol{w}^T\boldsymbol{x}$ 的值域應該是 $[-\infty,+\infty]$ ，但我們在不同任務上所得到的 $y$ 值分布可能沒有這麼廣 ( 例如: 二分類任務 $y\in\{0,1\}$ )，這種分布上面的差異會使得擬合出現問題，此時，便需要一個函數將指數族分布的均值 $\mu$ 一對一映射至 $[-\infty,+\infty]$ 。

當然這函數理所當然地便稱為 " Link Function "。由於這是一個一對一函數，其反函數必存在，並稱其為 " Response Function "


由上述的解釋中，或許可以更清楚理解廣義線性模型藉由上述的假設以及 Link Function 的引入，創造了更多的可能性，也使預測模型更加強健。有了廣義線性模型，我們可以藉此來解釋一些機器學習中的問題。

補充 : 深度學習中的 Sigmoid , Softmax 函數
---

在深度學習中，當我們針對特定任務建構網路結構時，最後一層的 activation function 似乎就隨著任務型態而確定，為什麼會這樣子呢 ? 或許我們可以從 GLM 的角度來解釋這一切。

### 二元分類問題 Binary Classification

在指數族分布的段落，我們呈現了 Bernoulli Distribution 的指數族分布型態 

$$
P(y\mid\eta)=\exp\Big\{\eta y-\log(1+e^{\eta})\Big\}
$$

藉此我們可推得

$$
h(y)=1\\
t(y)=y\\
\eta=\log\dfrac{\pi}{1-\pi}\Longrightarrow \pi=\dfrac{e^{\eta}}{1+e^{\eta}}\\
a(\eta)=\log(1+e^{\eta})=-\log(1-\pi)
$$

且

$$
E\big[t(y)\big]=E\big[y\big]=\dfrac{\partial a(\eta)}{\partial\eta}=\pi=\dfrac{e^{\eta}}{1+e^{\eta}}=\dfrac{e^{\boldsymbol{w}^T\boldsymbol{x}}}{1+e^{\boldsymbol{w}^T\boldsymbol{x}}}=Sigmid(\boldsymbol{w}^T\boldsymbol{x})
$$

$E\big[y\big]$ 為我們擬合的預測結果，它剛好是經由隨機變數經過線性組合後 $\boldsymbol{w}^T\boldsymbol{x}$ 經過 sigmoid 函數而得，這也是最後一層選用 Sigmoid activation function 的原因。


### 多元分類問題 Multinomia Distribution

Multinomia Distribution 的指數族分布型態

$$
P(x\mid\boldsymbol{\eta})=\exp\Big\{\boldsymbol{\eta^T}\boldsymbol{\delta}+\log(1-\sum_{k=1}^me^{\eta_k}) \Big\}\\
\text{where }\eta_k=\log\Big(\dfrac{\pi_k}{1-\sum_{j=1}^{m-1}\pi_j}\Big)\text{ and }\delta_k=\delta(x=k)
$$

可以推得

$$
h(y)=1\\
t(y)=\delta(y=k)\\
\eta=\begin{bmatrix}\eta_1\\\eta_2\\\vdots\\\end{bmatrix}\text{ and }\eta_k=\log\Big(\dfrac{\pi_k}{1-\sum_{j=1}^{m-1}\pi_j}\Big)=\log\dfrac{\pi_k}{\pi_m} \\
\Longrightarrow\pi_k=\dfrac{e^{\eta_k}}{1+\sum_{j=1}^{m-1}e^{\eta_k}}\\
a(\eta)=\log(1+\sum_{k=1}^me^{\eta_k})=-\log(1-\sum_{k=1}^{m-1}\pi_k)
$$

從上面的結論我們可以知道

$$
E\big[t(y)\big]=E\big[\delta(y=k)\big]=\pi_k=\pi_k=\dfrac{e^{\eta_k}}{1+\sum_{j=1}^{m-1}e_{\eta_k}}=\pi_k=\dfrac{e^{\boldsymbol{w}^T\boldsymbol{x}}}{1+\sum_{j=1}^{m-1}e^{\boldsymbol{w}^T\boldsymbol{x}}}=Softmax(\boldsymbol{w}^T\boldsymbol{x})
$$

APPENDIX
---

### Fisher–Neyman factorization theorem 

$$
T\text{ is sufficient statistic}\Longleftrightarrow P_{\theta}(\boldsymbol{x})=h(\boldsymbol{x})g_{\theta}(\boldsymbol{T})\\
$$


#### < Proof >

($\Longrightarrow$)
$\because T(x) \text{ is sufficient statistic}\Longrightarrow P(\boldsymbol{x}\mid\theta,\boldsymbol{T})=P(\boldsymbol{x}\mid\boldsymbol{T})$
$\therefore P_{\theta}(\boldsymbol{x})=P(\boldsymbol{x}\mid\theta)=P(\boldsymbol{x,T}\mid\theta)=P(\boldsymbol{x}\mid\theta,\boldsymbol{T})P(\boldsymbol{T}\mid\theta)=P(\boldsymbol{x}\mid \boldsymbol{T})P(\boldsymbol{T}\mid\theta)\overset{let}{=}h(\boldsymbol{x})g_{\theta}(\boldsymbol{T})$

($\Longleftarrow$)
$\because P(\boldsymbol{x}\mid\theta)=h(\boldsymbol{x})g_{\theta}(\boldsymbol{T})$

$\Longrightarrow\displaystyle{\int_{\boldsymbol{T}} P(\boldsymbol{x}\mid\theta)d\boldsymbol{x}=\int_{\boldsymbol{T}} h(\boldsymbol{x})g_{\theta}(\boldsymbol{T})d\boldsymbol{x}}=g_{\theta}(\boldsymbol{T})\int_{\boldsymbol{T}} h(\boldsymbol{x})d\boldsymbol{x}=g_{\theta}(\boldsymbol{T})H(\boldsymbol{T})$

$\Longrightarrow P(\boldsymbol{T}\mid\theta)=g_{\theta}(\boldsymbol{T})H(\boldsymbol{T})$

$\Longrightarrow g_{\theta}(\boldsymbol{T})=\dfrac{P(\boldsymbol{T}\mid\theta)}{H(\boldsymbol{T})}$

$\Longrightarrow P_{\theta}(\boldsymbol{x})=h(\boldsymbol{x})g_{\theta}(\boldsymbol{T})=h(\boldsymbol{x})\dfrac{P(\boldsymbol{T}\mid\theta)}{H(\boldsymbol{T})}$

$\because P(\boldsymbol{x}\mid\theta)=P(\boldsymbol{x,T}\mid\theta)=P(\boldsymbol{x}\mid\theta,\boldsymbol{T})P(\boldsymbol{T}\mid\theta)$

$\therefore P(\boldsymbol{x}\mid\theta,\boldsymbol{T})=\dfrac{P(\boldsymbol{x}\mid\theta)}{P(\boldsymbol{T}\mid\theta)}=\dfrac{h(\boldsymbol{x})}{H(\boldsymbol{T})}\Longrightarrow P(\boldsymbol{x}\mid\theta,\boldsymbol{T}) \text{ is independent of } \theta$



### Properties of Exponential Family Distribution

$$
E\big[t(\boldsymbol{x})\big]=\dfrac{\partial a(\boldsymbol{\eta})}{\partial\boldsymbol{\eta}}\text{ and   } Var\big[t(\boldsymbol{x})\big]=\dfrac{\partial^2 a(\boldsymbol{\eta})}{\partial\boldsymbol{\eta}^2}
$$

#### < Proof >

(1)

$$
\dfrac{\partial a(\boldsymbol{\eta})}{\partial\boldsymbol{\eta}}=\dfrac{\partial}{\partial\boldsymbol{\eta}}\Big(\log\int h(\boldsymbol{x})\exp\big(\boldsymbol{\eta}^Tt(\boldsymbol{x})\big)d\boldsymbol{x}\Big)\\
=\dfrac{1}{\int h(\boldsymbol{x})\exp\big(\boldsymbol{\eta}^Tt(\boldsymbol{x})\big)d\boldsymbol{x}}\cdot\frac{\partial}{\partial\boldsymbol{\eta}}\Big(\int h(\boldsymbol{x})\exp\big(\boldsymbol{\eta}^Tt(\boldsymbol{x})\big)d\boldsymbol{x}\Big)\\
=\dfrac{1}{\int h(\boldsymbol{x})\exp\big(\boldsymbol{\eta}^Tt(\boldsymbol{x})\big)d\boldsymbol{x}}\cdot\int h(\boldsymbol{x})\exp\big(\boldsymbol{\eta}^Tt(\boldsymbol{x})\big)t(\boldsymbol{x})d\boldsymbol{x}\\
=\dfrac{1}{\exp(-a(\boldsymbol{\eta}))}\cdot\int h(\boldsymbol{x})\exp\big(\boldsymbol{\eta}^Tt(\boldsymbol{x})\big)t(\boldsymbol{x})d\boldsymbol{x}\\
=\int h(\boldsymbol{x})\exp\big(\boldsymbol{\eta}^Tt(\boldsymbol{x})-a(\boldsymbol{\eta})\big)t(\boldsymbol{x})d\boldsymbol{x}\\
=\int P(\boldsymbol{x}\mid\boldsymbol{\eta})t(\boldsymbol{x})^2d\boldsymbol{x}\\
=E\big[t(\boldsymbol{x})\big]
$$

(2)

$$
\dfrac{\partial^2 a(\boldsymbol{\eta})}{\partial\boldsymbol{\eta}^2}=\dfrac{\partial E\big[t(\boldsymbol{x})\big]}{\partial\boldsymbol{\eta}}=\int h(\boldsymbol{x})\exp\big(\boldsymbol{\eta}^Tt(\boldsymbol{x})-a(\boldsymbol{\eta})\big)t(\boldsymbol{x})\cdot\Big(t(\boldsymbol{x})-\dfrac{\partial a(\boldsymbol{\eta})}{\partial\boldsymbol{\eta}}\Big)d\boldsymbol{x}\\
=\int h(\boldsymbol{x})\exp\big(\boldsymbol{\eta}^Tt(\boldsymbol{x})-a(\boldsymbol{\eta})\big)t(\boldsymbol{x})^2d\boldsymbol{x}-\dfrac{\partial a(\boldsymbol{\eta})}{\partial\boldsymbol{\eta}}\cdot \int h(\boldsymbol{x})\exp\big(\boldsymbol{\eta}^Tt(\boldsymbol{x})-a(\boldsymbol{\eta})\big)t(\boldsymbol{x})d\boldsymbol{x}\\
=\int P(\boldsymbol{x}\mid\boldsymbol{\eta})t(\boldsymbol{x})d\boldsymbol{x}-\Big(\dfrac{\partial a(\boldsymbol{\eta})}{\partial\boldsymbol{\eta}}\Big)^2\\
=E\big[t(\boldsymbol{x})^2\big]-E\big[t(\boldsymbol{x})\big]^2\\
=Var\big[t(\boldsymbol{x})\big]
$$




參考資料
---

1. [剖析深度學習 (4)：Sigmoid, Softmax怎麼來？為什麼要用MSE和Cross Entropy？談廣義線性模型](https://www.ycc.idv.tw/deep-dl_4.html)
2. [Sufficient statistics - Arizona Math](https://www.math.arizona.edu/~tgk/466/sufficient.pdf)
3. [知乎 - 数理统计|笔记整理（3）——充分统计量](https://zhuanlan.zhihu.com/p/87520809)
4. [知乎 - 求大神给解释解释充分统计量啊？](https://www.zhihu.com/question/41367707)
5. [百度百科 - 指数族分布](https://baike.baidu.com/item/%E6%8C%87%E6%95%B0%E6%97%8F%E5%88%86%E5%B8%83/19098398)
6. [GLM(广义线性模型) 与 LR(逻辑回归) 详解](https://blog.csdn.net/Cdd2xd/article/details/75635688)
7. [EXPONENTIAL FAMILY AND GENERALIZED LINEAR MODELS](https://www.cs.princeton.edu/courses/archive/spring09/cos513/scribe/lecture11.pdf)
8. [Introduction: exponential family, conjugacy, and sufficiency](https://www.cs.princeton.edu/~bee/courses/scribe/lec_09_02_2013.pdf)
9. [知乎 - 广义线性模型中, 联系函数(link function) 的作用是不是就是将不是正态分布的Y转换成正态分布？](https://www.zhihu.com/question/28469421/answer/402258374)
