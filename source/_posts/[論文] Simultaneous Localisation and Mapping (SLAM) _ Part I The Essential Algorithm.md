---
title: "[論文] Simultaneous Localisation and Mapping (SLAM) : Part I The Essential Algorithm" 
date: 2020-03-02 15:35:54
categories:
- 論文 Paper
image: https://i.imgur.com/qZ7jeQN.gif
mathjax: true
---

筆者前言
---

此篇論文距今已有十多年歷史，但就初入 SLAM 領域的初心者來說，這仍是一篇入門的論文可讓我們以極短時間了解 SLAM 的整個核心概念與一些經典的解決方法。但雖說入門，仍然需要不少的先備知識 (例如 : 機率統計、卡爾曼濾波器、粒子濾波器....) ，要能完全掌握也必須要花一些時間。

比較需要注意的是，這篇真的僅是入門，不能作為理解 SLAM 問題的全貌，畢竟在 2006年以後到現在，十多年的時間也發展了不少新技術，更有甚者結合了 Deep Learning 有更好的 performance，但我相信在此論文的基礎下，去了解最新技術會相對來說簡單一些，而這也是我會選擇閱讀此篇論文的主要原因之一。

另外，本篇論文筆記會加入一些補充資料在其中，以利完全沒有相關經驗及先備知識的讀者在進行論文閱讀時能更加清楚整個論文的內容。因此整篇論文筆記的內容或許會與原文有一些差異，並不會完全相同，在對照原始論文時可能要稍微注意的地方。

<!-- more -->


簡介 Introduce
---

若將一個移動式機器人置於一未知環境中，是否能讓其同時進行定位並建立此環境的精確地圖，這一直是在機器人領域中一個重要的問題，這種問題我們統稱為 SLAM ( Simultaneous Localisation and Mapping )。

在此篇論文提出的近十年中 ( 約 1996-2006)，SLAM 問題已有不少進展，在這些進展底下看似已經出現 SLAM 問題的解決方案，但就現實來說仍然存在不少問題。

在這篇論文中，第一與第二部分主要介紹 SLAM 的歷史與演進，第三部分定義出了 SLAM 之架構，第四部份則是針對這樣的 SLAM 架構介紹當時兩種主流解決方案 :以擴展卡爾曼波器 Extended Kalman Filter 為基礎的 EKF-SLAM 與以粒子濾波器為基礎的 FastSLAM。

下一篇論文 "*Simultaneous Localisation and Mapping (SLAM) : Part II State of the Art*" 則是會介紹當時較為先進的做法以及一些 SLAM 的相關實現。( 此篇論文筆者暫無計畫閱讀 )





歷史  History of the SLAM Problem
---

最早 SLAM 概念發展主要是在 1986 年於舊金山舉行的 IEEE Robotics and Automation Conference 會議上，在這個時候，機率概念正被引入到 AI 與機器人領域之中，也因此許多研究人員也努力想利用估計的方式處理定位及製圖的問題。

在當時取得一致的共識，同意機率映射 (Probablistic mapping) 是在機器人技術中的一個重要概念，並且可以應用在許多的計算問題之中。在後續的幾年中，產出了相當多的關鍵論文，在這個領域中建立了完整的統計基礎，其中最重要的是，確定了定位的估計與地圖中不同地標應具有高度相關性，並且這樣的相關性會因後續持續的觀測而增長。
 
在 " *Estimating uncertain spatial relationships in robotics.* " 這篇關鍵論文中也說明了當移動式機器人在未知環境中移動觀測時，針對這些地標的估計值會因為定位產生的一般性誤差而具有相關性。這項發現表明了，定位跟製圖這兩個問題必須要一起解決，也就是說，移動式機器人本身的定位與地標的估計應視為一個聯合狀態 (joint state)，在每一次地標觀測後都進行一次整體的更新。但，這也表示說狀態向量的維度會非常高 ( 除了定位的座標資訊外，還要包含所有的地標資訊 )，而且計算的複雜度將會提高到地標數量的平方。

這樣的研究方向非常好，但後來卻停滯不前。主要的原因是當時的人們並不認為持續觀察的誤差會收斂，最可能的狀況是地圖的估計誤差會持續增長並沒有上界，如果要避免這樣的狀況，研究人員就必須假設或強迫所有地標估計之間的相關性必須要盡可能的小或是甚至無相關性，最終會導致地標估計與定位估計會分開來處理。

但隨著整個概念的進展，大家開始意識到將定位以及地標估計統一進行估計，事實上估計值會收斂，且前述提到的相關性越高反而會讓解決方案越成功。

SLAM 這個名詞的發表主要是在 1995 年 International Symposium on Robotics Research 會議上，不管是其收斂性或是製圖定位的問題都有不同的單位著手進行。

目前 (此篇論文提出的當時)，SLAM 的研究方向大多在計算效率的提升以及解決資料間關聯性或閉環問題。而 SLAM 會議在 1999 年第一次舉行，著重在融合 Kalman-Filter、機率定位 (Probabilistic localisation) 以及製圖方法 (Mapping method)，後續的 SLAM 相關會議也吸引越來越多研究人員參與。

