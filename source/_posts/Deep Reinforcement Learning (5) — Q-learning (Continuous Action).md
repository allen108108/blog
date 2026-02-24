---
title: "Deep Reinforcement Learning (5) --- Q-learning (Continuous Action)" 
date: 2019-12-05 12:17:09
categories:
- 課程筆記 Course
- 李宏毅 Machine Learning and having it Deep and Structured
image: https://i.imgur.com/MdVC5JZ.png
mathjax: true
---

Continuous Action
---

前面的部分大概都是在 discrete action 的前提下作討論，但在許多現實狀況下，action 可能是連續，例如 : 機器人關節的彎曲程度。

<!-- more -->

Continuous Action 的情況之所以困難，最主要的原因還是在於難以找出下式的最佳解

$$
a=\arg\max\limits_{a}Q(s,a)
$$

以下提供幾個可能的處理方式供大家參考 : 

### Solution 1. : MC-based

第一種解法其實跟蒙地卡羅法的概念很接近，Action 是一個我們很難估測的函數，那我就來利用抽樣資料來去估測 Aciton。

我們直接抽樣出 $N$ 筆資料，再將其帶入上式看看哪一個可以獲得最大的 Q 值即可。這樣的方式非常的快速，但是卻不能確保其準確度。


### Solution 2. : Gradient Ascent

將 $a$ 視為參數，將上面的優化問題轉化成要找出一組參數使上式最大化，這樣就可以利用 Gradient-Based 的方式來進行求解。

但這種方式必須要進行迭代更新，計算成本相對高，而且也有可能會卡在 local maximum 的狀況。

### Solution 3. : Neural Network

第三個方法感覺很天馬行空，概念其實就是特別設計一個 DQN，讓它輸出的時候就可以把答案找出來這樣。

![](https://i.imgur.com/C9XQBg8.png)

上圖就是這個特別的 DQN，其輸出為一個向量 $\mu$、一個矩陣 $\Sigma$ 以及一個純量 $V$，再利用這三者組合成我們需要的 $Q(s,a)$

$$
Q(s,a)=-(a-\mu(s))^T\Sigma(s)(a-\mu(s))+V(s)\\
\Longrightarrow \mu(s)=\arg\max\limits_{a}Q(s,a)
$$

### Solution 4. : A3C

留待下一篇文章再來介紹。