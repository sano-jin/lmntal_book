---
description: 再帰関数を簡単に定義するための「let rec」を用意します。
---

# 再帰関数

実は先の「再帰しない関数」で不動点コンビネータを定義出来るのだが、それで終わらせるのはあんまりだ。ということで`let_rec`も用意しよう。

## let rec の評価

基本的に再帰関数は自分の中の式に自分の名前を記述することによって実現される（もちろん無名再帰というのもある）。従って「自分の名前」を覚えておく必要がある。

再帰関数の定義を行う際は`R = let_rec(FN, Var, FB, Body)`のように表現することにする。 `FN`はその再帰関数の名前である。`let rec fn var = fb in body`のイメージだ。 `let_rec`を評価すると、`FN`に再帰関数（本当はクロージャ）`val(rec_fun(FN, Var, FB, Body))`が束縛され環境に追加される。ここで、このクロージャがその再帰関数の「名前」も持っていることに注意しよう。

```text
eval_let_rec @@
 R = eval(let_rec(FN, Var, FB, Body), Env) :- 
 string(Var), ground(Env), string(FN) |
 R = eval(Body, [let(FN, val(rec_fun(FN, Var, FB, Env)))|Env]. 
```

カリー化のための糖衣構文解消用ルールも追加してしまおう。

```text
 R = let_rec(FN, [Var|T], FB, Body) :- 
 R = let_rec(FN, Var, fun(T, FB), Body). 
```



## 適用の評価

「適用」は再帰しない関数とほとんど一緒だが、再帰関数は自らの中で再度呼ばれる可能性があるため、「環境」に自分自身をもう一度追加しておいてやる必要がある。

```text
app_rec_fun @@
 R = fireable(app, val(rec_fun(FN, Var, Body, Env)), val(Val)) :- 
 ground(Env), ground(Body), string(FN), string(Var) |
 R = eval(Body, [let(Var, val(Val)), 
	let(FN, val(rec_fun(FN, Var, Body, Env)))|Env]). 
```

