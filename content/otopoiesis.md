
#programminglanguage #sound

https://github.com/tomoyanonymous/otopoiesis

DAWをプログラマブルにする試み

---

## 思想

Brandt(2002) の[[Temporal Type Constructor]]（以下TTC）という概念を使う。

TTCはジェネリックなタイプ`A`に対して、以下の3つの型コンストラクタを用意することでジェネリックに時間信号を取り扱う思想。

以下はRustの擬似コード。

```rust
type time = Real;
//時間に紐づいたイベント。MIDIノートとか
struct Event<A>{v:A, t:time} 
//有限ベクトル。オーディオファイルとか
type Vec<A> = std::Vec<A> 
//無限ベクトル、またはストリーム。1論理時刻毎にA型のものを返す漸化式（内部状態を持つかもしれない）
type iVec<A> = Box<dyn FnMut()->A> 
```

例えばMIDIの記録されたデータは

```rust
type NOTE= Event<(u8,u8)>//ノート番号、ベロシティ
type MIDI = Vec<NOTE> 
```

みたいになる

## 構造

基本的なイメージはこんな感じ？

```
type Project<V> = Vec<Track<_,__>> -> iVec<V>
type Track<I,O> =  Device<I> * Device<O> //デバイス情報
				*(
				  Vec<Region<O>> 
				| Vec<Event<I,O>>
				| Generator<O>
				)
type Region<V> = (time*time)* //start,duration
				(Vec<V> //　オーディオデータ
				  | Generator<V>
				  | Project<V>) //プロジェクトも再帰的に埋め込める
type Generator<T> = iVec<T>			
```

なんだけど、TrackAで使われてるGeneratorの中のParameterとしてTrackBの値をアサインしたい、みたいなことを表現できたらプログラミングとして面白くなる、という話

```
//Freq440Hz,Gain1.0,Phase0.0
let Track1 = Generator::SineWave(Constant(440),Constant(1.0),Constant(0.0));
let Track2 = Generator::SineWave(Track1,Constant(1.0),Constant(0.0));
```
これをあんまり動的ディスパッチじゃない感じで実装したい。そしてこの辺までは別にMaxとかと同じレベルの話

ここからがDAWをプログラミングで操作できる面白いとこで、例えばリージョンに対するフェードインアウトとかを`Region<T>->Region<T>`の関数として定義できるところ

CubaseにおけるインストゥルメントトラックとかはMIDIトラック＋シンセサイザーの合成なので、
`Track<NOTE,NOTE>`に`Vec<NOTE>->iVec<Audio>`みたいなのを適用する関数としてあらわせ、、、る？






---
以下は昔に考えていたこと

コードの例(モジュレーションされているサイン波＋ディレイ)

```
project{
	track: [
		delay{
			sinosc{
			freq:
				sinosc:{
					freq: float{20..20000,1000,"freq"}
					phase: 0.0},
			phase: 0.0
			},
		time: 1000
		}
	]
}
```

UIは基本的にプロジェクトツリーの`Param`型と、UIだけで使われる`State`をそれぞれ可変参照として持つ
(Reference カウントするのではなく、有限なライフタイムを持つ可変参照で作る)

```rust
struct UI<'a>{
	param: &'a mut Param,
	state: &'a mut State
}
impl<'a> egui::Widget for UI<'a>{
	fn ui(self, ui: &mut egui::Ui) -> egui::Response {
		///...
	}
}
```

eguiはimmiditate モードだから毎フレームこのUI型を生成している（egui標準のSliderとかもこの方式）

オーディオプロセッサーもこのやり方にできるか？


## 開発メモ

クリップのサムネイル生成はgeneratorじゃなくてregion側でやろう

fileplayerのui実装もgeneratorからregionに移そう

そうなるとaudio側の実装もそっちに合わせるのが自然だよな・・・

