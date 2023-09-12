#programminglanguage 

軽量で埋め込みやすいスクリプティング言語。音楽用ソフトでは[[Reaper]]とかでも使われている。

コンパイラは基本的なやつはANSI Cで実装されている。

JITコンパイラで超高速に動く別の処理系[[LuaJIT]]もある

Luaでオーディオビジュアルライブコーディングをするための[[LuaAV]]などがある（更新されてないけど）

動的型付けで手続き型っぽいが、関数を第一級オブジェクトとして扱える。そのためクロージャを使用できるが、この実装にupvalueという方式を採用していることで有名（？）

tableというArrayと連想配列の両方が扱えるような唯一のデータ構造をもち、tableに関数を放り込むことでオブジェクト指向っぽくメソッドを定義することもできる。

FFIでは、C言語側でユーザーデータというC言語側からしか直接触れないデータを経由する（LuaからはC言語側で別途定義したデータ操作用の関数を叩く）ことでアプリと繋ぐことがしやすい仕組みになっている。標準ライブラリのファイルIOとかもこの仕組みで実装されているらしい。

GCは最新の実装ではインクリメンタルマーク＆スイープ。

Cからアクセスするには`luaState*`を受け取って`int`を返す関数だけを通じて、LuaのVMのスタックに値を積んだり、取り出したりすることで操作するような感じになる。

数値型は基本的に64bit floatのみ（埋め込む時はコンパイル時にマクロで切り替えることができる）。内部的にはIntとかboolの区別はあって、いい感じにキャストできるようになっている。



## リファレンス

[Lua 5.4 Reference Manual](https://www.lua.org/manual/5.4/)

[Lua 5.3 マニュアル日本語訳](http://milkpot.sakura.ne.jp/lua/lua53_manual_ja.html)

言語機能もそうだが、埋め込み用のC APIの説明では結局内部の設計について知らないと動かせないので言語開発者向けの情報が多い感じ。

## upvalue関連

[1パスコンパイラでのクロージャの実装方法(Upvalue)](https://tyfkda.github.io/blog/2020/01/03/clox-closure.html)

[[Crafting Interpreter]]で解説されているUpvalueの日本語での分かりやすい解説
## Rustでの実装

[Writing a minimal Lua implementation with a virtual machine from scratch in Rust](https://notes.eatonphil.com/lua-in-rust.html)

パーサと簡単なVMの実装。GCとかはない

[Build a Lua Interpreter in Rust](https://wubingzheng.github.io/build-lua-in-rust/en/)

ほぼフル機能の実装チュートリアル。中国語からの英語自動翻訳だが普通に読める。特にクロージャやupvalue周りの実装が参考になりそう。GCに関してはRustの`Rc`をそのまま使っているため、VMの機能としては存在しない。

Lua C APIではCのクロージャを投げるときには、そもそもC言語にクロージャがないので何が嬉しいのかよくわからない感じだったが、このチュートリアルの実装だとRust側から`Rc<RefCell<Box<dyn FnMut>>>`のクロージャを投げることができるので、Rustの中でLuaを埋め込むんだったら分かりやすくて嬉しい。

[purua](https://github.com/udzura/purua/tree/2a2007a562fd6e2fdfa6183b0558c4809ec2a212)

ミニマルな実験的実装。GCとかなさそうだけど、その分コンパクトなので分かりやすいかも。なぜか途中でコードをほぼ全部消すコミットが積まれていて最新版は中身がなくなっている・・・