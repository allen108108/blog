---
title: 針孔相機模型  Pinhole Camera Model
date: 2020-02-06 18:35:16
categories:
- 影像處理 Image processing
image: https://i.imgur.com/RwYTful.png
mathjax: true
---


光學基本平面 Cardinal Plane
---




在搞清楚 Pinhole Camera Model 以前，或許我們要先理解一些專有名詞 ： 
* 光學基本平面 (Cardinal Plane)
* 基點（cardinal points）
* 主平面（principle plane）
* 主點（principle point）
* 焦平面（focal plane）
* 焦點（focal point）

<!-- more -->

![](https://i.imgur.com/Hn4cVyP.png)

光線由物體發出，經過鏡頭成像，可區分空間為物空間 (object space) 及像空間 (image space)，物空間指的是光線在經過鏡頭前的空間，而像空間則是光線經過鏡頭過後的空間稱之。

從物空間的角度來看，入射光與出射光之延伸會交會於一平面，我們稱之為第一主平面 (The First Principle Plane)，相反的，從像空間的角度來看，入射光與出射光之延伸也會教會於一平面，我們稱之為第二主平面。此兩平面與光軸的交會點則稱之為第一、二主點 ($H_1,\ H_2$ The Principle Point)

在透鏡厚度極薄的情況下，兩個主平面會幾乎重合，而兩主點會重合於透鏡中心。


針孔相機模型 Pinhole Camera Model
---

在 Computer Vision 的領域中，Pinhole Camera Model 的用途時十分廣泛，不管是要做相機的校準或者是 3D 重建都會需要用到 Pinhole Camera Model 的基礎進行。

所謂的 Pinhole Camera Model 描繪的其實就是一個理想的針孔相機 （ 針孔孔徑無限小 ）如何將 3D 世界中的物體投影到一個 2D 平面上，其數學表達式可以下列呈現 ：

$$
m=K\begin{bmatrix}R|T\end{bmatrix}M\\
\Longrightarrow \begin{bmatrix}u\\v\\1\end{bmatrix}=\begin{bmatrix}f_x & s &c_x\\0 & f_y & c_y\\0&0&1 \end{bmatrix}\begin{bmatrix}r_{11}&r_{12}&r_{13}&t_x\\r_{21}&r_{22}&r_{23}&t_y\\r_{31}&r_{32}&r_{33}&t_z\end{bmatrix}\begin{bmatrix}X_w\\Y_w\\Z_w\\1\end{bmatrix}
$$

其中

$(X_w,Y_w,Z_w):$ 世界座標，即一絕對座標系
$(X_c,Y_c,Z_c):$ 相機座標，以相機為中心的相對坐標系
$s:$ 投射平面若兩軸非垂直的調整參數
$(u,v):$ 投射平面上的座標 ( 單位為像素 )
$(c_x,c_y):$ 主點 Principle Point
$\begin{bmatrix}R|T\end{bmatrix}:$ 將世界座標轉換為相機座標之轉換矩陣，其實就是一個旋轉平移的線性變換





<img width=500 src="https://i.imgur.com/RwYTful.png" >


### 數學推導 Derivative of Formula

**Step 1.**

假設物體的世界座標為 $(X_w,Y_w,Z_w)$，那我們必須先做一個線性變換，轉換成以相機為主的相機座標。

$$
\begin{bmatrix}X_c\\Y_c\\Z_c\end{bmatrix}=R\begin{bmatrix}X_w\\Y_w\\Z_w\end{bmatrix}+T=\begin{bmatrix}r_{11}&r_{12}&r_{13}\\r_{21}&r_{22}&r_{23}\\r_{31}&r_{32}&r_{33}\end{bmatrix}\begin{bmatrix}X_w\\Y_w\\Z_w\end{bmatrix}+\begin{bmatrix}t_x\\t_y\\t_z\end{bmatrix}\\\Longrightarrow\begin{bmatrix}X_c\\Y_c\\Z_c\end{bmatrix}=\begin{bmatrix}r_{11}&r_{12}&r_{13}&t_x\\r_{21}&r_{22}&r_{23}&t_y\\r_{31}&r_{32}&r_{33}&t_z\end{bmatrix}\begin{bmatrix}X_w\\Y_w\\Z_w\\1\end{bmatrix}
$$


**Step 2.**

接下來我們要計算物件投影到平面的座標，根據三角關係，我們可以知道 ： 

$$
u_c=\frac{f}{Z_c}\cdot X_c\\
v_c=\frac{f}{Z_c}\cdot Y_c\\
$$

我們當然可以改寫成以下形式 ：

$$
\begin{bmatrix}u_c\\v_c\end{bmatrix}=\begin{bmatrix}{\frac{f}{Z_c}}&0\\0&{\frac{f}{Z_c}}\end{bmatrix}\begin{bmatrix}X_c\\Y_c\end{bmatrix}
$$

但在圖像的座標中，圓點往往不是中心點，而是圖像的左下角，因此，以圖像中心為原點的$(u_c,v_c)$ 座標還需要做修正

$$
\begin{bmatrix}u\\v\end{bmatrix}=\begin{bmatrix}{\frac{f}{Z_c}}&0\\0&{\frac{f}{Z_c}}\end{bmatrix}\begin{bmatrix}X_c\\Y_c\end{bmatrix}+\begin{bmatrix}c_x\\c_y\end{bmatrix}
$$

接下來要做的是單位的轉換，目前 $(u,v)$ 是以長度為單位 (cm, inch,...)，然而在圖像上通常是以像素 (pixel) 為計算單位，所以必須要再做一個單位上的轉換。

每一個像素長寬比並非都是 $1:1$ 因此在長與寬上，每長度單位上的像素數量也可能會不同，因此我們引進 $k$, $l$ 兩個參數進來，其單位為 ( $\frac{pixel}{cm}$ )，意義即為長與寬中每長度單位的像素數量。

我們便可以修改上式：

$$
\begin{bmatrix}u\\v\end{bmatrix}=\begin{bmatrix}{\frac{f}{Z_c}\cdot k}&0\\0&{\frac{f}{Z_c}\cdot l}\end{bmatrix}\begin{bmatrix}X_c\\Y_c\end{bmatrix}+\begin{bmatrix}c_x\\c_y\end{bmatrix}\\
\Longrightarrow \begin{bmatrix}u\\v\end{bmatrix}=\begin{bmatrix}{\frac{f_x}{Z_c}}&0\\0&{\frac{f_y}{Z_c}}\end{bmatrix}\begin{bmatrix}X_c\\Y_c\end{bmatrix}+\begin{bmatrix}c_x\\c_y\end{bmatrix}=\begin{bmatrix}{\frac{f_x}{Z_c}}X_c+c_x\\{\frac{f_y}{Z_c}}Y_c+c_y\end{bmatrix}\\
\text{where }(f_x,f_y)\text{ are the focal lenths express in pixel units.}
$$

**Step 3.**

然而，有沒有一種方式可以將這種非線性變換用類似線性變換的矩陣乘法來做更好的表述 ？在電腦視覺領域中，最常用的方式就是齊次座標系 (Homogeneous coordinate system) 的轉換 : 

$$
Euclidean\ Coordinate\sim Homogeneous\ Coordinate\\
(\frac{a}{d},\frac{b}{d},\frac{c}{d})\sim (a,b,c,d)
$$

所以我們可以齊次坐標系來重新表述 ：

$$
\begin{bmatrix}u\\v\\1\end{bmatrix}=\begin{bmatrix}f_x&0&c_x\\0&f_y&c_y\\0&0&1\end{bmatrix}\begin{bmatrix}X_c\\Y_c\\Z_c\end{bmatrix}\\
\text{where }(u,v)=(\frac{f_x}{Z_c}X_c,\frac{f_y}{Z_c}Y_c)\sim (u,v,1)=(\frac{f_x}{Z_c}X_c,\frac{f_y}{Z_c}Y_c, 1)\sim (f_xX_c,f_yY_c,Z_c)\\
\Longrightarrow \begin{bmatrix}u\\v\\1\end{bmatrix}=\begin{bmatrix}f_x&0&c_x\\0&f_y&c_y\\0&0&1\end{bmatrix}\begin{bmatrix}X_c\\Y_c\\Z_c\end{bmatrix}\\
=\begin{bmatrix}f_x&0&c_x\\0&f_y&c_y\\0&0&1\end{bmatrix}\begin{bmatrix}r_{11}&r_{12}&r_{13}&t_x\\r_{21}&r_{22}&r_{23}&t_y\\r_{31}&r_{32}&r_{33}&t_z\end{bmatrix}\begin{bmatrix}X_w\\Y_w\\Z_w\\1\end{bmatrix}
$$


**Step 4.**

此外，以上都是假設當相機 CCD 兩軸為垂直時，投影的圖像不會有變形的狀況，某些特別的相機成像中，兩軸本身並不一定會是完全方正的，可能存在一些微小的變形，這個時候就需要 $s$ (skew parameter) 這個參數來做調整



$$
K=\begin{bmatrix}f_x&s&c_x\\0&f_y&c_y\\0&0&1\end{bmatrix}\\
\text{where } s=f_y\tan\theta
$$

這個 $K$ 矩陣完全由相機本身的參數所構成，我們會稱這樣的矩陣為「內部參數矩陣」（
Intrinsic Parameter Matrix）。

總結以上便可以推得 

$$
m=K\begin{bmatrix}R|T\end{bmatrix}M
$$

APPENDIX
---

### A. 齊次座標系統 Homogeneous Coordinate System

![](https://i.imgur.com/KqZ4UQy.jpg)

鐵軌的兩側是平行的，在我們常用的笛卡爾座標系 ( Cartesian Coordinate System )中，平行的兩條線不會有交點，然而在上面的圖像中，鐵軌的兩側交會了。再舉一個例子，一個長方體的面紙盒，如果我們拿相機從某一個角度進行拍攝，你也會發現，整個面紙盒不再以矩形的方式呈現在照片中。

在數學上，要處理這樣的問題，便要使用到「投影幾何」( Projactive Geometry )，來處理相關的問題。

在投影幾何中，相對應要處理的變換 (transformation) 遠比一般歐氏幾何來得多樣且廣泛，不只有旋轉平移跟縮放，而數學家要處理這樣的變換，使用到的方法便是齊次座標系 ( Homogeneous Coordinate System ) 的轉換。

轉換方式很簡單

$$
\text{Catesain Coordinate System :}(x,y)\longrightarrow \text{Homogeneous Coordinate System :}(x,y,1)\\
\text{Homogeneous Coordinate System :}(u,v,w)\longrightarrow \text{Catesain Coordinate System :}(\frac{u}{w},\frac{v}{w})
$$

在這樣的轉換之下，我們便可以說，兩條平行線交會於某一點 $(x,y,0)$

此外，從上面的轉換來看，我們可以發現，一個 Catesain Coordinate 可以用無限個 Homogeneous Coodinate 來表示 ： 

$$
(1,2)\sim (1,2,1)\sim (2,4,2)\sim(w\cdot1,w\cdot 2,w)\\
\text{where }w\neq 0
$$


<img width=500 src="https://i.imgur.com/LCvcD1Z.png" >



### B. 偏斜係數 Skew Parameter

![](https://i.imgur.com/muXKisw.png)

在現今的相機硬體技術上，幾乎已經克服了這樣的問題，從一些書籍或是文獻上其實都已經可以將其忽略不看，從 OpenCV 的[文件](https://docs.opencv.org/4.0.1/d9/d0c/group__calib3d.html#ga3207604e4b1a1758aa66acb6ed5aa65d)中也可以發現他們已經將其省略，但我們這邊還是稍微討論一下。

首先，不管是哪一種變換，我們可以知道 $x$ 的單位長度不變，僅平移。而兩者差異在於 $y$ 軸單位長度是否會發生變化。( Case1： $y$ 軸單位長會變長 )
 
一般來說在討論 "skew" 這樣的轉換，我們會比較偏向於 Case(2) 的情況，但在現實情況中兩者均可能會發生。

**Case(1)**

如果是這樣的狀況，稱作 "Shear Transformation"，其高度不變，但取而代之的是 Y 軸變長（斜邊較長），因此在座標的轉換上就僅需考量 X 軸方向的偏移即可。


<img width=500 src="https://i.imgur.com/XzsNHeT.png" >





原本 $P$ 點應該是 $(\hat{x},\hat{y})$，經過變換後應該變成 $(\hat{x}-\hat{y}\cot\theta,\displaystyle{\frac{\hat{y}}{\sin\theta}})$，所以 ：


$$
K=\begin{bmatrix}f_x&-f_x\cot\theta&c_x\\0&f_y&c_y\\0&0&1\end{bmatrix}
$$

要注意的地方是，$f_y$ 其實也會跟著調整，但調整過後再轉換便會抵銷掉，我們寫清楚的話應該是 ：

$$
f_y^{shear}=f_y^{original}\cdot \sin\theta\\
$$
調整完再經過變換後
$$
f_y^{shear}\cdot\frac{1}{\sin\theta}=f_y^{original}
$$

**Case(2)**

$P$ 點經過變換後一樣變成 $(\hat{x}-\hat{y}\cot\theta,\displaystyle{\frac{\hat{y}}{\sin\theta}})$，但差別在於 $f_y$ 不需要再做調整，所以 ：

$$
K=\begin{bmatrix}f_x&-f_x\cot\theta&c_x\\0&\displaystyle{\frac{f_y}{\sin\theta}}&c_y\\0&0&1\end{bmatrix}
$$



參考資料
---

1. [The Concept Of Homogeneous Coordinates](https://prateekvjoshi.com/2014/06/13/the-concept-of-homogeneous-coordinates/)
2. [Homogeneous Coordinates](http://www.songho.ca/math/homogeneous/homogeneous.html)
3. [Camera intrinsics: Axis skew](https://blog.immenselyhappy.com/post/camera-axis-skew/#pixel-coordinates-with-the-shear-transform)



系列文章
---

* [針孔相機模型 Pinhole Camera Model](https://allen108108.github.io/blog/2020/02/06/%E9%87%9D%E5%AD%94%E7%9B%B8%E6%A9%9F%E6%A8%A1%E5%9E%8B%20%20Pinhole%20Camera%20Model/)
* [影像畸變 Image Distortion](https://allen108108.github.io/blog/2020/02/15/%E5%BD%B1%E5%83%8F%E7%95%B8%E8%AE%8A%20Image%20Distortion/)