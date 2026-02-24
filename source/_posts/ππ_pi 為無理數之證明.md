---
title: $\pi$ 為無理數之證明
date: 2020-05-01 06:02:19
categories:
- 數學筆記 Mathematics Note
image: https://i.imgur.com/oP5SEfg.jpg
mathjax: true
---
 

假設 $\pi$ 為有理數，即， $\pi=\dfrac{p}{q}\text{,}$ and $p,q\in\mathbb{N}$。 令 : 

<!-- more -->

$$
f_n(x)=\dfrac{1}{n!}q^nx^n(\pi-x)^n=\dfrac{1}{n!}x^n(p-qx)^n
$$

則 $f_n(x)$ 為一個 $2n$ 次多項式，且 $x^n(p-qx)^n$ 展開後係數為整數。

(1)

$\because f_n'(x)=\dfrac{1}{n!}q^n\cdot n\cdot (\pi x-x^2)^{n-1}\cdot (\pi-2x)$

$\therefore f_n(\dfrac{\pi}{2})=M_n$ 是 $f_n$ 在 $[0,\pi]$ 區間中的最大值

$\Longrightarrow M_n=f_n(\dfrac{\pi}{2})=\dfrac{1}{n!}q^n\cdot(\dfrac{\pi}{2})^n$

$\Longrightarrow \displaystyle\lim_{n \to \infty}M_n=0$ $\big(\because \displaystyle\lim_{n \to \infty}\dfrac{a^n}{n!}=0\big)$

令

$$
I_n=\int_0^\pi f_n(x)\sin xdx
$$

$\because f_n(x)>0\text{ , }\sin x>0\text{ , }0<x<\pi$

$\therefore\displaystyle\int_0^\pi f_n(x)\sin xdx=I_n>0$

$\Longrightarrow I_n=\displaystyle\int_0^\pi f_n(x)\sin xdx<\int_0^\pi M_n\sin xdx=2M_n$

$\because \displaystyle\lim_{n\to\infty}M_n=0$

$\Longrightarrow \exists\ n>0$ 滿足 $M_n<\dfrac{1}{2}$

$\Longrightarrow I_n<2M_n<1$


(2)

$f_n(x)=\dfrac{1}{n!}x^n(p-qx)^n\overset{let}{=}\dfrac{1}{n!}\displaystyle\sum_{k=n}^{2n}\alpha_kx^k$，$\alpha_k\in\mathbb{Z}$

$\therefore f_n(x)=\alpha_nx^n+\alpha_{n-1}x^{n-1}+\cdots+\alpha_{2n}x^{2n}$

$\Longrightarrow\begin{cases}f_n^{(m)}(0)=0&\in\mathbb{Z}&\mbox{, }m < n\\
f_n^{(m)}(0)=\alpha_n n!&\in\mathbb{Z}&\mbox{, }m = n \\
f_n^{(m)}(0)=\alpha_m m!&\in\mathbb{Z}&\mbox{, }m>n\end{cases}\Longrightarrow f_n^{(m)}\in\mathbb{Z}\text{ , }\forall m>0$

且

$f_n(x)=f_n(\pi-x)\Rightarrow f_n^{(m)}(x)=f_n^{(m)}(\pi-x)\cdot(-1)^m$

$\Longrightarrow f_n^{(m)}(\pi)=f_n^{(m)}(0)\cdot(-1)^m\in\mathbb{Z}$

$\Longrightarrow f_n^{(m)}(0), f_n^{(m)}(\pi)\in\mathbb{Z}$


(3)

令

$F_n(x)\ \ \ =f_n(x)-f_n^{(2)}(x)+f_n^{(4)}(x)-f_n^{(6)}(x)+\cdots+(-1)^nf_n^{(2n)}(x)$

$F_n^{(2)}(x)=\ \ \ \ \ \ \ \ \ \ +f_n^{(2)}(x)-f_n^{(4)}(x)+f_n^{(6)}(x)+\cdots+(-1)^{n-1}f_n^{(2n)}(x)+(-1)^{n}f_n^{(2n+2)}(x)$

$\therefore F_n(x)+F_n''(x)=f_n(x)+(-1)^{n}f_n^{(2n+2)}(x)=f_n(x)$

$\big(\because f_n(x)=\alpha_nx^n+\alpha_{n-1}x^{n-1}+\cdots+\alpha_{2n}x^{2n}\Longrightarrow f_n^{(2n+2)}(x)=0\big)$

$\big[F_n'(x)\sin x-F_n(x)\cos x\big]'=\Big(F_n''(x)\sin x+F_n'(x)\cos x\Big)-\Big(F_n'(x)\cos x+F_n(x)(-\sin x)\Big)$

$=F_n''(x)\sin x+F_n'(x)\cos x-F_n'(x)\cos x+F_n(x)\sin x$

$=F_n''(x)\sin x+F_n(x)\sin x=f_n(x)\sin x$

(4)

由微積分基本定理 (Fundamental Theorem of Calculus) 可知 : 

$I_n=\int_0^{\pi}f_n(x)\sin xdx=\Big(F_n'(x)\sin x-
f_n(x)\cos x\Big)_0^{\pi}=F_n(0)+F_n(\pi)$

$\overset{by\text{ (3)}}{\Longrightarrow} F_n(0)=f_n(0)-f_n^{(2)}(0)+f_n^{(4)}(0)-f_n^{(6)}(0)+\cdots+(-1)^nf_n^{(2n)}(0)\in\mathbb{Z}$

$\overset{by\text{ (2)}}{\Longrightarrow} F_n(\pi)\in\mathbb{Z}$

$\overset{by\text{ (1)}}{\Longrightarrow} I_n=F_n(0)+F_n(\pi)\in\mathbb{Z}$ ($\rightarrow\leftarrow$)

$\therefore \pi \text{ is irrational.}$
