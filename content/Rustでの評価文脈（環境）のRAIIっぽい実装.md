#programming-language #compiler-design

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