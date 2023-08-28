#memo #mimium #programming-language 

[[音楽プログラミング言語の形式化#mimium と 多段階計算]]

[[多段階計算]]を取り入れたい

とりあえず$W$ Calculusを自然に多段階に拡張してみる
## 型定義



a個の実数のタプルである$R_a$

n以下の自然数$I_n$ (ディレイのbounded access用)

$$
\begin{align}
\tau ::=&\quad R_a \quad & a \in \mathbb{N}\\
      |&\quad R_a → R_b \quad &a,b \in \mathbb{N}\\
      |&\quad I_n \quad &n \in \mathbb{N} \\
      |&\quad \langle \tau \rangle
\end{align}
$$
とりあえず1要素のタプルと普通のRは区別しないことにする
(そしてよく見るとこれは関数→関数のような高階関数を許してないんだな)
そうか高階関数を考えなければクロージャを考慮する必要もないものな

## シンタックス

$$
\begin{align}
e,f \; ::=& \quad x \quad x \in \mathbb{V} \quad & [value]\\
     |& \quad \lambda x.e  \quad & [lambda]\\
     |& \quad fix \; x.e  \quad & [fix]\\
     |& \quad feed \; x.e \quad & [feed] \\
     |& \quad f \; e \quad & [app]\\
     |& \quad (e_1,e_2) \quad & [product]\\
     |& \quad \pi_n e \quad n\in \mathbb{N},\; n>0 \quad & [project]\\
     |& \quad \langle e \rangle \quad & [code] \\
     |& \quad \textasciitilde e \quad & [escape]
\end{align}
$$

基本演算（Intrinsic）は直感に任せる

本来はfixの中でfeedを使ったり、feedの中でfixを使うとエラーだが、結局シンタックスレベルでは排除できないので型でエラーとして弾くことにする…？
いや値レベルでの切り分けは不可能なので、こうする

$$
\begin{align}
e,f \; ::=& \quad x \quad x \in \mathbb{V} \quad & [value]\\
     |& \quad \lambda x.e  \quad & [lambda]\\
     |& \quad f \; e \quad & [app]\\
     |& \quad (e_1,e_2) \quad & [product]\\
     |& \quad \pi_n e \quad n\in \mathbb{N},\; n>0 \quad & [project]\\
     |& \quad \langle e \rangle \quad & [code] \\
     |& \quad \textasciitilde e \quad & [escape] \\
e_{fix} \; ::=& \quad e\\
		|&  \quad fix \; x.e_{fix}  \quad & [fix]\\
e_{feed} \; ::=& \quad e\\
		|&  \quad feed \; x.e_{feed}  \quad & [feed]\\
\end{align}
$$

こんな雰囲気で相互再帰させれば、、、いいんか？ダメだな

ステージの概念さえ導入できれば値レベルで弾けるのかな

%%
$$
\frac{\Gamma x = R_a }{\Gamma \vdash x:R}
$$%%
