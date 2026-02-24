---
title: 影像畸變 Image Distortion
date: 2020-02-15 08:43:40
categories:
-  影像處理 Image processing
image: https://i.imgur.com/IF1bVRK.png
mathjax: true
---


在前面的文章 "[針孔相機模型 Pinhole Camera Model](https://allen108108.github.io/blog/2020/02/06/%E9%87%9D%E5%AD%94%E7%9B%B8%E6%A9%9F%E6%A8%A1%E5%9E%8B%20%20Pinhole%20Camera%20Model/#more)" 中我們有提到在一個理想的針孔相機中，一個 3D 目標怎麼投影到 2D 平面上。

但是，實際上一個光學系統必須經由多組鏡頭所組成，在這樣的系統下，單純理想的針孔相機模型便無法擬合真實情況。加入鏡頭、透鏡後，成像必然會有某些程度上的失真、變形，這樣的情況我們稱之為「畸變」(Distortion)。

<!-- more -->

以下，我們將畸變區分成三種形式 ： 輻射畸變 (Redial Distortion)、離心畸變 ( Decentering Distortion ) 以及薄透鏡畸變 ( Thin Prism Distortion )。




輻射畸變 Radial Distortion
---

當光線進入透鏡時，由邊緣射入的光線會比從中心射入的光線有著更大的偏折，這便是造成輻射畸變的主要原因，而這樣的現象在透鏡越小時，會越加明顯。

輻射畸變又分為兩種 ： 桶狀畸變 Barrel Distortion & 枕狀畸變 Pincushion Distortion

![](https://i.imgur.com/IF1bVRK.png)

在攝影的領域來看，通常使用廣角鏡頭容易出現桶狀畸變，而使用長焦段鏡頭則易產生枕狀畸變。這兩者其實就是一體兩面，如果我們拿廣角鏡頭來作為投放鏡頭，便會產生枕狀畸變，反之亦然。

因此，這樣的畸變我們可以利用相同的參數來進行控制 $k_1, k_2,k_3,\cdots$，在一班的狀況下，利用 $k_1,k_2$ 已經可以量化絕大多數的畸變狀況，除非程度真的很嚴重才會利用更多的參數來表徵。



離心畸變 Decentering Distortion
---

前面說過，一個光學系統由多種鏡頭組合而成，在現實狀況中，這些鏡頭的在現實狀況中，這些鏡頭的光軸不見得都能完全重合在同一條中心線上，因為這樣的原因而造成成像的畸變，我們稱之為 「離心畸變」。

離心畸變係數 (Decentering Distortion Coeddicient) 我們通常以 $p_1,p_2$ 來表示。

值得注意的是，在很多的技術文章中會把離心畸變的部分直接以正切畸變 (Tangential Distortion) 來表示，包含 Open CV document 亦是如此。

但在這篇文章中我不選用這樣的名稱主要是因為，正切畸變應該是一個畸變的「組成要素」，如同任一個方向向量都可以由切線方向跟法線方向兩種向量組成，而輻射畸變與正切畸變也應該是這樣的要素。

在論文 " *Camera Calibration with Distortion Models and Accuracy Evaluation.* " 中也提到，離心畸變應該是包含輻射畸變與正切畸變的。


薄透鏡畸變 Thin Prism Distortion
---

當相機的 CCD 與鏡頭透鏡沒有平行，也會造成成像的變形。

![](https://i.imgur.com/toxBP6v.png)

薄透鏡畸變主要發生在鏡頭設計與零組件組合的瑕疵上，如同離心畸變，薄透鏡畸變也同時存在輻射與正切兩種畸變量


我們利用薄透鏡畸變係數 (Thin Prism Distortion Coefficient ) $s_1\sim s_4$ 來計算出畸變的狀況。


畸變相機模型 Camera Model with Distortion
---

從許多研究中，發展出一套完整的相機模型，可以將畸變的因素計算在其中，假定某點其世界
座標為 $(X_c,Y_c,Z_c)$ ，則未考量畸變因素時我們可以計算出其成像座標 $(x',y')$ 為 ：

$$
x' = \displaystyle{\frac{X_c}{Z_c}}\text{ and }y'= \displaystyle{\frac{Y_c}{Z_c}}
$$

若考量畸變因素進去，則必須再一次修正成像座標為 $(x'',y'')$ : 

$$
x'' = x'(k_1r^2+k_2r^4+k_3r^6+\cdots)+p_1(2x'^2+r^2)+2p_2x'y'+s_1r^2+s_3r^4\\
y'' = y'(k_1r^2+k_2r^4+k_3r^6+\cdots)+2p_1x'y'+p_2(2y'^2+r^2)+s_2r^2+s_4r^4\\
\text{where } r^2=x'^2+y'^2
$$

