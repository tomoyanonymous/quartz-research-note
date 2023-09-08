#memo #mimium #programming-language 

[[音楽プログラミング言語の形式化#mimium と 多段階計算]]

[[多段階計算]]を取り入れたい

とりあえず$W$ Calculusを自然に拡張してみる。
$W$ Calculusとmimiumの形式は似ているが、主に2つの違いがある。

1.　$W$ Calculus はLinear-Time Invariant なシステムを想定しているため、基本演算は項の加算と、項と定数の乗算しか使えない。
2. $W$ Calculusでは関数が数をとって数を返すものしかない。つまり、関数やfeedの項を取ったり返すような高階関数は想定されていない。

問題になるのは後者の方だ。

## 型



n以下の自然数$I_n$ (ディレイのbounded access用)

$$
\begin{align}
\tau ::=&\quad R_a \quad & a \in \mathbb{N}\\
      |&\quad I_n \quad &n \in \mathbb{N} \\
      |&\quad \tau → \tau \quad &a,b \in \mathbb{N}\\
      % |&\quad \langle \tau \rangle
\end{align}
$$
とりあえず1要素のタプルと普通のRは区別しないことにする
(そしてよく見るとこれは関数→関数のような高階関数を許してないんだな)
そうか高階関数を考えなければクロージャを考慮する必要もないものな

## 値

一旦タプルについては考えないことにしよう

$$
\begin{align}
v \; ::= & \quad R \\
	| & \quad \lambda x:T.e  \quad & [lambda]\\
     |& \quad feed \; x.e \quad & [feed] \\
\end{align}
$$
## 項

項

$$
\begin{align}
e \; ::=& \quad x \quad x \in \mathbb{V} \quad & [value]\\
     |& \quad \lambda x.e  \quad & [lambda]\\
     |& \quad fix \; x.e  \quad & [fix]\\
     |& \quad feed \; x.e \quad & [feed] \\
     |& \quad e \; e \quad & [app]\\
     %%|& \quad (e_1,e_2) \quad & [product]\\
     %%|& \quad \pi_n e \quad n\in \mathbb{N},\; n>0 \quad & [project]\\
     %%|& \quad \langle e \rangle \quad & [code] \\
     %%|& \quad \textasciitilde e \quad & [escape]
\end{align}
$$

基本演算（Intrinsic）は直感に任せる

本来はfixの中でfeedを使ったり、feedの中でfixを使うとエラーだが、結局シンタックスレベルでは排除できないので型でエラーとして弾くことにする…？
いや値レベルでの切り分けは不可能なので、こうする



## 実例

```rust
fn 

```