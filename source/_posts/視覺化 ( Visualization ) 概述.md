---
title: 視覺化 ( Visualization ) 概述 
date: 2019-10-08 01:28:09
categories:
- 雜記 Essay
image: https://i.imgur.com/poXiirU.png
mathjax: true
---



## Visualization Wheel

我們在探討視覺化前，我們先利用 Alberto Cairo 在 *the functional art* 一書中所提到的 " Visualization Wheel " 來了解針對視覺化評判的標準到底有哪些 ? 也藉此了解到視覺化圖表的要素有什麼。

<!-- more -->

![](https://i.imgur.com/poXiirU.png =450x)


1. Abstraction V.S Figuration ( 抽象—成形 )
     表徵的真實與否，決定了視覺化維度抽象或具象
     
2. Functionality V.S Decoration ( 功能—裝飾 )
     功能完整、無裝飾的圖形更接近數據本身，但有裝飾的圖形更便於記憶與聯想
3. Density V.S Lightness ( 厚實—清新 )
     訊息量的多寡會決定視覺畫圖形的厚實度

4. Multidimensionality V.S Undimensionality ( 多維—一維 )
     維度越高，對於現象描述會越整體，低維度則僅限於關注少數項目

5. Originality V.S Familiarity ( 原創—通用 )
     可根據想表達的目的來決定是否自己創造出新的呈現方式

6. Novelty V.S Redundancy ( 新穎—重複 )
     以單一種方式述說、解釋，或用多種方式來闡述同一個事件

根據資料的背景、功能、目的...，在這個「視覺化轉輪」中會呈現的狀態便相當不同 : 

![](https://i.imgur.com/FpH8NxJ.jpg)

## Data Ink Ratio

除了從上述的轉輪來判分析一個視覺化圖形之外，我們還可以利用 Ediward Tuffe 的 " Data Ink Ratio " 計算出一個視覺化圖形的分數 : 

$Ink-Ratio=\frac{data-ink}{total ink require to print the graphic}$

這裡並非要你真正將墨水消耗量計算出來，重點在建議刪除圖形中不含任何資訊的元素。

我們可以從一個例子來看

![](https://i.imgur.com/GuLrsmZ.png)

這是一張蠻容易見到的 Bar plot ，在 Tuffe 的觀點來看，這樣的圖表並不合格，因為裡面有太多不帶任何資訊的元素存在 : 

* 淺褐色背景
* 多餘的文字說明
* 邊框
* 顏色
* 陰影效果
* 線條

他認為可以將這些不必要元素移除或修改成味更精簡也更讓人能一眼看懂的圖形

![](https://i.imgur.com/x8wCsN2.png)

## Chart Junk

從 ink ratio 我們可以延伸出 Chart Junk 的概念，在 Tuffe 的介紹中， Chart Junk 主要有三大類 : 

**1. 非故意的光學藝術 -- 過度陰影、莫爾圖案**

![](https://i.imgur.com/TN3k7Gl.jpg =450x)


**2. 網格**

**3. 整個圖表示一個裝飾，而失去定量資訊**

![](https://i.imgur.com/vnznHLX.png =450x)

## Sparkline

在一堆文字中加入一些 Sparkline 可以適度地彌補文字、數字與文本之間的差距

![](https://i.imgur.com/7JY5Jg8.png =550x)

## Lie Factor

所謂Lie Factor 指的是，因為圖形顯示上的誤差而造成閱讀者的誤解

![](https://i.imgur.com/gVsVjsq.png)

就上圖來說，從圖片上來看，從1978年 18 miles per gallon 到1985年 27.5 miles per gallon，增長率約 53%，但圖形的線段長度由0.6 inches 增加成 5.3 inches ，足足增長的 783%，這便可能讓閱讀者產生混淆。

而這邊所指的 $Lie Factor=\frac{Graphical increase}{Data increase}=\frac{783%}{53%}=14.8$

## 總結

從上面這些會影響視覺化圖形的因素來看，我們可以分析出一個好的視覺化圖形應該有那些特點 : 

1. Truthful (真實)
2. Functionality (功能性強)
3. Beauty (美觀)
4. Insightful (洞察力)
5. Enlightening (啟發性)



---

有一些文章或者比較現代一點的資料科學家，會認為 Tuffe 的觀點有一些老套，的確，在上面某一些圖形也顯示出或許有些觀念似乎不是這麼適合現代的視覺化系統，但是，由 Tuffe 所提出的各項概念，仍有不少是值得我們在視覺化過程中注意的。

不管我們接受什麼資訊或方法，永遠要記住一點，這些都是我們的工具，並非絕對是用於各種狀況，我們都必須要了解每一個想法的優缺點，才能在未來的使用上更能得心應手，也更能貼近資料所要表達的內涵。