若，我們忽略 $k_3,k_4,\cdots,s_3,s_4$ 所造成的影響，上式可以整理成：

$$
x'' = x'(k_1r^2+k_2r^4)+p_1(2x'^2+r^2)+2p_2x'y'+s_1r^2\\
y'' = y'(k_1r^2+k_2r^4)+p_2(2y'^2+r^2)+2p_1x'y'+s_2r^2\\
$$

最後再轉換成以像素為單位的座標

$$
u=f_xx''+c_x\\
v=f_yy''+c_y\\
$$

以上便是一個通用的畸變相機模型。



Open CV 中的畸變相機模型
---

在 Open CV [document](https://docs.opencv.org/master/d9/d0c/group__calib3d.html) 中有將整個畸變參數及其方程式列出 ：

差異的部份僅在於 $k_1\sim k_6$ 的部分

$$
x''=\displaystyle{\frac{1+k_1r^2+k_2r^4+k_3r^6}{1+k_4r^2+k_5r^4+k_6r^6}}x'+2p_1x'y'+p_2(r^2+2x'^2)+s_1r^2+s_2r^4\\
y''=\displaystyle{\frac{1+k_1r^2+k_2r^4+k_3r^6}{1+k_4r^2+k_5r^4+k_6r^6}}y'+p_1(r^2+2y'^2)+2p_2x'y'+s_3r^2+s_4r^4\\
\text{where }r^2=x'^2+y'^2
$$

但在一般的情況下，即不考慮 $k_3\sim k_6,s_3,s_4$ 的影響，Open CV 所提供的相機模型與一般通用的畸變相機模型其實是等價的。

畸變視覺化 Distortion Visualization
---

這麼多不同的畸變參數，造成的成因也都不一樣．光是看上面敘述可能會很難理解，究竟圖像「畸變」過後會變成什麼樣的圖像？

因為我們已經知道了整個畸變模型，引此我們就可以直接定義出畸變的函式如下：

```python=
def distortion(x,y,k1=0,k2=0,p1=0,p2=0,s1=0,s2=0):
    r_square = x**2 + y**2
    k = 1 + k1*r_square + k2*(r_square**2)
    u = x*k + 2*p2*x*y + p1*(r_square + 2*(x**2)) + s1*r_square
    v = y*k + p2*(r_square + 2*(y**2)) + 2*p1*x*y + s2*r_square
    return u, v
```

經過不同參數調整後的變化我們呈現如下

![](https://i.imgur.com/cW2BdrY.png)
![](https://i.imgur.com/HZYU5iW.png)
![](https://i.imgur.com/R1H5TPQ.png)
![](https://i.imgur.com/Xsy9pXO.png)
![](https://i.imgur.com/oup3eMh.png)


藉由畸變相機模型我們可以掌握整個成像的狀況，更重要的是，我們可以藉由這樣的方程式進行逆轉，也就是進行圖像的校準 ( calibration )，進而取得我們希望的圖像平面座標。

結論 Conclusion
---

" *[針孔相機模型 Pinhole Camera Model](https://)* " 與本文主要簡介了整個從 3D 立體空間投影到 2D 平面圖像空間的整個過程 ：

![](https://i.imgur.com/Stdq5dx.png)

當然，這都還是以比較單純的狀況下來討論，畢竟畸變只是多種像差的其中一種 ( 初級像差分為五種 ： 球面像差、彗型像差、像散、場曲及畸變 ) 。但從這兩篇文章的討論中，也大致上描繪出整個相機校準及3D重建的整體樣貌。


參考資料
---

1. [Reconstruction: The Mighty Camera](https://medium.com/@DroneDeployEng/reconstruction-the-mighty-camera-624619bf8cf6)
2. [Camera Calibration and 3D Reconstruction](https://docs.opencv.org/master/d9/d0c/group__calib3d.html)
3. Juyang Weng, Paul Cohen, and Marc Herniou. 1992. Camera Calibration with Distortion Models and Accuracy Evaluation. IEEE Trans. Pattern Anal. Mach. Intell. 14, 10 (October 1992), 965–980.
4. Jianhua Wang, Fanhuai Shi, Jing Zhang, and Yuncai Liu. 2008. A new calibration model of camera lens distortion. Pattern Recogn. 41, 2 (February 2008), 607–615.

系列文章
---

* [針孔相機模型 Pinhole Camera Model](https://allen108108.github.io/blog/2020/02/06/%E9%87%9D%E5%AD%94%E7%9B%B8%E6%A9%9F%E6%A8%A1%E5%9E%8B%20%20Pinhole%20Camera%20Model/)
* [影像畸變 Image Distortion](https://allen108108.github.io/blog/2020/02/15/%E5%BD%B1%E5%83%8F%E7%95%B8%E8%AE%8A%20Image%20Distortion/)