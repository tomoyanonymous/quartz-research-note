#programming-language #compiler-design #lisp 

[Make A Lisp](https://github.com/kanaka/mal)よりさらに最小限の実装ってどのくらいかなー、というのを考えた。

クロージャも作れるのでこのくらいのことはできる。

```lisp
(let double (lambda (x) (* x x)) (double 6)) 
```

まあプリミティブ演算は四則演算だけで再帰関数は自力で定義
```lisp
(let fix (lambda (f) ((lambda (x) (f (lambda (y) (x x y) ))) ())  ) )
```

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

const read = (str) => {
    const res = parse(tokenize(str));
    return res
}

const binop = (args, fn) => fn(args[0], args[1]);

const env = {
    "parent": null,
    "+": args => binop(args, (a, b) => a + b),
    "-": args => binop(args, (a, b) => a - b),
    "*": args => binop(args, (a, b) => a * b),
    "/": args => binop(args, (a, b) => a / b)
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
    if (!Array.isArray(expr)) {
        return !isNaN(parseFloat(expr)) ? parseFloat(expr) : try_find(expr, env)
    } else {
        const op = expr.shift();
        switch (op) {
            case 'let': {
                let newenv = { "parent": env };
                env[expr[0]] = eval(expr[1], newenv);
                return eval(expr[2], env)
            }
            case 'lambda': return ['closure', expr[0], expr[1], env]//ids,body,env
            default: {
                let fn = try_find(op, env);
                if (fn) {
                    if (Array.isArray(fn) && fn[0] === "closure") {
                        for (let i = 0; i < fn[1].length; i++) {
                             fn[3][fn[1][i]] = expr[i]
                        }
                        return eval(fn[2],  fn[3])
                    } else {
                        return fn(expr.map(e => eval(e, env)))
                    }
                } else {
                    console.error(`symbol ${op} not found`)
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