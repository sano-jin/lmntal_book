# 論理演算、if式

## ブール式の評価

次は真偽値を扱う。

新たな二項演算として`and`と`or`を導入する。以下がその評価のためのルールだ。

```text
 R = fireable(and, val(true), val(Y)) :- R = val(Y).
 R = fireable(and, val(false), val(Y)) :- unary(Y) | R = val(false).
 
 R = fireable(or, val(false), val(Y)) :- R = val(Y).
 R = fireable(or, val(true), val(Y)) :- unary(Y) | R = val(true).
```

{% hint style="info" %}
短絡評価をしているように見えるかもしれないが、決定的にこれを行おうとすると少々面倒なのでしていない。実のところ双方の引数を評価してから演算を行っている。
{% endhint %}

これらの二項演算子もわざわざ`binop`と一緒に書くのは面倒なので以下の糖衣構文解消用ルールを追加する。

```text
 R = and(X, Y) :-
 R = binop(and, X, Y).
 
 R = or(X, Y) :-
 R = binop(or, X, Y).
```

算術演算子のように中置記法にすることは出来ないがないよりはマシだろう。

{% hint style="info" %}
記号以外の名前のアトムは中置記法には（私の知る限りでは）できない
{% endhint %}

また、単項演算子`not`用のルールを以下のように定義する。

```text
 R = eval(not(X)) :- R = fireable(not, eval(X)).
 R = fireable(not, val(true)) :- R = val(false).
 R = fireable(not, val(false)) :- R = val(true).
```

単項演算子が他にもあるなら`binop`のように単項演算子の入力を評価する部分をまとめるためのアトムを導入しても良いがとりあえずは単項演算子は`not`しか考えないのでこれで良いだろう。

肝心の真偽値の評価を行うルールを以下に示す。

```text
 R = eval(true) :- R = val(true).
 R = eval(false) :- R = val(false).
```

先の積と和の式の評価の例に新しく追加した`and`、`or`、`not`のルールやその糖衣構文解消用のルールも含めて以下の例を走らせてみて欲しい。

```text
% (not false) && (false || true) --> true 
test = eval(and(not(false), or(false, true))).
```

次のような結果を得られるはずだ。

```text
 test(val(true)).
```



## 関係演算子

各演算用のアトム名の意味は以下の通りである。それぞれ大文字で書いた部分のみ抜き出してアトム名とした。

* 「EQual to」
* 「Not EQual to」
* 「Less Than」
* 「Greater Than」
* 「Less than or Equal to」
* 「Greater than or Equal to」

以下のようなルールを追加する。

```text
 R = fireable(eq, val(X), val(Y)) :- X == Y | R = val(true).
 R = fireable(eq, val(X), val(Y)) :- X \= Y | R = val(false).
 
 R = fireable(neq, val(X), val(Y)) :- X \= Y | R = val(true).
 R = fireable(neq, val(X), val(Y)) :- X == Y | R = val(false).
 
 R = fireable(lt, val(X), val(Y)) :- X < Y | R = val(true).
 R = fireable(lt, val(X), val(Y)) :- X >= Y | R = val(false).
 
 R = fireable(gt, val(X), val(Y)) :- X > Y | R = val(true).
 R = fireable(gt, val(X), val(Y)) :- X =< Y | R = val(false).
 
 R = fireable(le, val(X), val(Y)) :- X =< Y | R = val(true).
 R = fireable(le, val(X), val(Y)) :- X > Y | R = val(false).
 
 R = fireable(ge, val(X), val(Y)) :- X >= Y | R = val(true).
 R = fireable(ge, val(X), val(Y)) :- X < Y | R = val(false).
```

`eq`、`neq`では`unary`型の比較演算子を用いている。これは`ground`型のものにすることもできるが、これで十分なのでしない（オーバーキルになる）。

これらの二項演算子もわざわざ`binop`と一緒に書くのは面倒なので以下のルールを糖衣構文解消用ルールへ追加する。

```text
 R = '='(X, Y) :- R = binop(eq, X, Y).
 R = '\='(X, Y) :- R = binop(neq, X, Y).
 
 R = '<'(X, Y) :- R = binop(lt, X, Y).
 R = '>'(X, Y) :- R = binop(gt, X, Y).
 
 R = '=<'(X, Y) :- R = binop(le, X, Y).
 R = '>='(X, Y) :- R = binop(ge, X, Y).
```



## if式の評価

いわゆる「普通の言語」において「if &lt;e1&gt; then &lt;e2&gt; else &lt;e3&gt;」は条件式&lt;e1&gt;が真であったときは&lt;e2&gt;を評価し、偽であったときは&lt;e3&gt;を評価する。

逆に&lt;e1&gt;が真であったときは&lt;e3&gt;を評価してはならないし、偽であったときは&lt;e2&gt;を評価してはならない。わざわざ評価順を細かく制御するようにしたのはこれを実現するためである。

参考までにLMNtalの「if」モジュールでは膜を用いてこの制御を行っている。ただし、関数型言語においては関数も第一級オブジェクトであり、「if」式を含んでいてもコピーや消去が出来なくては困る（コピーや消去がなくともなんとか出来るのかもしれないけど少なくともそう簡単ではないはず）。しかし、膜を含む構造は（それが含んでいる膜に言及することなく）コピーすることは出来ないため、膜を用いずに`eval`アトムなどを用いて逐次化している。

if式は`R = if(B, X, Y)`のように表現することにする。

```text
eval_condition @@
 R = eval(if(B, X, Y)) :-
 R = fireable(if, eval(B), X, Y).
 
eval_then @@
 R = fireable(if, val(true), X, Y) :-
 ground(Y) |
 R = eval(X).
 
eval_else @@
 R = fireable(if, val(false), X, Y) :-
 ground(X) |
 R = eval(Y).
```

* 例

  ```text
  % if 1 < 2 then 3 else 4 --> 3
  test = eval(if(1 < 2, 3, 4)).
  ```

以下のような結果を得られるはずだ。

```text
test(val(3)). 
```

