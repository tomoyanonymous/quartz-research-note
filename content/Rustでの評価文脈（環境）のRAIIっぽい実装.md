#programming-language #compiler-design

chumskyのチュートリアルで、評価する関数をライフタイム付きでこんな感じになってたの頭いいなと思ったので、RAIIにしたらもっとシンプルに見えるのではと思った

```rust
fn eval<'a>(expr:Expr,env:&'a mut Vec<(String,Val)>)->Result<Val,Error>{
	match expr {
	...
	Expr::Let(name,e,then)=>{
		env.push((name,eval(e,env)));
		let res = eval(then,env);
		env.pop();
		res
	}
	...
	
	}
}
```

Environmentは評価全体で見ればLetやLambdaごとに分岐していく構造だけれど、局所的には1列のベクタで表現できる

こんな感じすかねえ

```rust
struct EnvironmentT<'a, T>(&'a mut Vec<(String, T)>, usize);

impl<'a, T> EnvironmentT<'a, T> {
    pub fn new(vec: &'a mut Vec<(String, T)>, mut names: Vec<(String, T)>) -> Self {
        let len = vec.len();
        vec.append(&mut names);
        Self(vec, len)
    }
    pub fn drop(&mut self) {
        self.0.truncate(self.1);
    }
    pub fn lookup(&self, name: &String) -> Option<&'a T> {
        let res = self
            .0
            .iter()
            .rev()
            .filter(|(n, _v)| name == n)
            .collect::<Vec<_>>();
        res.get(0).map(|(_, v)| v)
    }
}
```