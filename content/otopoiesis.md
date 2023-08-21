
#programming-language #sound

https://github.com/tomoyanonymous/otopoiesis

DAWをプログラマブルにする試み

---

## 構造

いわゆるオーディオプラグインのように、各コンポーネントがUI＋オーディオ処理を記述していて、それをツリー状に組み合わせていくのではなく、プロジェクトツリー、UIのツリー、オーディオ処理用の構造体とそれぞれツリーができ、全てのノードが対応づけされている

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

