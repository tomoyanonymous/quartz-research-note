#programming-language #memo #logic

[[Coq]]を用いた定理証明支援の基礎

- [Software Foundations 日本語訳](https://chiguri.info/sfja/)
	- 大体これ読めばいいっぽい（プログラミング言語系は特に）
	- 京都大学　[[五十嵐淳]] [「計算と論理」　授業資料(2023)](https://www.fos.kuis.kyoto-u.ac.jp/~igarashi/class/cal/)
- 神戸大学 [Coq による定理証明入門  髙橋真(2023)](http://herb.h.kobe-u.ac.jp/coq/coq.pdf)
- 千葉大学大学院 [定理証明支援系 Coq での 論理的なリーゾニング 集中講義 アフェルト レナルド(2017)](https://staff.aist.go.jp/reynald.affeldt/coq/riron.pdf)
- IIJ技術研究所 [Coq を始めよう 池渕未来(2011)](https://www.iijlab.net/activities/programming-coq/coqt1.html)
- [Real World Coq – An introduction to effective theorem proving](https://ejgallego.github.io/real-world-coq/) by  [[Emilio Jesús Gallego Arias]]
	- <https://x80.org/rwc/code/lesson_1.html> jsCoqを使ってインタラクティブに学べる（DeepLとかのブラウザ拡張でレイアウトが崩れるっぽい）
## 備忘録

- `Inductive`が変数宣言
- `Definition`が再帰しない関数宣言
- `Fixpoint`が再帰関数の宣言
	- 再帰しないとwarningが出る

・・・というのは全部、あくまで普通の関数型言語で考えたら、というので、厳密には全然違う

問題はCoqがラムダキューブ最上位のCalculus of Constructionを採用しているところ。つまり次の全てが扱える

- 項に依存する項 単純型付ラムダ計算
- 型に依存する項（ジェネリクス）System F
- 型に依存する型（カインド多相）System F $\underline{\omega}$
- 型依存の型と型依存の項の両方 System F$\omega$
- 項に依存する型（依存型） $\lambda\Pi$
- 型依存の項と依存型　$\lambda\Pi2$
- 型依存の項、項依存の型、型依存の項 CoC

Theoremとかで型を取って型を返す関数とかも当然作れる

> inductives are truly "pieces of data"

```ocaml
Inductive natural : Type :=
  | O : natural
  | S : natural -> natural.
```

自然数を表す型`natural`があったとすると、2のことを表す`S(S(O))`は`natural`に属する項だけど、`natural`型という集合の一部でもある…みたいな話だよな


- 証明したい命題は`Goal`、`Theorem`、`Lemma`
	- `Goal`は後で再利用されない、最後に証明したいもの
	- `Theorem`と`Lemma`に違いはない
		- こういうのめんどくさいな
- `Notation`って単純な文字列置き換えマクロとして実装されてるのかしら？

