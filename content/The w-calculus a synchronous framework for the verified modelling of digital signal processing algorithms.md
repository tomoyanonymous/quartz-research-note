#paper #programming-language 

https://dl.acm.org/doi/10.1145/3471872.3472970

## 概要

ラムダ計算に`feed`という1サンプルフィードバックを導入する形式を取る

[[mimium]]で`self`を使って1サンプルフィードバックができる、というのと近い（ちょうどmimiumと同じカンファレンスで発表された）

ただし対象は線形時不変システムの形式化で、異なるトポロジーのフィルターが最終的に同一のものであるのが証明できたりすると嬉しいよね、みたいなモチベだと理解してる（なので、項同士の加算は定義されているが、乗算は項と定数のみに制限されている）


[[Coq]]で証明が実装されている https://github.com/ejgallego/mini-wagner-coq/

よく見たらこのAuthorの[[Emilio Jesús Gallego Arias]]、Coq本体の開発をゴリゴリやってる人だ（強いわけだ）



### 実装

最終的にはA:意味論としてわかりやすいDenotational、B:有限なメモリと計算結果の再利用を考慮したImperativeなインタプリタが[[OCaml]]で、C:[[MetaOCaml]]を使って最適化したコードの3つが出てくる。

#### A:Denotationalなインタプリタ

- `machine`がある時刻のサンプルを計算し返却する関数
- `machine_n`が指定した時刻までのすべてのサンプルを計算しリストとして返却する関数

以上2つの関数を相互再帰させている
毎サンプルごとに全てのサンプルのhistoryを再生成してたりするので著しく非効率的

```ocaml
let rec machine_n : type a. int → a expr → env → a list = 
	fun step e env → 
		if step < 0 
			then [] 
			else machine step e env :: machine_n (step-1) e env 
and machine : type a . int → a expr → env → a = 
	fun step e env → match e with 
		| Var (idx, id) → lookup env id idx 
		| Lam e → fun hist → machine step e (hist :: env) 
		| App (ef, ea) → let a_hist = machine_n step ea env in 
			machine step ef env a_hist 
		| Feed e → let e_hist = machine_n (step-1) (Feed e) env in 
			machine step e (e_hist::env) 
		|...
```


#### B:Imperativeなインタプリタ

この実装が多分現在のmimiumの実装に近い

```ocaml
let heap = ... (* pre-allocated *) 
let rec imachine : expr → env → value = 
	fun e env → match e with 
		| Var (idx, id) → 
			let p = ptr_of id env in 
				lookup heap p idx 
		| Lam e → 
			fun ptr → imachine e (ptr :: env) 
		| App (p,ef,ea) → 
			let va = imachine ea env in 
				shift_one heap ptr va; imachine ef env ptr 
		| Feed (ptr,e) → 
			let v = imachine e (ptr::env) in 
				shift_one heap ptr v; v 
		|... 
		
let rec repeat : int → (unit → 'a) → 'a = 
	fun n c → 
		if n = 0
			then c () 
			else (c (); repeat (n-1) c) 

let eval e env n = repeat n (fun () → imachine e env)
```

#### C:[[MetaOCaml]]での多段階インタプリタ

```ocaml
let rec eval : type a. env → a expr → a code = 
	fun g e → match e with 
		| Cst r → let module M = Lift_array(Lift_float) in M.lift r 
		| Var id → 
			.< Array.(unsafe_get .~(heap) .~(List.nth g id)) >. 
		| Idx (vec, id) → 
			.<  let v = .~(eval g vec) in 
				let i = .~(eval g id) in 
					Array.(unsafe_get v i) >. 
		| Add (e1,e2) → 
			.<  let v1 = .~(eval g e1) in 
				let v2 = .~(eval g e2) in 
					WArray.map2 (+.) v1 v2 >. 
		| Lam e → 
			.< fun p → .~(eval (.<p>.::g) e)>. 
		| App (p, f, e) → 
			.< let v = .~(eval (.<p>.::g) e) in
				 shift_one v Array.(unsafe_get .~(heap) p); .~(eval g f) p >. 
		|...
```

---

A:DenotationalとB:Imperativeが等しい計算を行なっていることの形式的証明はOpen Questionとなっている。
