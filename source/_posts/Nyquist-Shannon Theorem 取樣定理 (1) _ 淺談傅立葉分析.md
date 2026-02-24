---
title: "Nyquist-Shannon Theorem 取樣定理 (1) : 淺談傅立葉分析"
date: 2020-11-04 06:23:48
categories:
- 數學筆記 Mathematics Note
image: https://i.imgur.com/hTpy1pK.png
mathjax: true
---


在工程數學中，通常牽扯到兩種變換模式，一個是拉普拉斯變換 ( Laplace Transform )，另一個就是本篇主要的對象 -- 傅立葉變換 ( Fourier Transform )。

我們都知道，在量子領域中，粒子均有波粒二項性，因此，在物理世界中，討論「波」、「波動」這件事情變得十分重要，例如 : 聲波、光波...等。不只是物理，在訊號處理的領域中，訊號也往往以「波」的方式呈現，例如在生物醫學中的訊號 : 腦波、心電圖...等等。

<!-- more -->

這種種的 「波」，如果是以簡單、週期性的方式呈現，要進行分析或許還容易些，但是真實世界中的訊號呈現往往複雜甚至不具週期性，我們要對這樣的訊號進行處理簡直是難上加難 - 至少在傅立葉變換出現以前是如此。

![](https://i.imgur.com/hTpy1pK.png)
( 圖 (一) -- 圖片來源 : [使用MATLAB進行數位訊號處理簡介](https://micro.rohm.com/tw/deviceplus/how-tos/arduino-guide/arduino-dsp-intro-to-digital-signal-processing-using-matlab-part-1/) )

一個化合物，如果我們可以知曉其組成成分，那麼或許可以了解其特性，訊號亦是如此。傅立葉變換提供了一個數學工具，讓我們可以確保所有的「波」均可以分解成 不同頻率的 $\sin$ 及 $\cos$ 函數，進而藉由各個組成週期函數的相位及頻率了解原訊號的特性。



傅立葉級數 Fourier Series
---

如果今天我們原訊號本身為一個週期性函數 $f(t)$，週期為 $2T$，則滿足

$$\begin{align}
f(t)=f(t+2T) \nonumber \\
\end{align}$$

傅立葉認為， 若 $f:\mathbb{R}\rightarrow\mathbb{C}$ 滿足上述條件的 $f(t)$ 且在長度為 $2T$ 的區間可積，均可以利用三角級數來進行表徵[^1]，意即

$$
f(t)=\frac{a_0}{2}+\sum_{n=1}^{\infty}a_n\cos(nt)+b_n\sin(nt)
$$

其中 $a_n$, $b_n$ 我們稱之為傅立葉係數

$$
a_n=\displaystyle{\frac{1}{T}\int_{2T}f(t)\cos (nt)dt},\mbox{   }\forall n\in\mathbb{N}\cap\{0\}
$$
$$
b_n=\displaystyle{\frac{1}{T}\int_{2T}f(t)\sin(nt)dt},\mbox{   }\forall n\in\mathbb{N}\\
$$

這樣的級數我便是我們所稱的傅立葉級數 (Fourier Series)。證明過程需要有高等微積分作為先備知識，如果有興趣的讀者可以參考筆者所附的參考資料 [10][11][12]。

傅立葉級數給了一個強而有力的論證，讓所有的週期函數，均可以用不同頻率的 $\sin$ 以及 $\cos$ 加權疊加而形成。也就是說，我們可以將一個看似複雜的週期性函數進行分解，利用這些已知的週期三角函數來分析原函數的各項性質，這是多麼有趣的現象 !

傅立葉變換 Fourier Transform
---

在介紹傅立葉變換 (Fourier Transform) 以前，有必要先來介紹一下神奇的歐拉公式，這也是傅立葉變換或是前面介紹的傅立葉級數中的一個核心概念。

### 歐拉公式

歐拉公式之所以重要，最主要的貢獻是它將自然對數、三角函數以及複數這三個看似不相關的數學概念利用一個公式串接在一起 : 

$$
e^{i\theta}=\cos\theta+i\sin\theta,\mbox{  }\forall\theta\in\mathbb{R}
$$

歐拉公式的證明只需要有高中數學為背景即可證明，詳細的證明筆者亦不贅述。

### 傅立葉變換

#### 週期為 $2\pi$


前面提到了周期函數均可以利用傅立葉級數來進行分解，但在真實世界中，許多的訊號都是非週期性的，那麼是不是也可以有類似完美的方法對訊號進行分解呢 ?

當然可以 (否則就沒有接下來的文章內容了) ! 有別於傅立葉級數，針對非週期性函數，我們稱其為(連續)傅立葉變換 (Fourier Transform)。

首先我們先考慮函數 $f:\mathbb{R}\rightarrow\mathbb{C}$ 為區間可積之週期函數，且其週期為 $2\pi$，那麼我們可以利用歐拉公式來對傅立葉級數進行調整。從歐拉公式我們可以推得

$$
\cos(nt)=\frac{(e^{int}+e^{-int})}{2}
$$
$$
\sin(nt)=\frac{(e^{int}-e^{-int})}{2i}
$$

將其帶入傅立葉級數後

$$
f(t)=\frac{1}{2\pi}\int_{-\pi}^{\pi}f(t)dt+\sum_{n=1}^{\infty}\big[\cos(nt)\frac{1}{\pi}\int_{-\pi}^{\pi}f(t)\cos(nt)dt+\sin(nt)\frac{1}{\pi}\int_{-\pi}^{\pi}f(t)\sin(nt)dt\big]
$$
$$
=\frac{1}{2\pi}\int_{-\pi}^{\pi}f(t)dt+\sum_{n=1}^{\infty}\big[\frac{(e^{int}+e^{-int})}{2}\frac{1}{\pi}\int_{-\pi}^{\pi}f(t)(\frac{(e^{int}+e^{-int})}{2})dt+\frac{(e^{int}-e^{-int})}{2i}\frac{1}{\pi}\int_{-\pi}^{\pi}f(t)(\frac{(e^{int}-e^{-int})}{2i})dt\big]
$$
$$
=\frac{1}{2\pi}\int_{-\pi}^{\pi}f(t)dt+\frac{1}{4\pi}\sum_{n=1}^{\infty}\big[(e^{int}+e^{-int})\int_{-\pi}^{\pi}f(t)(e^{int}+e^{-int})dt-(e^{int}-e^{-int})\int_{-\pi}^{\pi}f(t)(e^{int}-e^{-int})dt\big]
$$
$$
=\frac{1}{2\pi}\Big[\int_{-\pi}^{\pi}f(t)dt+\frac{1}{2}\sum_{n=1}^{\infty}\big[(e^{int}+e^{-int})\int_{-\pi}^{\pi}f(t)(e^{int}+e^{-int})dt-(e^{int}-e^{-int})\int_{-\pi}^{\pi}f(t)(e^{int}-e^{-int})dt\big]\Big]
$$
$$
=\frac{1}{2\pi}\Big[\int_{-\pi}^{\pi}f(t)dt+\frac{1}{2}\sum_{n=1}^{\infty}\big[e^{-int}\int_{-\pi}^{\pi}f(t)e^{int}dt+e^{int}\int_{-\pi}^{\pi}f(t)(e^{-int})dt\big]\Big]\cdots\cdots(1)
$$

而由於其對稱性，我們可以得知

$$
f(t)=\frac{1}{2\pi}\Big[\int_{-\pi}^{\pi}f(t)dt+\frac{1}{2}\sum_{n=-\infty}^{-1}\big[e^{-int}\int_{\pi}^{\pi}f(t)e^{int}dt+e^{int}\int_{-\pi}^{\pi}f(t)(e^{-int})dt\big]\Big]\cdots\cdots(2)\\
$$

且當 $n=0$ 時，

$$
e^{-int}\int_{\pi}^{\pi}f(t)e^{int}dt+e^{int}\int_{-\pi}^{\pi}f(t)(e^{-int})dt=2\int_{-\pi}^{\pi}f(t)dt\cdots\cdots(3)
$$

將 $(1)(2)(3)$ 式相加後

$$
2f(t)=\frac{1}{2\pi}\sum_{n=-\infty}^{\infty}\big[e^{-int}\int_{\pi}^{\pi}f(t)e^{int}dt+e^{int}\int_{-\pi}^{\pi}f(t)(e^{-int})dt\big]
$$
$$
=\frac{1}{2\pi}\cdot2\cdot\sum_{n=-\infty}^{\infty}\big[e^{int}\int_{-\pi}^{\pi}f(t)(e^{-int})dt\big]
$$


因此可以改寫傅立葉級數為下列型態

$$
f(t)=\frac{1}{2\pi}\sum_{n=-\infty}^{\infty}f_n(t)e^{int}
$$
$$
\mbox{where }f_n(t)=\int_{-\pi}^{\pi}f(t)(e^{-int})dt\cdots\cdots(4)
$$

#### 週期為 $4\pi$

現在我們考慮一個狀況，假設 $f(t)$ 為一個區間可積且週期為 $4\pi$ 的週期函數， 那麼上述的傅立葉級數便無法使用。不過我們可以知道 $f(2t)$ 的週期就是 $2\pi$ ，因此

$$
g(t)\overset{let}{=}f(2t)=\frac{1}{2\pi}\sum_{n=-\infty}^{\infty}e^{int}\int_{-\pi}^{\pi}g(t)e^{-int}dt
$$
$$
\Longrightarrow f(2t)=\frac{1}{2\pi}\sum_{n=-\infty}^{\infty}e^{int}\int_{-\pi}^{\pi}f(2t)e^{-int}dt
$$
$$
\Longrightarrow f(t)=\frac{1}{2\pi}\Big(\frac{1}{2}\sum_{n=-\infty}^{\infty}e^{\frac{int}{2}}\int_{-2\pi}^{2\pi}f(t)e^{\frac{-int}{2}}dt\Big)\\
$$

同理可證，任何一個週期為 $2k\pi$ 的週期函數 $f(x)$ 均可以下列方式表示

$$
f(x)= \frac{1}{2\pi}\Big(\frac{1}{k}\sum_{n=-\infty}^{\infty}e^{\frac{int}{k}}\int_{-k\pi}^{k\pi}f(x)e^{\frac{-int}{k}}dx\Big)\cdots\cdots(5)
$$

在數學中可以證明 : 

$$
\mbox{If } \int_{-\pi}^{\pi}|f(x)|dx \mbox{ exists, then }\exists \hat{f} \mbox{ s.t. } \hat{f}(\frac{n}{k})=\int_{-k\pi}^{k\pi}f(x)e^{\frac{-int}{k}}dx
$$

則 (5) 式可以改寫成

$$
f(x)= \frac{1}{2\pi}\Big(\frac{1}{k}\sum_{n=-\infty}^{\infty}f(\frac{n}{k})e^{\frac{int}{k}}\Big)
$$

換個角度來說，如果今天一個無週期的函數，我們將其視為週期無窮大，則便可得其傅立葉變換為

$$
f(x)= \frac{1}{2\pi}\Big(\lim_{k\rightarrow\infty}\big[\frac{1}{k}\sum_{n=-\infty}^{\infty}f(\frac{n}{k})e^{\frac{int}{k}}\big]\Big)
$$

中括號內是我們高中到大學初等微積分所學的黎曼和，因此只要 $f$ 為區間可積分，則

$$
f(x)= \frac{1}{2\pi}\Big(\lim_{k\rightarrow\infty}\big[\frac{1}{k}\sum_{n=-\infty}^{\infty}f(\frac{n}{k})e^{\frac{int}{k}}\big]\Big)=\frac{1}{2\pi}\int_{-\infty}^{\infty}\hat{f}(w)e^{iwt}dw
$$

這便是我們最常說非週期函數的(連續)傅立葉變換型態，將非週期函數 $f$ 以無限多個實數頻率的週期函數 $\hat{f}$ 來進行表徵。 

傅立葉變換的物理意義
---

現在我們已經知道了整個傅立葉變換的原理，那我們就要來看看在實際生活中傅立葉變換的意義在哪裡。

![](https://i.imgur.com/b24lApz.gif)
( 圖片來源 : [傅立葉變換 wiki](https://zh.wikipedia.org/zh-tw/%E5%82%85%E9%87%8C%E5%8F%B6%E5%8F%98%E6%8D%A2) )

上圖是傅立葉變換的維基百科中的一張 Gif 動畫圖，如果我們將紅色的 $f$ 視為聲波，經由傅立葉變換後我們可以視為是多個藍色的 $\hat{f}$ 組合，原本聲波是一個時(空)域的函數，經由傅立葉變換後我們可得其頻域。

簡單來說，傅立葉變換就是一個由時空域變換到頻域的工具。

在物理意義上來說，通常時空域的訊號繁雜且難以處理，我們利用傅立葉變換將其轉換成頻域，在頻域中，藉由不同頻率的週期函數進行處理、加工，最後藉由傅立葉變換為可逆的特性，還原成時空域上的訊號。

除此之外，時空域上的訊號傳換成頻域後，幾乎都會發現到絕大多數的頻率在頻域上都接近 0，也就是說，大部分的資訊量都集中在某些頻率中。藉由傅立葉變換，我們可以找出訊息量較高的頻率來做分析即可，其餘的頻率均可忽略不計。

以一個非週期性的 2 維圖像來說，其傅立葉變換後會發現訊號幾乎都集中在某一個部分，藉此我們就可以針對這些頻率進行圖像上的分析

![](https://i.imgur.com/OO2NHxS.jpg)
(圖像來源 : CMU 2017 Fall Computational Photography Course 15-463)

小結
---

傅立葉級數/變換筆者一直都想找機會弄清楚，這次剛好藉由 Nyquist-Shannon Theorem 把之前的思緒整理一下。傅立葉變換，是 Nyquist-Shannon Theorem  的核心概念，我們將在下一篇來了解一下去樣定理究竟在說些什麼。

參考資料
---
1. Morris Kline . (1972) . *Mathematical thought : from ancient to modern times, Vol.3*. Oxford University Press, USA.
2. Tom M.Apostol. (1981). *Mathematical Analysis (2nd ed.)*. Addison Wesley.
3. 單維彰 (1998)。傅立葉級數。*數學傳播，22-1*，81-92。
4. 單維彰 (1998)。傅立葉變換。*數學傳播，24-2*，22-30。
5. 蔡志強 (1999)。積分發展的一頁滄桑。*數學傳播，23-3*，3-20。
6. [從傅立葉級數到快速傅立葉轉換](https://blog.yeshuanova.com/2019/04/fft_intro/)。
7. [An Interactive Introduction to Fourier Transforms.](http://www.jezzamon.com/fourier/index.html).Website.
8. [傅立葉級數](https://zh.wikipedia.org/zh-tw/%E5%82%85%E9%87%8C%E5%8F%B6%E7%BA%A7%E6%95%B0), wikipedia.
9. [傅立葉變換](https://zh.wikipedia.org/zh-tw/%E5%82%85%E9%87%8C%E5%8F%B6%E5%8F%98%E6%8D%A2), wikipedia.
10. [分析一：【函數的Fourier級數1】連續函數的均勻逼近1：Cesàro求和與Fejér定理(以三角級數均勻逼近連續週期函數)](https://www.youtube.com/watch?v=JYylDOlezv8&list=PLil-R4o6jmGhUqtKbZf0LIFKd-xN__g_M&index=15).Youtube.
11. [分析一：【函數的Fourier級數2】分析一：【函數的Fourier級數2】連續函數的均勻逼近2：標準三角函數族在週期為2π的連續函數空間中滿足Parseval條件；Weierstrass逼近定理](https://www.youtube.com/watch?v=tIx0PlABlHA&list=PLil-R4o6jmGhUqtKbZf0LIFKd-xN__g_M&index=16).Youtube.
12. [分析一：【函數的Fourier級數3】Fourier級數收斂定理](https://www.youtube.com/watch?v=6PbZlnNJYyE&list=PLil-R4o6jmGhUqtKbZf0LIFKd-xN__g_M&index=17).Youtube.


註解
---
[^1]: 傅立葉將傅立葉級數的概念發表於西元 1822 年 《熱的解析理論》( Théorie analytique de la chaleur )之中，書中利用傅立葉級數來解決熱傳播的相關問題。然而傅立葉級數的發表過程其實非常坎坷， 1811 年傅立葉向巴黎科學院提交關於熱傳導的論文，被當時的數學家 Lagrange, Laplace 及 Legendre 所拒絕，因此傅立葉後來將所有研究發表於自己的 《熱的解析理論》 一書中，直到兩年後，傅立葉成為巴黎科學院的秘書後，才得以經 1811 年的論文發表在巴黎科學院的《研究報告》 ( Memoires ) 中。