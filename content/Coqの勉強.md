#programming-language #memo #logic

[[Coq]]を用いた定理証明支援の基礎

- [Software Foundations 日本語訳](https://chiguri.info/sfja/)
	- 大体これ読めばいいっぽい（プログラミング言語系は特に）
	- 京都大学　[[五十嵐淳]] [「計算と論理」　授業資料(2023)](https://www.fos.kuis.kyoto-u.ac.jp/~igarashi/class/cal/)
- 神戸大学 [Coq による定理証明入門  髙橋真(2023)](http://herb.h.kobe-u.ac.jp/coq/coq.pdf)
- 千葉大学大学院 [定理証明支援系 Coq での 論理的なリーゾニング 集中講義 アフェルト レナルド(2017)](https://staff.aist.go.jp/reynald.affeldt/coq/riron.pdf)
- IIJ技術研究所 [Coq を始めよう 池渕未来(2011)](https://www.iijlab.net/activities/programming-coq/coqt1.html)
## 備忘録

- `Inductive`が変数宣言
- `Definition`が再帰しない関数宣言
- `Fixpoint`が再帰関数の宣言
	- 再帰しないとwarningが出る
- 証明したい命題は`Goal`、`Theorem`、`Lemma`
	- `Goal`は後で再利用されない、最後に証明したいもの
	- `Theorem`と`Lemma`に違いはない
		- こういうのめんどくさいな
- `Notation`って単純な文字列置き換えマクロとして実装されてるのかしら？

- Coqでは排中律が使えないので、ゴールの論理が論理和$A \cup B$とかの場合$A$か$B$のどちらかを証明するかを決めるしかない
- 
