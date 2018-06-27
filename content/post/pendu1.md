---
title: 二重振り子のシミュレーション　第一回
slug: 二重振り子１
date: '2018-06-27'
tags:
    - "力学"
    - "二重振り子"
categories:
    - "作ってみた"
image: "/img/3DP/1.png"
description: "ゴリゴリの力学です"
author: H﨑
draft: true
comments: true
math: true
---

## 二重振り子とは？？
<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">物理部らしいことしてみた<br><br>二重振り子はカオス現象ですが、運動方程式をラグランジアンに変換して、Runge–Kutta法で微分方程式の近似解出したらシミュレーションできます(言ってみたかった) <a href="https://t.co/fuN8Pdsiui">pic.twitter.com/fuN8Pdsiui</a></p>&mdash; 甲陽学院高等学校 物理部 (@koyobutsuri) <a href="https://twitter.com/koyobutsuri/status/1011218367280107520?ref_src=twsrc%5Etfw">2018年6月25日</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

こんなやつです。
普通の振り子の先にもう一つ振り子をつないだだけの簡単な構造なのですが、これがまた興味深い。いわゆるカオスな運動をします。

> カオス理論は、力学系の一部に見られる、数的誤差により予測できないとされている複雑な様子を示す現象を扱う理論である。ここで言う予測できないとは、決してランダムということではない。その振る舞いは決定論的法則に従うものの、初期値鋭敏性ゆえに、数値解析の過程での誤差によっても、得られる値と真の値とのずれが増幅される。そのため予測が事実上不可能という意味である。(Wikipediaより引用)

つまり、法則はあるけれど、どうしても誤差が出てしまうということです。

### 誤差あるなら、シミュレーションの意味なくね？

それ言ったら、全ておしまいなので、先進みます。

## ひたすら計算
<img src="/img/pendu/1.jpg" alt="Drawing" style="width: 50%"/>

二つのおもりで構成される二重振り子について考えてみたいと思います。  
おもり1は質量$m_1$で紐の長さは$l_1$で角度を$\theta_1$、  
おもり2は質量$m_2$で紐の長さは$l_2$で角度を$\theta_2$とします。  
そうすると



$$\frac{\partial^2 p}{\partial t^2} = c^2 \nabla^2 p$$





次回のエントリでは $T = S((1 + \frac{E}{R})^{F} - 1)$ について書こうと思います。

\begin{align}
\ddot{x}_1&#038;=r_1\sin{\theta_1}\\\\\\
y_1&#038;=-r_1\cos{\theta_1} \\\\\\
x_2&#038;=r_1\sin{\theta_1}+r_2\sin{\theta_2}\\\\\\
y_2&#038;=-r_1\cos{\theta_1}-r_2\cos{\theta_2}\\\\\\
\end{align}
