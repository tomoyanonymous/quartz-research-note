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
      |&\quad I_n \quad &n \in \mathbb{N} \\\
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
	| & \quad \lambda x:\tau.e  \quad & [lambda]\\
     |& \quad feed \; x.e \quad & [feed] \\
\end{align}
$$
## 項


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
fn cascade_f(order:int,fb,x)->float{
	letrec cascade = if(order>0){
		|x|{
			cascade(N-1)(x) *(1-fb) + self*fb 
		}
	}else{
		|x| x
	}
	cascade(x)
}
```

まあ、ともあれコピーキャプチャのクロージャでも問題ないってことですね

```rust
fn cascade(order:int,fb)->(float->float){
	if(order>0){
		|x|{
			cascade(N-1)(x) *(1-fb) + self*fb 
		}
	}else{
		|x| x
	}
}
```


ちょっとわかりやすさのために`self`を使わずfeedにしてみる

```rust
fn cascade(order:int,fb)->(float->float){
	if(order>0){
		|x|{
			feed(y) { cascade(N-1)(x) *(1-fb) + y*fb }
		}
	}else{
		|x| x
	}
}
```

あー、今までは`fn(x)`でself使うものを`feed(self).lambda(x).e,`って感じに自動的に変換してたけど、変換するとしたら`lambda(x).feed(x).e`の方が良かったってことなんだな

これを`cascade(3,0.9)`とかで簡約してみるか

```rust
cascade(3,0.9)

|x|{
	let res = cascade(2)(x);
	feed(y) { res*0.1 + y*0.9 }
}
|x1|{
	let res1 =|x2|{
		 let res2 = cascade(1)(x2);
		 feed(y2) { res2*0.1 + y2*0.9 }
	}(x1);
	feed(y1) { res1*0.1 + y1*0.9 }
}
|x1|{
	let res1 =|x2|{
		 let res2 = |x3|{
			 let res3 = cascade(0)(x3);
			 feed(y3) { res3*0.1 + y3*0.9 }
		}(x2);
		 feed(y2) { res2*0.1 + y2*0.9 }
	}(x1);
	feed(y1) { res1*0.1 + y1*0.9 }
}
|x1|{
	let res1 =|x2|{
		 let res2 = |x3|{
			 let res3 = |x|{x}(x3);
			 feed(y3) { res3*0.1 + y3*0.9 }
		}(x2);
		 feed(y2) { res2*0.1 + y2*0.9 }
	}(x1);
	feed(y1) { res1*0.1 + y1*0.9 }
}
|x1|{
	let res1 =|x2|{
		 let res2 = |x3|{
			 let res3 = x3;
			 feed(y3) { res3*0.1 + y3*0.9 }
		}(x2);
		 feed(y2) { res2*0.1 + y2*0.9 }
	}(x1);
	feed(y1) { res1*0.1 + y1*0.9 }
}
|x1|{
	let res1 =|x2|{
		 let res2 = |x3|{
			 feed(y3) { x3*0.1 + y3*0.9 }
		}(x2);
		 feed(y2) { res2*0.1 + y2*0.9 }
	}(x1);
	feed(y1) { res1*0.1 + y1*0.9 }
}
|x1|{
	let res1 =|x2|{
		 let res2 = feed(y3) { x2*0.1 + y3*0.9 };
		 feed(y2) { res2*0.1 + y2*0.9 }
	}(x1);
	feed(y1) { res1*0.1 + y1*0.9 }
}
|x1|{
	let res1 =|x2|{
		 feed(y2) { feed(y3) { x2*0.1 + y3*0.9 } *0.1 + y2*0.9 }
	}(x1);
	feed(y1) { res1 *0.1 + y1*0.9 }
}
|x1|{
	let res1 = feed(y2) { feed(y3) { x1 *0.1 + y3*0.9 } *0.1 + y2*0.9 };
	feed(y1) { res1 *0.1 + y1*0.9 }
}
|x1|{
	feed(y1) { 
		feed(y2) { 
			feed(y3) { x1*0.1 + y3*0.9} *0.1 + y2*0.9 }*0.1 + y1*0.9 }
}
```

feedの項に対する加算とか乗算の計算は簡約がしづらいなあ

時間0の時のyは全て0として、

```rust
|x1,y1_ref,y2_ref,y3_ref| {
	let y1 = *y1_ref;
	let y1_next= {
		let y2 = *y2_ref;
		let y2_next = {
			let y3 = *y3_ref;
			let y3_next = x1*0.1 + y3*0.9;
			*y3_ref = y3_next;
			y3_next
		}*0.1 + y2*0.9;
		*y2_ref = y2_next;
		y2_next	}*0.1 + y1*0.9;
	*y1_ref = y1_next;
	y1_next
}
```

やっぱdenotationalの方が定義しやすいかもなあ

ああでもfeedを無事に展開できるということは、feedの項に対して`Cell`を割り当てることそのものには間違いはないのか

ただ、例えばFeedの項の中に関数が残っちゃうような可能性もあるため、Feedのhistoryの中にLambdaの項が保存されるような状況が回避できない。

型付規則の中でfeed x.eのeがプリミティブというか、Boxedにならないものしか取れないようにすればいいのかね。そうするとValueはCopyトレイトを実装できて、feedの中に実際にはラムダが入ってたとしても、簡約後は必ずValueになっていると

## 型（改正版）

というわけで型の定義再訪

$$
\begin{align}
\tau_p ::=&\quad R_a \quad & a \in \mathbb{N}\\
      |&\quad I_n \quad &n \in \mathbb{N} \\
\tau  :: = &\quad \tau_p\\
      |&\quad \tau → \tau \quad &a,b \in \mathbb{N}\\
      % |&\quad \langle \tau \rangle
\end{align}
$$

でFeedの型付け規則(xが)こういう感じになると

$$
\frac{\Gamma, x:\tau^a \vdash e:\tau^b}{\Gamma \vdash \lambda x.e:\tau^a \to \tau^b }\\
\\
\frac{\Gamma, x : \tau_p^a \vdash e: \tau_p^a }{\Gamma \vdash feed\ x.e:\tau_p^a}
$$

タプルとかレコードもできるけど、関数をタプルの要素にしたりはできない（できないでもないけど、「そういう型をとれるタプル」と「そういうのできないタプル」を分けて考える必要がある）、って感じでユーザーにはややこしいですねえ