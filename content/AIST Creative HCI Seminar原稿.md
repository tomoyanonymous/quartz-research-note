#talk #programming #music #creativity-support-tool

https://chci.pages.dev/aist-seminar

2023/11/23(木)

---

## イントロダクション

こんにちは。[[松浦知也]]です。

本日は、この様な機会を与えてくださってありがとうございます。
この発表では芸術表現のための創造性支援ツールというテーマの中で、先ほどXavierがプログラマブルなDAW（[[PAW]]）という実例を見せてくれたこともあるので、私はより理論や歴史的な背景を補足しようと思います。

その上で、現在私が開発している関数型プログラミングとDAWを統合したソフトウェア[[otopoiesis]]について簡単に紹介します。

## 自己紹介(ここまで1分)

私は現在東京芸術大学の芸術情報センターという、大学のネットワークインフラを管理したり、学生向けのFablab的な設備を提供している場所で働いています。

私は自分のことを音楽土木工学（Civil Engineering of Music）との研究者と呼んでいます。
これは実際には存在しない学問領域ですが、名前の通り、テクノロジーを音楽に応用するのではなく、音楽実践を通じて基幹的な技術インフラを考え直す学問です。

具体的には、自作楽器を使っての演奏活動や、音楽のためのプログラミング言語”mimium”の開発や、今日お話しするプログラマブルな音楽ソフトウェアotopoiesisの設計などに取り組んでいます。


## 音楽とプログラミングの歴史(4分)

要点：音楽、音声信号をコンピューター上で表現するためには、音にまつわるデータをどの様に定義するか自体が重要だが、UGen以外のモデルは検討されてきたのだろうか？

### 1950s: 音響遅延線メモリ、デバッグのための音の利用、転じてメロディの実現（hooter）

### 1960s: MUSIC N / UGenパラダイムの登場、モジュラーシンセサイザーとの等価性

### 1970s: [[GROOVE]] / Alan Kay, Metamedia
### 1980s: UGenのハードウェア化

### 1990s Digidesign Accelarator

## 音楽における”Direct Manipulation”（4分）

ビジュアルの場合は、出力される画像・映像を確かに直接操作している。音楽制作ソフトウェアにおけるGUIは一体何を「直接」操作しているのか？

→音を生成するメカニズムの視覚表象
	テープ、鍵盤、シーケンサー、パンチカード、エフェクターのつまみ

問題：現実のメタファーに頼ると現実世界に存在する程度のことしかできない

- ビジュアルプログラミングはビジュアルで表現できる程度の複雑さまでしか表現できない
- データの再帰的抽象化ができない
	- 例えば、Maxのようなデータフロープログラミングはデータフローの中にパッチそのものを流すことはできない[^kronos]


[^kronos]: [[Kronos]]のビジュアルエディターVeneerとかはlambdaオブジェクトが存在するなど、それに近しいことを実現してはいる。

## 音楽プログラミングとデータ抽象 (3分)

- プログラマブルなDAWはいくつかある（例えば[[Reaper]]、[[Ardour]]など）。しかし新しいデータ型を作れるわけではない・・・つまりオートメーションである。
- また、CSoundにDAWっぽいフロントエンドを載せたプロジェクトとして[[Blue]]などがある。
	- しかし、CSound自体のメタ表現力はそんなに高くはない
- タイムラインベースの汎用プログラミング環境もある。例えば[[OSSIA Score]]や[[IanniX]]
	- しかし、これもデータの抽象化そのものができるかというと
- インタラクティブな音楽を作るためのものとして、ゲーム音楽用のオーディオミドルウェアというのもある。[[Wwise]]や[[ADX]]、[[Fmod]]
	- しかし、どっちかというとこれもDAWと連携する方向だよね

### グラフベースからの脱却

### DAWのデータフォーマット

https://github.com/bitwig/dawproject

例えばDAWprojectの様なオープン化を目指したものが出てきている

昔ながらのもので言えば、OMFやAAF（オーディオ・ビデオデータのみ）とSMF（シーケンスデータ、複数のMIDIデータの集合）とか

Program as a Format - MPEG-Structured Audio(CSoundベースの音源配布フォーマット)


## [[otopoiesis]]について


### プロジェクトの構造

まだシンタックス（パーサー）が実装されてないので、Rust風の擬似ソースコード

```rust
let sinewave =|freq,amp,phase|{
	...
}
let apply_fadeinout = |start,dur,time_in,time_out,content|{
	...
}
let FadeInOut = |time_in,time_out,origin|{
	let time_in = Param("time_in",0.01,0.0..inf);
	let time_out = Param("time_out",0.01,0.0..inf);
	Region({
		start: origin.start
		dur  : origin.dur
		content: apply_fadeinout(start,dur,time_in,time_out,origin.content)
	})	
}
let project = |sample_rate|{
	let tracks = Track([
		FadeInOut(
			0.1,
			0.1,
			region: {
						start:0.0,
						dur: 1.0,
						content: || sinewave{
							Param("freq",440.0,20.0..20000.0)
							Param("amp",1.0,0.0..1.0)
							Param("phase",0.0,0.0..1.0)
						}
					}
		)
	]);
	tracks.map(|t| t.render()).sum()
}

```