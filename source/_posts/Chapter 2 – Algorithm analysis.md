---
title: Chapter 2 -- Algorithm analysis
date: 2019-10-03 22:11:15
categories:
- 課程筆記 Course
- 江蕙如 Algorithm
image: https://blog.techbridge.cc/img/kdchang/cs101/algorithm/algorithm-cover.png
mathjax: true
---

Programming、Data Structure 與 Algorithm 其實是一體的三面，在解決問題時通常從高層次的 Algorithm 開始，搭配著 Data Structure 設計好之後再進入 Implementation 的 Programming 部分。

一個好的演算法必須搭配適當的資料結構才能達到高效，而一個好的眼算法只要 Programming 不要太差，往往都能有著還不錯的結果。


<!-- more -->

什麼是好的演算法 ?

1. Corectness : 以證明確認其正確性
2. Efficiency : 執行時間快速 (Time) 且 memory 使用量少 (Space)
$\Longrightarrow$ Time complexity & Space complexity

然而，隨著 input size 的增加，在 resource requirement 上面必然需求會增加，我們在這裡在意的應該是增加的 「幅度」。且，當我們在執行演算法的時候，很難確定會在何種設備下運作，也很難確認我們每一次遇到的 case 的情況是如何，因此在討論演算法效率的時候通常是 Platform-Independent & Instance-Independent。

另外，一個演算法想要進行效率上的比較往往有太多的變因需要討論，input size、演算法的設計都會影響整個解決問題的效率。為了解決這樣的問題，大家便有了一個共識，我們就比較最糟糕的狀況 ( Worse Case ) 下，任意 input size 會造成什麼樣的後果，只要最糟糕的狀況都還能被接受，那麼這樣的演算法應該就不會太差。

這邊給出了一個定義 : 

$An\ algorithm\ is\ efficient\ if\ it\ has\ a\ polynomial\ running\ time.$[^註1][^註2]

這裡不是數學上的 $if$ 或是 $if\ only\ if$ 主要是因為上述定義並沒有這麼嚴謹 : 
1. 若 c,d 兩常數都非常大時，他仍可以被稱為具有 polynomial running time，但整個城市在執行仍然是非常緩慢的
2. 事實上，當一個 Procedure 沒有 polynomial running time，但在執行上仍然可能會有 efficient 的效果 ( 例如 : Simplex Method 、 Unix Grep )，原因在於，這樣的 Procedure 在執行時往往 Worse case 的狀況不會出現。


Asymptotic Order of Growth
---

討論完 efficient 概念後，我們要進入的就是量化的過程。

如果今天有一個 Procedure 可以計算出執行需要 $1.62n^2+3.5n+8$ primitive coputational steps，如果我們利用這個函數來比較其實會造成ㄧ些問題 : 
1. 太過仔細
2. 意義不大 : 既然都是利用 primitive coputational steps 來計算，那麼其差異通常大多都只有係數上的變化，這樣的比較意義並不大。
3. 不容易進行程度上的分組 : 原則上我們還是希望針對效率有一個概括性分類，如果像上式的型態，我們不容易針對個別差異做區分。

因此，引進了三個符號 $O, \Omega, \Theta$，來對 Worse Case running time function $T(n)$ 的上下界做限制。以下是其定義　：　
* $T(n)= O\big(f(n)\big)\Longleftrightarrow\exists\ c>0, n_0>0\ s.t.\ T(n)\leq cf(n),\ \forall n>n_0$
* $T(n)= \Omega\big(f(n)\big)\Longleftrightarrow\exists\ c>0, n_0>0\ s.t.\ T(n)\geq cf(n),\ \forall n>n_0$ 
* $T(n)= \Theta\big(f(n)\big)\Longleftrightarrow T(n)= O\big(f(n)\big)\ and\ T(n)= \Omega\big(f(n)\big)$

在這樣的定義下，有兩個部分需要注意。

