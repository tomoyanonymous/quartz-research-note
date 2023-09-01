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

```rust
struct<'a,T> Environment(&'a mut Vec<T>)

impl<'a,T> Environment{
	pub fn new()->Self{
		Self(&mut Vec::new())
	}

}

```