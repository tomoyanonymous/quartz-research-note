#memo #mimium #programming-language 

[[音楽プログラミング言語の形式化#mimium と 多段階計算]]

[[多段階計算]]を取り入れたい

## 型定義

a個の実数のタプルである$R_a$

n以下の自然数$I_n$ (ディレイのbounded access用)

$$
\begin{align}
\tau ::=& R_a \quad & a \in \mathbb{N}\\
      |& R_a → R_b \quad &a,b \in \mathbb{N}\\
      |& I_n \quad &n \in \mathbb{N}
\end{align}
$$
とりあえず1要素のタプルと普通のRは区別しないことにする

$$
\begin{align}
e,f ::=& \quad x \quad x \in \mathbb{V}\\
     |& \quad \lambda x.e\\
     |& \quad f \; e
\end{align}
$$