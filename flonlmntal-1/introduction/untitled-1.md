---
description: 変数を追加します。
---

# 束縛、環境

今まで出てきていなかった「変数」を追加しよう。

「変数」は基本的に「変数名」と「値」を持つ。この関係を「束縛」という。例えば、`let x = 1`では`x`は`1`に束縛されている。

「束縛」の集合を「環境」という。例えば、「xは1に束縛されていて、yは2に束縛されている」などのものがあり得る。

今回は「変数名」は文字列アトムで表すことにする。また、「環境」を`[let(“x”, 1), let(“y”, 2)]`のようなリストで表すことにする。「値」の探索はリストを先頭から辿っていって行う。

「変数」の評価にはこの「環境」が必要になるため、`eval`は評価する式とともに「環境」をその子ノードとして持つことにする。

最初は「環境」は空であるので、テストコードは以下のようになる。

```text
 test = eval(1 + 2 * 3, []).
```

`eval`に空のリストを追加した。



## let式の評価

`let x = 1`のような式を`R = let(Var, Val, Body)`で表す。ただし、`Var`は文字列アトムが接続される。この式を評価する場合、まず`Val`が評価され「環境」に`let(Var, Val)`を追加し、`Body`の評価を行う。

```text
 eval_let_val @@
 R = eval(let(Var, Val, Body), Env) :- 
 ground(Env) |
 R = fireable(let, Var, eval(Val, Env), Body, Env).
 
 eval_let_body @@
 R = fireable(let, Var, val(Val), Body, Env) :- 
 R = eval(Body, [let(Var, val(Val))|Env]).
```

このように、多くの式において「環境」は複製しなくてはいけない。

## 変数の解消

「変数（名）」を評価する場合は「環境」を辿って、対応する「値」を求めてやる必要がある。

```text
 resolve_val @@
 R = eval(Var1, [let(Var2, Val)|T]) :- 
 string(Var1), Var1 == Var2, ground(T) |
 R = Val.
 
 lookup_val @@
 R = eval(Var1, [let(Var2, Val)|T]) :- 
 string(Var1), Var1 \= Var2, ground(Val) |
 R = eval(Var1, T).
```

## 変更したルール

`eval`に「環境」を追加したため、今までに登場したものも以下のように多少変更が必要になるものがある。ここでは変更したルールだけ記載した。基本的に`eval`（や`fireable`）に「環境」も追加しているだけだ。

```text
 eval_left_child @@
 R = eval(binop(OP, X, Y), Env) :-
 ground(Env) |
 R = almost_fireable(OP, eval(X, Env), Y, Env).
 
 eval_right_child @@
 R = almost_fireable(OP, val(X), Y, Env) :-
 R = fireable(OP, val(X), eval(Y, Env)).
 
 eval_value @@
 R = eval(Int, Env) :- unary(Int), ground(Env) | R = val(Int).
 R = eval(true, Env) :- ground(Env) | R = val(true).
 R = eval(false, Env) :- ground(Env) | R = val(false).
 
 
 R = eval(not(X), Env) :- R = fireable(not, eval(X, Env)).
 
 eval_condition @@
 R = eval(if(B, X, Y), Env) :- 
 ground(Env) | 
 R = fireable(if, eval(B, Env), X, Y, Env).
 
 eval_then @@
 R = fireable(if, val(true), X, Y, Env) :- 
 ground(Y) | 
 R = eval(X, Env). 
 
 eval_else @@
 R = fireable(if, val(false), X, Y, Env) :- 
 ground(X) | 
 R = eval(Y, Env).
```