1. 我們關注的是當 n 足夠大的時候，整個演算法的執行步驟是否可以被一個函數 Bound 住，因此在前期、或是可觀察到的期間的表現可能不是這麼好也沒關係，這不是我們著重的重點。


![](https://i.imgur.com/127Y2gI.jpg)
(圖片來源 : [Data Structures & Algorithms: Video 2 Big O, Big Omega, and Big Theta](https://www.youtube.com/watch?v=zYNlOKTRd2Y))


2. 在演算法中，$T(n)= O\big(f(n)\big)$ 這樣的表達式似乎已為大眾所接受，但就嚴謹的數學定義上來看，符合條件的 $f(n)$ 顯然有無限多個，而 $O\big(f(n)\big)$ 應該是一個集合的概念。既然如此，一個 function " 等於 " 一個 set，這樣的敘述本身就很奇怪。嚴格來說應該寫成這樣 : $T(n) \in O\big(f(n)\big)$ 會比較貼近其定義。

### Property of $O,\Omega\ and\ \Theta$

* Transitivity : 
$f(n)=\prod(g(n))$ and $g(n)=\prod(h(n))\Longrightarrow f(n)=\prod(h(n))$ , where $\prod=O,\Omega$ or $\Theta$

* Rule of sums : ( Procedure 中每一行 code 的複雜度加總 )
$f(n)+g(n)=\prod\big(\max(f(n),g(n))\big)$ , where $\prod=O,\Omega$ or $\Theta$

* Rule of products : ( 內外迴圈的總複雜度計算 )
$f_1(n)=\prod(g_1(n)$ , $f_2(n)=\prod(g_2(n)\Longrightarrow f_1(n)f_2(n)=\prod(g_1(n)g_2(n))$ , where $\prod=O,\Omega$ or $\Theta$

* Transpose symmetry : 
$f(n)=O\big(g(n)\big)\Longleftrightarrow g(n)=\Omega\big(f(n)\big)$

* Reflexivity : 
$f(n)=\prod(f(n))$ , where $\prod=O,\Omega$ or $\Theta$

* Symmetry : 
$f(n)=\Theta\big(g(n)\big)\Longleftrightarrow g(n)=\Theta\big(f(n)\big)$

以上性質均可從定義證明。

最後，列舉ㄧ些常見的複雜度於下圖


![](https://i.imgur.com/DPg3Dm8.jpg)


紅框代表的是 Polynomial-Time complexity，在這邊通常會希望至少都要達到 $O(n^2)$ 的複雜度才算是比較理想的演算法。



Polynomial-Time complexity of Gale & Shapley Algorithm
---

回顧一下 Gale & Shapley Algorithm

```text=1
def stableMatching :
    Initialize all m ∈ M and w ∈ W to free 
    # 初始化所有人均為單身
    while ∃ free man m who still has a woman w to propose to 
       w = first woman on m's list to whom m has not yet proposed
       # 男生要進行 Propose 必然會先從名單上最高分的女性進行 Propose
       if w is free
         (m, w) become engaged
       # 如果女性單身，則配對
       else some pair (m', w) already exists
         if w prefers m to m'
            m' becomes free
           (m, w) become engaged 
         else
           (m', w) remain engaged
       # 若女性已被有配對，則比較兩位男性對於這女性的名單 ranking　高低來決定與誰配對
    return set(S) of engaged pairs 
```

因為 Rule of sums ，我們可以分解上面的步驟來計算個別複雜度再加總。

``` python=2
Initialize all m ∈ M and w ∈ W to free 
```
初始化所有的男女，因此只須掃過全部的人就可以完成，複雜度為 $O(n)$

```text=17
    return set(S) of engaged pairs 
```
僅僅 output 每一對的結果，複雜度也是 $O(n)$

之前我們有證明過整個 Gale & Shapley Algorithm 最多會執行 $n^2$ 次 iterations，那麼我們便期望在演算法中的 while loop 內部的步驟都可以限制再 $O(1)$ ，這樣整個 Algorithm $n^2$ iterations 後就可以被限制在 $O(n^2)$

```text=4
∃ free man m who still has a woman w to propose to
```

演算法要配合合適的資料結構才能達到高效。

在這一個部分
每一個男女的 ranking list 我們可以用 array 來儲存，因為其長度固定都是 n。
但這邊我們想要儲存 free men 就不適合用 array，因為 free men 的名單會動態增減，並非固定。
所以這邊比較適合使用 Singly-linked list (單向鏈表) 來存儲 free men list。



![](https://i.imgur.com/AucQ56Z.png)


只要有人恢復 free man 角色，或是有對象 engaged ，都可以利用 Singly-linked list 的插入、取出，來進行更動。

但在這一行，真正重要的是有沒有存在一個 free man，我們關心的只是 head 有沒有指向一筆資料。這跟你有多少人都沒有關係，這樣的複雜度就是 $O(1)$。

```text=5
w = first woman on m's list to whom m has not yet proposed
# 男生要進行 Propose 必然會先從名單上最高分的女性進行 Propose
if w is free
    (m, w) become engaged
    # 如果女性單身，則配對
```

這裡我們要個別儲存兩個 array : NEXT & CURRENT，初始值為 1 及 0。

NEXT ，長度為 n，其中每一個元素代表的是此男性下一個 propose 的對象是他自己 ranking list 的第幾名。又，每一個男性必然會先跟他心目中第一名的對象 propose，所以初始值均為 1。

而 CURRENT 中元素代表的即為此女性目前與她的 ranking list 中第幾名 engage。一開始所有女性均為 free，因此初始值均為 0。

在這裡，演算法只需要去 check 某男性在 NEXT 中接下來要跟誰 propose，而此女性是否為 free，這樣的動作依然與男女人數無關，複雜度為 $O(1)$。

```text=10
else some pair (m', w) already exists
    if w prefers m to m'
        m' becomes free
        (m, w) become engaged 
    else
        (m', w) remain engaged
# 若女性已被有配對，則比較兩位男性對於這女性的名單 ranking　高低來決定與誰配對
```
如果今天有兩位男生對某個女生 propose，女方選擇要拒絕誰的依據就是 ranking list，必須比較兩位男性的優先順序，worse case 就是要掃過一整輪的 ranking list ( worse case : 有一個男生在ranking list 上是最後一名，那她就要從第一名看到最後一名才能確認這個男生的名次 )，也就是 $O(n)$ 複雜度，這樣太不符合期待。

因此我們要將資料 ( 男生-名次 ) 做一點重新排列，不要按照 ranking 排列，改為按照男生原本的排序來排列

![](https://i.imgur.com/QicnFsM.png)

這樣的排列，女方只要知道是誰對她 propose 之後就立刻可以知道名次，這跟男生人數有多少人也無關，複雜度就是 $O(1)$。


統整一下整個 Gale & Shapley Algorithm 的複雜度 : 
* 初始以及回傳複雜度都是 $O(n)$
* $n^2$ iterations 的迴圈內部複雜度都是 $O(1)$，這個迴圈部分的複雜度就是 $O(n^2)$

所以最後 Gale & Shapley Algorithm 的複雜度便是 $O(n^2)$

從分析 Gale & Shapley Algorithm 複雜度的過程我們可以了解，即便同一個 Procedure 讓不同的人寫成程式都會有不同的運算效率，因為資料結構選擇的方式不同。

一個好的演算法，必須搭配一個好的資料結構，才能發揮最好的效率。

註釋
---

[^註1]
Polynomial running time : 
$\forall\ input\ size\ N, \exists\ c>0, d>0\ s.t.\ it's\ running\ time\ is\ bounded\ by\ cN^d\ primitive\ computational\ steps.$

[^註2]
Primitive coputational step : 
在一個 Procedure 中，步驟可以被切分成數個無關 input size 的子步驟，這樣的子步驟稱之。
