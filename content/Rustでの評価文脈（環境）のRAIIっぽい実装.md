#programminglanguage #compiler-design

[chumskyのチュートリアル](https://github.com/zesterer/chumsky/blob/main/tutorial.md)で、評価する関数の実装がライフタイム付きでこんな感じになってたの頭いいなと思ったので、RAIIにしたらもっとシンプルに見えるのではと思った

OCamlとかの関数型言語とかみたいに、`env :: newbind`とかするのはRustではイテレータとか使いづらいしやっぱ微妙、というのもある

```rust
fn eval<'a>(expr: &'a Expr, vars: &mut Vec<(&'a String, f64)>) -> Result<f64, String> {
    match expr {
	...
        Expr::Let { name, rhs, then } => {
            let rhs = eval(rhs, vars)?;
            vars.push((name, rhs));
            let output = eval(then, vars);
            vars.pop();
            output
        },
	...
	
	}
}
```

Environmentは評価全体で見ればLetやLambdaごとに分岐していく構造だけれど、局所的には1列のベクタで表現できるので、実は`Vec`で十分

こんな感じすかねえ

[[Rust]]初心者には逆にわかりにくかもしれないな、、、

lookupでは値をコピーして返してる（このオブジェクトが有効な期間は中身のベクタの不変参照を返すことができないため）

```rust
struct EnvironmentT<'a, T: Clone>(&'a mut Vec<(String, T)>, usize);

impl<'a, T: Clone> EnvironmentT<'a, T> {
    pub fn new(vec: &'a mut Vec<(String, T)>, mut names: Vec<(String, T)>) -> Self {
        let len = vec.len();
        vec.append(&mut names);
        Self(vec, len)
    }
    pub fn lookup(&self, name: &String) -> Option<T> {
        let res = self
            .0
            .iter()
            .rev()
            .filter(|(n, _v)| name == n)
            .collect::<Vec<_>>();
        res.get(0).map(|(_, v)| v.clone())
    }
}

impl<'a, T: Clone> Drop for EnvironmentT<'a, T> {
    fn drop(&mut self) {
        self.0.truncate(self.1);
    }
}
```

---

なんかDropのタイミングがうまく制御できずコンパイルを通せない、、、
し、綺麗ではあるけど言うほど記述量が減るわけではない

ので、evalを相互再帰するヘルパー関数とかでこういう感じにした方が楽かも

```rust
fn lookup(env: &mut Vec<(String, Value)>, name: &String) -> Option<Value> {
    let res = env
        .iter()
        .rev()
        .filter(|(n, _v)| name == n)
        .collect::<Vec<_>>();
    res.get(0).map(|(_, v)| v.clone())
}
fn eval_with_new_env<'a>(
    e_meta: Box<WithMeta<ast::Expr>>,
    env: &'a mut Vec<(String, Value)>,
    mut names: Vec<(String, Value)>,
) -> Result<Value, Error> {
    let len_origin = env.len();
    env.append(&mut names);
    let res = eval_ast(e_meta, env);
    env.truncate(len_origin);
    res
}
fn eval_ast<'a>(
    e_meta: Box<WithMeta<ast::Expr>>,
    env: &'a mut Vec<(String, Value)>,
) -> Result<Value, Error>{
	...
	match e {
	...
        ast::Expr::Let(TypedId { id, ty: _t }, e, then) => {
            let e_v = eval_ast(e, env)?;
            match then {
                Some(t) => eval_with_new_env(t, env, vec![(id, e_v)]),
                None => Ok(Value::Unit),
            }
        }
	...
	
	}

}
```