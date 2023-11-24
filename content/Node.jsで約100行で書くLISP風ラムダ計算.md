#programming-language #compiler-design #lisp 

[Make A Lisp](https://github.com/kanaka/mal)よりさらに最小限の実装ってどのくらいかなー、というのを考えた。Cons、Cdr、Carは無い。演算は四則演算とifのみ。その代わりクロージャは作れるのでこのくらいのことはできる。

```lisp
(let double (lambda (x) (* x x)) (double 6)) 
```

ただ再帰は今のところ難しい。多分ListとAtomの区別がきちんとできてないせいで不動点コンビネータがきちんと動かない。

あくまで雰囲気重視で、ということなのだが、結果的にコードが短いため初学者にわかりやすいかと言われると、かなりコメントを足していかないと読めなさそうで矛盾してる気がする。

```js
const reader = require("readline").createInterface({
    input: process.stdin,
});

const tokenize = (str) =>
    str.match(/[\(]|[\)]|[\+\-\*\/]|[a-zA-Z_]+|(-?\d+(\.\d+)?)/g);

const parse_token = (token, stack, res) => {
    switch (token) {
        case '(': {
            let child = [];
            stack.push(child);
            return child;
        }
        case ')': {
            let c = stack.pop();
            if (stack.length == 0) {
                return c
            } else {
                stack[stack.length - 1].push(c)
                return stack[stack.length - 1]
            }
        }
        default: {
            res.push(token);
            return res
        }
    }
}

const parse = (tokens) => {
    let stack = [];
    let next_res = [];
    for (token of tokens) {
        next_res = parse_token(token, stack, next_res)
    }
    return next_res
}
const read = (str) => parse(tokenize(str));

const binop = (args, fn) => fn(args[0], args[1]);

const env = {
    "parent": null,
    "+": args => binop(args, (a, b) => a + b),
    "-": args => binop(args, (a, b) => a - b),
    "*": args => binop(args, (a, b) => a * b),
    "/": args => binop(args, (a, b) => a / b),
    "if": args => args[0] ? args[1] : args[2]
}

const try_find = (name, env) => {
    if (env)
        if (env[name]) {
            return env[name]
        } else {
            return try_find(name, env["parent"])
        } else {
        return null
    }
}
const eval = (expr, env) => {
	if (expr ==null){
		return null;
	}
    if (!Array.isArray(expr)) {
        return !isNaN(parseFloat(expr)) ? parseFloat(expr) : try_find(expr, env)
    } else {
        const op = expr.shift();
        switch (op) {
            case 'let': {
                let res = eval(expr[1], env);
                let newenv = { "parent": env };
                newenv[expr[0]] = res;
                return eval(expr[2], newenv)
            }
            case 'lambda': return ['closure', expr[0], expr[1], env]//ids,body,env
            default: {
				let fn = eval(op, env);
                if (fn) {
                    if (Array.isArray(fn) && fn[0] === "closure") {//closure
                        for (let i = 0; i < fn[1].length; i++) {
                             fn[3][fn[1][i]] = eval(expr[i],env)
                        }
                        return eval(fn[2],  fn[3])
                    } else {//extern function
                        return fn(expr.map(e => eval(e, env)))
                    }
                } else {
	                 return fn
                }
            }
        }
    }
}
const rep = (line) => {
    console.log(eval(read(line), env))
}
reader.on("line", (line) => {
    if (line) { rep(line); }
});
```