( 補充 ) 定位與測量 Localisation and Measurement 
---

在正式進入 SLAM 的領域前，先來談一下機器人怎麼利用觀測來為自己進行定位。我們先用一張圖，來非常概要的解釋整個機器人、自駕車的定位概念。


![](https://i.imgur.com/gKhc8hV.png)

* 最上面描繪出一個一維的機器人運動，在移動過程中會對周遭的環境進行測量 (measure)，在這個例子中，機器人會去觀測是否有看到綠色的門。
* BELIEF 指的是在這一維的 location 中，機器人的定位初始分佈，一開始並不確定機器人位置在哪，因此初始分佈會是呈現 Uniform 狀態。
* 後驗分佈 Posterior Distribution 即當機器人第一次觀測到綠色門時，機器人的定位分佈會有所變化，機器人的定位會有較高的機率在綠色門位置的地方。之所以稱為 Posterior 因為這個分佈是考慮觀測 (measurement) 後得到的機率分佈。
* 當機器人往右移動時，機率分佈也會隨之平移，這很合理，此時的機器人會有較高機率出現在綠門位置的旁邊一點點。這樣的分佈並非觀測後而得到的分佈，因此可以做為下一次觀測前的先驗分佈 Perior Distribution。
* 機器人繼續往右移動，神奇的是因為連續觀察到兩個綠色門，因此這個後驗分佈會在第二扇門的地方達到最高波峰，換句話說，也就是更確定了機器人的定位在哪個位置。

從上面的例子，我們可以知道，在機器人定位的理論當中，其實就是兩個步驟在重複運作 : 觀測 Measurement 與移動 Motion。藉由觀測獲得後驗機率，再利用移動的先驗機率預測移動的路徑，如此一直重複進行。

接下來，我們利用簡單的、離散的例子來理解這整個過程的數學原理

### 觀測 Measurement

![](https://i.imgur.com/AfVYGqc.png)

* 初始的 Belief 假設均為 0.2
* 當移動機器人在某個位置中有觀測到紅色的 Landmark 時，會給予紅色地標比較高的分數 (此例假設為 0.6)，非紅色的地標則給予較低的分數 (0.2)
* 此時，對於此機器人位置的後驗機率 $P(x_i\mid Z)$ 便會經由初始分佈與觀測的比重之乘積產生


### 移動 Motion

機器人的移動最麻煩的地方就是無法有一個確切的路徑，我們可以說，它們的移動具有不確定性。


![](https://i.imgur.com/McDwVG8.png)


* 上圖敘述了一維運動的不確定性。
* 當我們希望機器人以步伐為 1 的方式運動，我們可以預期他們會有很高的機率( 此例假設 0.8 ) 一次移動一個單位。當我們希望機器人以步伐為 2 的方式運動，我們可以預期他們會有很高的機率( 此例假設 0.8 ) 一次移動兩個單位。

了解了機器人定位的步驟與方法後，我們可以開始進入論文的最關鍵部分。


公式化及結構 Formulation and Structure of the SLAM Problem
---

SLAM 是一個過程，在無須任何先驗知識下，移動機器人可以利用這樣的過程進行地圖建置並同時於此地圖中推斷其位置。

![](https://i.imgur.com/WOQ6Vtp.png)

###  A. 預備知識 Preliminaries

這裡定義幾個符號 : 
* $\mathrm{x}_k$ 移動設備本身的位置向量及其面向之方向
* $\mathrm{u}_k$ 控制向量，可使移動設備自 $\mathrm{x}_{k-1}$ 移動至 $\mathrm{x}_k$
* $\mathrm{m}_i$ 第 $i$ 個地標的位置向量
* $\mathrm{z}_{ik}$ 移動設備在時間 $k$ 時針對第 $i$ 個地標進行的觀測，若每個時間點都針對多個地標做觀測可簡寫為 $\mathrm{z}_k$
* $\mathrm{X}_{0:k}=\{\mathrm{x}_0,\mathrm{x}_1,\cdots,\mathrm{x}_k\}=\{\mathrm{X}_{0:k-1},\mathrm{x}_k \}$ 移動設備在 $k$ 時間內的移動歷程
* $\mathrm{U}_{0:k}=\{\mathrm{u}_1,\mathrm{u}_2,\cdots,\mathrm{u}_k\}=\{\mathrm{U}_{0:k-1},\mathrm{u}_k \}$ 在 $k$ 時間內的控制輸入歷程
* $\mathrm{m}=\{\mathrm{m}_1,\mathrm{m}_2,\cdots,\mathrm{m}_n\}$ 所有的地標集合
* $\mathrm{Z}_{0:k}=\{\mathrm{z}_1,\mathrm{z}_2,\cdots,\mathrm{z}_k\}=\{\mathrm{Z}_{0:k-1},\mathrm{z}_k \}$ 移動設備在 $k$ 時間內的觀測歷程

### B. Probabilistic SLAM

首先，我們先引入一個分佈

$$
P(\mathrm{x}_k,\mathrm{m}\mid \mathrm{Z}_{0:k},\mathrm{U}_{0:k},\mathrm{x}_0)
$$

我們假設，在任一個時間點 $k$ 時，在已知起始點 $\mathrm{x}_0$、觀測 $\mathrm{Z}_{0:k}$ 以及控制輸入 $\mathrm{U}_{0:k}$ 的條件下，移動設備對於自身的定位 $\mathrm{x}_k$ 與 所有地標的估計 $\mathrm{m}$ 均服從上述的聯合後驗分佈 ( joint posterior distribution )。

而我們也很直觀的可以了解到這樣的機率分佈其時是一個遞迴的狀態，也就是說

$$
P(\mathrm{x}_{k-1},\mathrm{m}\mid \mathrm{Z}_{0:k-1},\mathrm{U}_{0:k-1},\mathrm{x}_0)\overset{\mathrm{u}_k,\mathrm{z}_k}{\longrightarrow}P(\mathrm{x}_k,\mathrm{m}\mid \mathrm{Z}_{0:k},\mathrm{U}_{0:k},\mathrm{x}_0)
$$

這樣的過程應該也要符合貝式定理 ( Bayes Theorem )。然而上述的過程，看似一步驟完成，實際上是由兩個步驟所組合而成 : 移動加上觀測。

移動設備的移動過程，這樣的過程需要一個模型來進行描述，我們稱之為 Motion Model，其遵從馬可夫假設 ( Markov Assumptions ) : 

$$
P(\mathrm{x}_k\mid \mathrm{x}_{k-1},\mathrm{m},\mathrm{Z}_{0:k-1},\mathrm{U}_{0:k},\mathrm{x}_0)=P(\mathrm{x}_k\mid \mathrm{x}_{k-1},\mathrm{u}_k)
$$

而觀測也會需要一個觀測模型 Observation Model 來進行描述，亦遵從馬可夫假設 : 

$$
P(\mathrm{z}_k\mid \mathrm{x}_{k},\mathrm{m},\mathrm{Z}_{0:k-1},\mathrm{U}_{0:k},\mathrm{x}_0)=P(\mathrm{z}_k\mid \mathrm{x}_k,\mathrm{m})
$$

![](https://i.imgur.com/bG9Au14.png)


這兩個分佈都很直覺，不需要太多解釋，現在我們利用這兩個步驟可以來描述一個 SLAM 演算法了，第一步驟我們稱為 **Time-Update**

$$
P(\mathrm{x}_k,\mathrm{m}\mid \mathrm{Z}_{0:k-1},\mathrm{U}_{0:k},\mathrm{x}_0)=\int P(\mathrm{x}_k\mid \mathrm{x}_{k-1},\mathrm{u}_k)\cdot P(\mathrm{x}_{k-1},\mathrm{m}\mid \mathrm{Z}_{0:k-1},\mathrm{U}_{0:k-1},\mathrm{x}_0)d\mathrm{x}_{k-1}
$$

第二步驟則稱之為 **Measurement-Update**

$$
P(\mathrm{x}_k,\mathrm{m}\mid \mathrm{Z}_{0:k},\mathrm{U}_{0:k},\mathrm{x}_0)=\displaystyle{\frac{P(\mathrm{z}_k\mid \mathrm{x}_k,\mathrm{m})P(\mathrm{x}_k,\mathrm{m}\mid \mathrm{Z}_{0:k-1},\mathrm{U}_{0:k},\mathrm{x}_0)}{P(\mathrm{z}_k\mid \mathrm{Z}_{0:k-1},\mathrm{U}_{0:k})}}
$$


### Structure of Probabilistic SLAM

為了簡化討論過程，作者會將符號做一些省略，像最前面提到的聯合後驗分佈

$$
P(\mathrm{x}_k,\mathrm{m}\mid \mathrm{Z}_{0:k},\mathrm{U}_{0:k},\mathrm{x}_0)
$$

可能會簡化成為 $P(\mathrm{x}_k,\mathrm{m}\mid \mathrm{z}_k)$ 或 $P(\mathrm{x}_k,\mathrm{m})$，這裡比較需要注意的地方在於，我們由 Observation Model 中可以看出 $\mathrm{x}_k$ 與 $\mathrm{m}$ 並非互斥，因此這個聯合後驗分佈不能簡化成以下形式

$$
P(\mathrm{x}_k,\mathrm{m}\mid \mathrm{z}_k)\neq P(\mathrm{x}_k\mid \mathrm{z}_k)\cdot P(\mathrm{m}\mid \mathrm{z}_k)
$$


回到上面的 Fig. 1.，其實不難發現誤差的出現是很常見的現象，而且幾乎都是因為移動設備本身定位誤差所造成。雖然我們很難準確估測某個地標的位置，但基本上對於地標與地標之間相對位置的預測都不太會有太大的誤差。

從機率的形式來看上述的現象，無法準確估測某一個地標 $\mathrm{m}_i$，可以看成其分佈 $P(\mathrm{m}_i)$ 非常的分散，但是對於 $\mathrm{m}_i, \mathrm{m}_j$ 的相對位置的估測仍然準確，就表明了 $P(\mathrm{m}_i,\mathrm{m}_j)$ 的分佈集中。

在 SLAM 的相關研究中，最重要的是理解到隨著觀測的增加，地標估計的相關性會單調增加，也就是說，就機率的角度來說，當觀測數量越來越多，$P(\mathrm{m})=P(\mathrm{m}_1,\mathrm{m}_1,\cdots,\mathrm{m}_n)$ 的分佈會越來越集中，就論文的說法就是 

> $P(\mathrm{m})\text{ becomes monotonically more peaked as more observations are made.}$


這種估測的收斂性最主要是因為移動設備對於自身定位估測與地標估測兩者間是「幾乎獨立」的。

再回到 Fig. 1. 來看，當估計自身的位置於 $\mathrm{x}_k$ 時，會觀察到 $\mathrm{m}_i$ 與 $\mathrm{m}_j$ 兩個地標，這兩個地標的相對位置顯然獨立於移動設備本身的坐標系之外。當移動設備繼續前進至 $\mathrm{x}_{k+1}$，此時只會觀測到 $\mathrm{m}_j$ 這一個地標，藉由這一次的觀測可以對地標的估測位置來進行更新。

雖說一次並沒有觀測到 $\mathrm{m}_i$，但因為 $\mathrm{m}_i$ 與 $\mathrm{m}_j$ 兩者的相對位置獨立坐標系外，我們仍然可以藉由這一次的觀測同時更新 $\mathrm{m}_i$ 與 $\mathrm{m}_j$，也同時增加這兩個地標的估測之間相關性。

![](https://i.imgur.com/KYXZU8r.png)

上圖其時可以看出來，在觀測初期，相對位置接近的地標，相關性會高很多，反之，距離較遠的相關性則低，這樣的訊息很直接，因為距離相近的，在觀測時容易被同時觀測到，爾後不管是否同時被設備觀測到，也能一同更新估測的位置。但隨著移動設備的來回走動，不管距離遠近，地標之間相關性都會單調增加。



解決方式 Soutions of the SLAM Problem
---

要解決 SLAM 的問題，最主要的核心就是要找到最適合的 Observation Model 與 Motion Model 表示法，來提供 Time-Update 與 Measurement-Update 有效率且一致的計算。

***[ 補充 ] : 上面的描述看起來很簡單，反正就是找兩個方程式來處理就好，然而，這中間最麻煩的問題就是 Noise。雖然理想化狀態相關性會隨觀測次數而遞增，但相對來說 Noise 也是，當一開始有 Noise 造成估測誤差後，後面的地標估測會因為相關性而傳遞誤差甚至疊加各自的誤差。這就是整個 SLAM 問題最需要解決的部分，也因此才會需要用到 「濾波器」的概念。***

在這一個部分主要介紹兩個經典的方法 : 

* 加入 Gaussian Noise 建立的狀態模型，進而利用擴展卡爾曼濾波器來解決 SLAM 問題，我們稱之為 EKF-SLAM
* 將移動設備模型描述成一個更通用的非高斯分佈模型 ( Non-Gaussian Distribution )，進而利用 RB 粒子濾波器 ( Rao-Blackwellised Particle Filter ) 來解決 SLAM 問題，我們稱之為 Fast-SLAM。

### 擴展卡爾曼濾波器 Extended Kalman Filter ( EKF-SLAM )

( 在進入以下論文內容前，建議先閱讀 Bzarg 的文章 " [How a Kalman filter works, in pictures](https://www.bzarg.com/p/how-a-kalman-filter-works-in-pictures/) " 以及 [Udacity - Artificial Intelligence for Robotics](https://www.udacity.com/course/artificial-intelligence-for-robotics--cs373)，這些應該是目前最通俗易懂的 Kalman Filter 解釋文章 / 課程了。 )

在 EKF-SLAM 中，我們利用下面的方式為 Motion 與 Observation 建模

$$
\mathrm{x}_k=f(\mathrm{x}_{k-1},\mathrm{u}_k)+\mathrm{w}_k\cdots\cdots(1)\\
\mathrm{z}_k=h(\mathrm{x}_{k},\mathrm{m})+\mathrm{v}_k\cdots\cdots(2)\\
$$

其中，$\mathrm{w}_k$ 與 $\mathrm{v}_k$ 是一個均值為 $0$ 並具有共變異數各為 $\mathrm{Q}_k$ 與 $\mathrm{R}_k$ 的高斯擾動，白話一點說，就是 Noise 。

$$
\mathrm{w}_k\sim N(0,\mathrm{Q}_k)\\
\mathrm{v}_k\sim N(0,\mathrm{R}_k)
$$

![](https://i.imgur.com/xWI3gdR.png)

左下角為前一個狀態下的分佈，當經過 $f$ 轉換後會呈現另外一個分佈 (上圖橢圓形) ，但轉換過程還會受到 Noise $v_k$ 的影響，因此最終分佈應該會再一次的轉變 ( 當然以 $\mathrm{m}$ 來說亦是如此 )


![](https://i.imgur.com/X7uChRt.png)

既然確定了分佈，就可以找出 $P(\mathrm{x}_k,\mathrm{m}\mid \mathrm{Z}_{0:k},\mathrm{U}_{0:k},\mathrm{x}_0)$ 的期望值及共變異數

$$
\begin{bmatrix}\hat{\mathrm{x}}_{k\mid k}\\ \hat{\mathrm{m}}_k\end{bmatrix}=E\begin{bmatrix}\mathrm{x}_k\\ \mathrm{m}_k \end{bmatrix}\cdots\cdots(3)\\
P_{k\mid k}=\begin{bmatrix}P_{xx}&P_{xm}\\P_{xm}&P_{mm}\end{bmatrix}_{k\mid k}=E\begin{bmatrix}\dbinom{\mathrm{x}_k-\hat{\mathrm{x}}_k}{\mathrm{m}_k-\hat{\mathrm{m}}_k}\dbinom{\mathrm{x}_k-\hat{\mathrm{x}}_k}{\mathrm{m}_k-\hat{\mathrm{m}}_k}^T\end{bmatrix}\cdots\cdots(4)
$$

論文這邊的符號很混亂，且沒有另外說明，原則上都是引用 " *A Solution to the Simultaneous Localization and Map Building (SLAM) Problem* " 這篇論文再加以調整

* $\hat{\mathrm{x}}_{p\mid q}=E\begin{bmatrix}\mathrm{x}_p\mid \mathrm{Z}_{0:q}\end{bmatrix}$ ,where $p\geq q$
* $P_{p\mid q}=E\begin{bmatrix}\big(\hat{\mathrm{x}}_{p\mid q}-\mathrm{x}_p\big)\big(\hat{\mathrm{x}}_{p\mid q}-\mathrm{x}_p\big)^T\mid \mathrm{Z}_q\end{bmatrix}$
* $P_{xx,p\mid q} :$ 在 $p,q$ 兩個狀態下定位誤差之共變異矩陣
* $P_{mm,p\mid q} :$ 在 $p,q$ 兩個狀態下地標估測誤差之共變異矩陣
* $P_{xm,p\mid q} :$ 在 $p,q$ 兩個狀態下定位與地標估測誤差之共變異矩陣


有了這兩部分，就可以進行 Time-Update 與 Measurement-Update
( 這邊的相關知識及理論推導很多，這裡我們大多 follow 論文，再加一點補充，詳細完整證明過程可以參閱 " *Stochastic Models, Estimation and Control Volume 1* " 一書 )

#### Time-Update

若今天有一組一維定位的先驗分佈 $(\mu_1,\sigma^2_1)$，以及「移動」本身的分佈 $(\mu_2,\sigma^2_2)$，很清楚可以得到移動後的分佈 $(\mu',\sigma'^2)$

$$
\mu'=\mu_1+\mu_2\\
\sigma'^2=\sigma^2_1+\sigma^2_2
$$

推廣到高維度的矩陣型態

$$
\mu'=\mu_1+\mu_2\cdots\cdots(5)\\
\Sigma'^2=\Sigma^2_1+\Sigma^2_2\cdots\cdots(6)\\
\text{where }\mu\text{ is mean matrix}, \Sigma\text{ is the covariance matrix}\\
$$

整合 $(1),(3),(4),(5),(6)$ 式便可進行 Time-update


$$
\hat{\mathrm{x}}_{k\mid k-1}=f(\hat{\mathrm{x}}_{k-1\mid k-1},\mathrm{u}_k)\\
P_{xx,k\mid k-1}=\nabla fP_{xx,k-1\mid k-1}\nabla f^T
+Q_k
$$

這裡會取 $f$ 的 Jacobian 主要考量到如果這是一個連續性的運動，所有的變動將會以極小的方式在進行。

#### Observation-update

假如我們今天有一組一維地標的先驗分佈 $(\mu_1,\sigma^2_1)$，以及經過觀測後得到的後驗分佈 $(\mu_2,\sigma^2_2)$，經由 Bayes Theorem 兩者相乘後即可得出下一個時間點地標的先驗分佈 $(\mu',\sigma'^2)$ ，經過數學可以證明

$$
\mu'=\dfrac{\mu_1\sigma^2_2+\mu_2\sigma^2_1}{\sigma^2_1+\sigma^2_2}=\mu_1+\mathrm{k}(\mu_2-\mu_1)\\
\sigma'^2=\dfrac{1}{\frac{1}{\sigma^2_1}+\frac{1}{\sigma^2_2}}=\sigma^2_1-\mathrm{k}\sigma_2^2\\
\text{where }\mathrm{k}=\dfrac{\sigma_1^2}{\sigma_1^2+\sigma_2^2}
$$

我們也可以推廣到多維以矩陣形式來呈現

$$
\mu'=\mu_1+\mathrm{K}(\mu_2-\mu_1)\cdots\cdots(7)\\
\Sigma'=\Sigma_1-\mathrm{K}\Sigma_2\\
\text{where }\mu\text{ is mean matrix}, \Sigma\text{ is the covariance matrix}\\
\text{and }\mathrm{K}\text{ is called
Kalman Gain}
$$


接下來整合 $(2),(3),(4),(7)$ 式便可進行 Observation-update

$$
\begin{bmatrix}\hat{\mathrm{x}}_{k\mid k}\\ \hat{\mathrm{m}_k}\end{bmatrix}=\begin{bmatrix}\hat{\mathrm{x}}_{k\mid k-1}\\ \hat{\mathrm{m}_{k-1}}\end{bmatrix}+K_k\cdot \tilde{y}_k
$$


其中

* 測量剩餘 Measurement Residual $\tilde{\mathrm{y}}_k=\mathrm{z}_k-h(\hat{x}_{k\mid k-1},\mathrm{m}_{k-1})$
* 測量剩餘共變異數 Measurement Residual Covariance $\mathrm{S}_k=\nabla hP_{k\mid k-1}\nabla h^T+R_k$
* 卡爾曼增益 Kalman Gain $\mathrm{K}_k=P_{k\mid k-1}\nabla h^TS^{-1}$


EKF-SLAM 是一種非常經典而且常見的 SLAM 解決方案，其優缺點如下 : 

**[優點]**

* 收斂性 : 在地標估測的過程中，其不確定性 (也就是分佈的標準差) 會收斂至一個下界 

**[缺點]**

* 計算成本 : 每一次進行 Observation-Update 都必須要計算共變異矩陣，這樣的計算複雜度高達 $O(n^2)$ ，但 EKF-SLAM 的變體可以解決這方面的問題。
* 資料關聯性 : 標準的 EFK-SLAM 容易造成地標間估測的錯誤關聯，當移動設備經過一大圈後返回時，這樣的狀況容易造成閉環 (Loop Closure) 不易。
* 通用性 : EKF-SLAM 利用線性模型來描述非線性的 Motion 與 Observation model，唯有在這樣的條件才能確保其收斂性。

### RB 粒子濾波器 Rao-Blackwellised Filter (Fast-SLAM)

開始理解論文之前，我們先用下面這張還蠻有名的動畫來理解整個粒子濾波器的作用是什麼

![](https://i.imgur.com/qZ7jeQN.gif)

* 一開始，我們會對機器人的定位給予一個先驗分佈，由於一開始完全不能確定機器人的位置，因此其定位分佈可能遍布整個領域。
* 隨著機器人的移動以及觀測，每一次移動都會根據觀測後的結果進行重要性採樣 ($SIS$, Sequential Importance Sampling)，篩除掉可能性低的部分。




Fast-SLAM 中，利用了遞迴 Monte-Carlo method 來進行非線性、非高斯分佈的建模過程。此外，在 Fast-SLAM 中，估計的會是整個歷程 (trajectory) ，而不是針對某一個時間點的 Location。這樣做的原因是，可以將自身位置的估計與地標估計視為互相獨立的事件，使得整個地標估計視為一個獨立 Gaussian Distribution，僅具有線性複雜度而不用去計算二次複雜度的聯合地標估計共變異數，這是 Fast-SLAM 之所以 Fast 的原因之一。

結合上面的簡單介紹， Fast-SLAM 在 Motion-Update 上，將每一個抽樣出來的歷程視為一個「粒子」。然後我們要利用隨機的抽樣「粒子」來估測母體，這便是 Monte-Carlo method 的精隨所在。除此之外，在 Measurement-Update Fast-SLAM 將每一個地標估計都視為一個分佈，具有其均值 $\mu$ 與變異數 $\Sigma$。

下圖描述了 Fast-SLAM 的中粒子的基本型態。(PS:對應論文，下標應該改為 $0:t$ )

![](https://i.imgur.com/I2Jyw94.png)



Fast-SLAM 的結構是 Rao-Blackwellised 狀態，這種狀態在某時間點 $k$ 的聯合分佈可由下面的集合表示

$$
\{w_k^{(i)},\mathrm{X}_{0:k}^{(i)},P(\mathrm{m}\mid \mathrm{X}_{0:k}^{(i)},\mathrm{Z}_{0:k})\}_i^N\\
\text{where }P(\mathrm{m}\mid \mathrm{X}_{0:k}^{(i)},\mathrm{Z}_{0:k})=\prod_j^MP(\mathrm{m}_j\mid \mathrm{X}_{0:k}^{(i)},\mathrm{Z}_{0:k})
$$

整個 Fast-SLAM 利用粒子濾波器進行定位更新，而地標預測更新則是使用 EKF 遞迴估計。對任一個粒子 (整個移動歷程)來說，地圖地標位置的更新非常簡單，只要進行單獨的 EFK 更新即可，沒有觀察到的地標估測就不予更新。

至於要預估路徑/整個移動歷程，就相對來說困難許多。

在某一個時間 $T$ 要進行路徑的預估，就必須要有一個先驗分佈，而這一個先驗分佈也不是人為指定的，而是從先前的分佈中遞迴而得

$$
P(\mathrm{X}_{0:T}\mid \mathrm{Z}_{0:T})\\=P(\mathrm{x}_0\mid \mathrm{Z}_{0:T})P(\mathrm{x}_1\mid \mathrm{x}_0,\mathrm{Z}_{0:T})P(\mathrm{x}_2\mid \mathrm{X}_{0:1},\mathrm{Z}_{0:T})P(\mathrm{x}_3\mid \mathrm{X}_{0:2} \mathrm{Z}_{0:T})\cdots P(\mathrm{x}_T\mid \mathrm{X}_{0:T-1} \mathrm{Z}_{0:T})
$$

接著就是要對這個分佈進行 $SIS$，但此時會遇到一個問題，$P(\mathrm{x}_k\mid \mathrm{X}_{0:k-1} \mathrm{Z}_{0:T})$ 這個真實的分佈我們並不知道，怎麼去重新進行採樣 ?

這裡引進了一個 Proposal Distribution $\pi$ ，我們既然無法直接知道 $P$ ，那我們就換一個方向，希望找到一個分佈 $\pi$ 是逼近 $P$ 的

$$
\mathrm{x}_k^{(i)}\sim \pi(\mathrm{x}_k\mid \mathrm{X}_{0:k-1}^{(i)},\mathrm{Z}_{0:k},\mathrm{u}_k)
$$

也藉此可以定義出進行 $SIS$ 所需要的重要性權重

$$
w_{k}^{(i)}=\dfrac{\text{target distribution}}{\text{proposal distribution}}=\dfrac{P(\mathrm{X}_{k}^{(i)}\mid \mathrm{Z}_{0:k},\mathrm{U}_{0:k})}{\pi(\mathrm{X}_{k}^{(i)}\mid \mathrm{Z}_{0:k-1},\mathrm{U}_{0:k})}=w_{k-1}^{(i)}\dfrac{P(\mathrm{z}_k\mid \mathrm{X}_{0:k}^{(i)},\mathrm{Z}_{0:k-1})P(\mathrm{x}_k^{(i)}\mid \mathrm{x}_{k-1}^{(i)},\mathrm{u}_k)}{\pi(\mathrm{x}_{k}^{(i)}\mid \mathrm{X}_{0:k-1}^{(i)},\mathrm{Z}_{0:k},\mathrm{u}_{k})}
$$

藉由上式計算出來的重要性權重重新進行抽樣。

$SIS$ 在每一次更新時都會進行重新抽樣，這使得 Fast-SLAM 有一個很重要的特性，那就是執行過 $SIS$ 後，系統會「遺忘」過去，而使得 Noise 不會持續累積下去。

下圖非常清楚的展示了 Fast-SLAM 的演算過程

![](https://i.imgur.com/BRp7CvP.png)
("*Adaptive prior boosting technique for the efficient sample size in FastSLAM*")

在文獻上，有 Fast-SLAM 1.0 及 Fast-SLAM 2.0 兩種版本，兩者的差別只在於 proposal distribution 的形式以及重要性權重的計算上有所更動。


Fast-SLAM 1.0

$$
\text{Proposal Distribution is Motion Model : } P(\mathrm{x}_k\mid \mathrm{x}_{k-1}^{(i)},\mathrm{u}_k)\\
w_k^i=w_{k-1}^iP(\mathrm{z}_k\mid \mathrm{X}_{0:k},\mathrm{Z}_{0:k-1})
$$

Fast-SLAM 2.0

$$
\text{Proposal Distribution is Motion Model including observation : } P(\mathrm{x}_k\mid \mathrm{X}_{0:k}^{(i)},\mathrm{Z}_{0:k},\mathrm{u}_k)\\
w_k^i=w_{k-1}^iC\\
\text{where }C=\dfrac{P(\mathrm{z}_k\mid \mathrm{x}_k,\mathrm{X}_{0:k-1}^{(i)},\mathrm{Z}_{k-1})P(\mathrm{x}_k\mid \mathrm{x}_{k-1}^{(i)},\mathrm{u}_k)}{P(\mathrm{x}_k\mid \mathrm{X}_{0:k-1}^{(i)},\mathrm{Z}_{k-1},\mathrm{u}_k)}
$$

Fast-SLAM 2.0 的優點在於，Proposal Distribution 是一個局部最佳解，也就是對於每一個粒子，都可以借此計算出一個最小的重要性權重。且可在室外生成準確的地圖。

但，不論是 Fast-SLAM 1.0 或 2.0，當重新採樣次數到達一定程度時，整個演算法會產生退化而失去準確度。

實現 Implementation of SLAM
---

(略)

APPENDIX : SLAM algorithm Derive
---

### Measurement-Update Derive

(略，其實就是 Bayes Theorem )

### Time-Updata Derive

$$
P(\mathrm{x}_k,\mathrm{m}\mid \mathrm{Z}_{0:k},\mathrm{U}_{0:k},\mathrm{x}_0)\propto P(\mathrm{z}_k\mid \mathrm{x}_k,\mathrm{m},\mathrm{Z}_{0:k-1},\mathrm{U}_{0:k},\mathrm{x}_0)\cdot P(\mathrm{x}_k,\mathrm{m},\mathrm{Z}_{0:k-1},\mathrm{U}_{0:k},\mathrm{x}_0)\\
\Big(\because P(B\mid A)=\displaystyle{\frac{P(A\mid B)P(B)}{P(A)}}\Longrightarrow P(B\mid A)\propto P(A\mid B)P(B)\Big)\\
\Longrightarrow P(\mathrm{x}_k,\mathrm{m}\mid \mathrm{Z}_{0:k},\mathrm{U}_{0:k},\mathrm{x}_0)=\eta  P(\mathrm{z}_k\mid \mathrm{x}_k,\mathrm{m},\mathrm{Z}_{0:k-1},\mathrm{U}_{0:k},\mathrm{x}_0)\cdot P(\mathrm{x}_k,\mathrm{m},\mathrm{Z}_{0:k-1},\mathrm{U}_{0:k},\mathrm{x}_0)\\
=\eta P(\mathrm{z}_k\mid \mathrm{x}_k,\mathrm{m})\cdot \int P(\mathrm{x}_k,\mathrm{m},\mathrm{Z}_{0:k-1},\mathrm{U}_{0:k},\mathrm{x}_0,\mathrm{x}_{k-1})d\mathrm{x}_{k-1}\\
\Big(\text{By the definition of marginal probability density function :} f_X(x)=\int f(x,y)dy  \Big)\\
=\eta P(\mathrm{z}_k\mid \mathrm{x}_k,\mathrm{m})\cdot\int P(\mathrm{x}_k\mid \mathrm{x}_{k-1},\mathrm{m},\mathrm{Z}_{0:k-1},\mathrm{U}_{0:k},\mathrm{x}_0)\cdot P(\mathrm{x}_{k-1},\mathrm{m},\mathrm{Z}_{0:k-1},\mathrm{U}_{0:k},\mathrm{x}_0)d\mathrm{x}_{k-1}\\
\Big(\text{By Bayes Theorem}\Big)\\
=\eta P(\mathrm{z}_k\mid \mathrm{x}_k,\mathrm{m})\cdot\int P(\mathrm{x}_k\mid \mathrm{x}_{k-1},\mathrm{m})\cdot P(\mathrm{x}_{k-1},\mathrm{m}\mid \mathrm{Z}_{0:k-1},\mathrm{U}_{0:k},\mathrm{x}_0) \cdot P(\mathrm{Z}_{0:k-1},\mathrm{U}_{0:k},\mathrm{x}_0)d\mathrm{x}_{k-1}\\
=\eta P(\mathrm{z}_k\mid \mathrm{x}_k,\mathrm{m})\cdot\int P(\mathrm{x}_k\mid \mathrm{x}_{k-1},\mathrm{m})\cdot P(\mathrm{x}_{k-1},\mathrm{m}\mid \mathrm{Z}_{0:k-1},\mathrm{U}_{0:k-1},\mathrm{x}_0) \cdot P(\mathrm{Z}_{0:k-1},\mathrm{U}_{0:k},\mathrm{x}_0)d\mathrm{x}_{k-1}\\
\Big(\because (\mathrm{x}_{k-1},\mathrm{m})\text{ is independent from }\mathrm{u}_k\Big)\\
=\eta P(\mathrm{z}_k\mid \mathrm{x}_k,\mathrm{m})\cdot\int P(\mathrm{x}_k\mid \mathrm{x}_{k-1},\mathrm{m})\cdot P(\mathrm{x}_{k-1},\mathrm{m}\mid \mathrm{Z}_{0:k-1},\mathrm{U}_{0:k-1},\mathrm{x}_0)d\mathrm{x}_{k-1}\\
\Big(\because P(\mathrm{Z}_{0:k-1},\mathrm{U}_{0:k},\mathrm{x}_0)\text{ is a constant for } \mathrm{x}_{k-1}\text{ and it can be fused into }\eta\Big)\\
$$

到了這一步驟後，將上式積分部分與 Measurement-Update 型態做比較便可以得到結果

$$
P(\mathrm{x}_k,\mathrm{m}\mid \mathrm{Z}_{0:k-1},\mathrm{U}_{0:k},\mathrm{x}_0)=\int P(\mathrm{x}_k\mid \mathrm{x}_{k-1},\mathrm{u}_k)\cdot P(\mathrm{x}_{k-1},\mathrm{m}\mid \mathrm{Z}_{0:k-1},\mathrm{U}_{0:k-1},\mathrm{x}_0)d\mathrm{x}_{k-1}
$$


參考資料 Reference
---

1. [StackExchange - How to derive the Time-Update equation of SLAM](https://robotics.stackexchange.com/questions/19152/how-to-derive-the-time-update-equation-of-slam)
2. [Bzarg - How a Kalman filter works, in pictures](https://www.bzarg.com/p/how-a-kalman-filter-works-in-pictures/)
3. [机器人的双眸：视觉SLAM是如何实现的？](https://www.leiphone.com/news/201607/GLQj0wrjKD4eHvq5.html)
4. [FastSLAM算法](https://gaoyichao.com/Xiaotu/?book=probabilistic_robotics&title=pr_chapter13)
5. [Udacity - Artificial Intelligence for Robotics](https://www.udacity.com/course/artificial-intelligence-for-robotics--cs373)
6. [知乎 - 重要性採樣（Importance Sampling）](https://zhuanlan.zhihu.com/p/41217212)
 