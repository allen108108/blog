---
title: L1 , L2 Regularization 到底正則化了什麼 ?
date: 2019-10-22 16:06:26
categories:
- 深度學習 Deep Learning
image: https://i.imgur.com/3UV63nV.png
mathjax: true
---



在機器學習中藉由降低 error 來訓練出一個能對未知資料有準確預測的 model，在利用訓練資料優化 error function 的過程中，不意外的會得到一個 error 極低的 model ，但是往往在測試資料上卻會有出乎意外的高 error

會造成這樣的原因是，我們利用訓練資料而得到的模型太過於擬合「訓練資料」本身，反而偏離了一般化的目標，我們稱這現象為 " Overfitting "

<!-- more -->

![](https://i.imgur.com/8Gnrd1m.png)
(圖片來源 : [Memorizing is not learning! — 6 tricks to prevent overfitting in machine learning.](https://hackernoon.com/memorizing-is-not-learning-6-tricks-to-prevent-overfitting-in-machine-learning-820b091dc42))

從上圖我們可以理解，在機器學習中，我們或許願意以一些準確度來交換一般性，使其在測試資料上的準確度可以提高。

要達成這樣的目標使用的方法就是 --- Regularization 正則化

## Regularization 

正則化我們最常使用的就是 L1 Regularization & L2 Regularization，這兩種方式其實就是在 Loss Function 中加上對應的 L1 及 L2 penalty (懲罰項)

L1 Penalty : $\lambda\sum\limits_{i=1}^{d}\mid w_i\mid$
L2 Penalty : $\lambda\sum\limits_{i=1}^{d}w_i^2$

現在的問題是，為什麼 regularization 可以解決 overfitting 的問題 ?

如果你 google "regularization" 很常會看到以下兩個圖來試圖解釋正則化的運作原理，但我想很多人可能不見得理解

![](https://i.imgur.com/3UV63nV.png)


要解釋這兩個圖，我想從數學的角度來解釋或許會比較清楚。

## Lagrange 乘數法

在微積分裡面有講到，當我們想要尋找函數 $f(x_1,x_2,\ldots,x_n)$ 的極值 ( minimum or maximum ) 時，我們可以藉由 $\nabla f(x_1,x_2,\ldots,x_n)\overset{let}{=}0$ 來找出使極值存在的 $(x_1,x_2,\ldots,x_n)$，前提是 $(x_1,x_2,\ldots,x_n)$ 並不存在任何限制。

倘若，$(x_1,x_2,\ldots,x_n)$ 必須符合某些限制 $\begin{cases}g_1(x_1,x_2,\ldots,x_n)=0(\leq0)\\g_2(x_1,x_2,\ldots,x_n)=0(\leq0)\\\vdots\\g_m(x_1,x_2,\ldots,x_n)=0(\leq0)\end{cases}$ ，這樣子我們便無法直接對 $f$ 求導，因為我們無法確定令其導數為0得到之解是否可以滿足這些條件。

而 Lagrange 乘數法便是讓我們解決這類有條件的極值問題 : 

令 $h(x_1,x_2,\ldots,x_n)=f+\sum\limits_{\forall m}\lambda_mg_m$ ，然後對 $h$ 求取梯度且令其為0來找出 $x_1,x_2,\ldots,x_n$ 與 $\lambda_m$的關係式後再帶回條件式中求得最佳解( 不難證明經過 Lagrange 乘數法處理之後的解並不會跑掉 ) 

但這邊需要注意的地方是，條件式為不等式之解並非 closed-form ，我們可以用許多優化演算法來找出最優解，而這個最佳解必定要符合 Karush-Kuhn-Tucker Condition。

了解了 Lagrange 乘數法是為了解決有條件的極值問題，那麼，他應該就可以拿來解決我們的 overfitting 問題。

## 將 Lagrange 乘數法與 Regularization 做連結

我們一開始有解釋，如果我們對 Loss function 求極值而得到最低的 error，到最後可能會產生 overfitting，於是我們的思考方向是，那我不要無條件的求得最低 error，我願意犧牲一點精準度來換取一個更泛化的模型。

那.....我們來給這些權重解加一點條件好了 ?

假設他們必須滿足 $w_1^2+w_2^2+\ldots+w_d^2\leq r^2$ 如何?
從幾何意義來看，我要限制這些解必須要在一個過原點的圓柱體內。

![](https://i.imgur.com/aleM87M.jpg)
(藍色為 loss function ，紅色則為我們限制的條件)

我們換個角度來看，是不是似曾相似 ?

<img width=500 src="https://i.imgur.com/oxsSI4z.jpg" >
<img width=500 src="https://i.imgur.com/WK4qrVu.png" >



沒錯，我們google出來大家常拿來解釋的圖形便為 loss function 與限制的 2D 投影，而 L2 regularization 便是我們為了不讓整個優化過程走的太遠、太精準而加上一個限制解必須在圓柱體內的條件。

所以我們要求這樣具條件的極限值問題，就可以利用上述的 Lagrange 乘數法將原本的loss function 與我們的條件做結合 :   $Loss(w_0,w_1,\ldots,w_d)+\lambda\sum\limits_{i=1}^{d}w_i^2$ 我們可以看到，後面的條件便為我們最開始介紹的 L2 penalty[^1][^2]

同樣的道理，如果我們將限制的條件改為 $\mid w_1\mid+\mid w_2\mid+\ldots+\mid w_d\mid$，幾何意義上就是把圓柱體的限制改為中心為原點且頂點都在三軸上的正四角柱體內。

## $\lambda 的影響$

經過了 Lagrange 乘數法的轉換後，Loss Function 加上 Penalty後便多了一個參數 $\lambda$，這個參數在 sklearn 裡面我們常用的參數是 $\alpha$，不管是哪一個符號其實都是在針對同樣的東西 --- Lagrange Multiplier。

那麼這個參數的大小對我們的 error 優化會產生怎麼樣子的影響呢?

如果 $\lambda=0$ ，那便等同於我們沒有做正則化，最後的 model 就可能會是 Overfitting 。 那麼當 $\lambda$ 一直變大，整個　model 便是從 overfitting $\rightarrow$ generalization $\rightarrow$ undefitting， 如下圖所示 : 

![](https://i.imgur.com/4cGQ7a4.png)




## 除了 L1 L2 以外的 Regularization 

既然 Regularization 是讓整個 Error 的優化過程不要走到最後，那麼除了加上 L1 或 L2 的 regularizer ( 我們也稱之為 Weight-decay regularization ) 還有什麼樣子的 regularization 可以避免 overfitting ?

以下出自林軒田機器學習技法第16講內容 : 

![](https://i.imgur.com/SlHCNdX.png)

我們可以把我們的超平面變的厚一點，可以容納更多的 Noise ( SVM )，也可以利用投票的方式來把整個 error 變的平均一點 ( bagging、Random Forest )，2; 我們也可以提早結束整個優化過程 ( NNet )....

從上面的整理我們可以發現，其實我們的很多演算法內，已經包含了 regularization 在其中，型態可能不像 L1 L2 如此明顯，但都在避免 overfitting 的發生。

註釋
---

[^1]:
其實這段寫得很簡單，不等式的 constraint ，最優解並非 closed-form，我們可以用許多的優化算法來找出解，在優化理論中，Karush-Kuhn-Tucker( KKT ) condition 則是非線性規劃中最佳解的必要條件。

[^2]:
如果仔細一點你會發現 : 
(1) penalty當中沒有 $w_0$ 項 
(2)$\lambda r^2$ 不見了
$w_0$ 這個參數 ( threshold )在計算 error 的時候就會被消去，因此不需要考慮。
至於 $\lambda r^2$ 這部分從 KKT condition 可以證明沒有這項的最佳化解仍然相同，並且 $\lambda$ 與半徑 $r$ 成反比。