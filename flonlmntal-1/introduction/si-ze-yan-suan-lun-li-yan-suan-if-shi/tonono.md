---
description: ちょっとした電卓を作ろう
---

# 積と和の式の評価

## 積と和の式の評価

まずは小手調べとして積と和だけの式を評価する。

ここで、「評価」とは何らかの計算などを行い、「値」を返すものである。「評価」は基本的に深さ優先的に行われる。

二分木の例では`oil`が伝わっていき、それぞれのノードは`fired`を返していた。ここでは`oil`を`eval`に、`fired`を`val`アトムにする。ただし`val`アトムには整数値を繋ぎ、これを戻り値とする。

二分木の例ではノードは`Parent = n(Left, Right)`であったが、構文木ではこれは積と和の二項演算に対応する。これらを`R = binop(OP, X, Y)`とあらわす。ここで、`OP`には`add`または`mul`アトムが接続される。

例

```text
% 1 + 2 * 3 --> 7
test = eval(binop(add, 1, binop(mul, 2, 3))).
```

子ノードへ評価を伝えてゆく部分に関しては二分木で行ったものと全く同じだ。

```text
eval_left_child @@
 R = eval(binop(OP, X, Y)) :-
 R = almost_fireable(OP, eval(X), Y).
 
eval_right_child @@
 R = almost_fireable(OP, val(X), Y) :-
 R = fireable(OP, val(X), eval(Y)).
```

それぞれの二項演算子は子ノード、つまり入力が全て値になったらそれぞれの演算を行う。

```text
add @@
 R = fireable(add, val(X), val(Y)) :-
 Z = X + Y |
 R = val(Z).
 
mul @@
 R = fireable(mul, val(X), val(Y)) :-
 Z = X * Y |
 R = val(Z).
```

葉ノードでは単に「値」を返す。

```text
eval_value @@
 R = eval(Int) :-
 int(Int) |
 R = val(Int).
```

コード全文はこんな感じになる。

```text
% 1 + 2 * 3
test = eval(binop(add, 1, binop(mul, 2, 3))).
 
eval_left_child @@
 R = eval(binop(OP, X, Y)) :-
 R = almost_fireable(OP, eval(X), Y).
 
eval_right_child @@
 R = almost_fireable(OP, val(X), Y) :-
 R = fireable(OP, val(X), eval(Y)).
 
add @@
 R = fireable(add, val(X), val(Y)) :-
 Z = X + Y |
 R = val(Z).
 
mul @@
 R = fireable(mul, val(X), val(Y)) :-
 Z = X * Y |
 R = val(Z).
 
eval_value @@
 R = eval(Int) :-
 int(Int) |
 R = val(Int).
```

プログラムを実行したら以下のような結果を得られるはずだ。

```text
test(val(7)). 
```

state viewer なども確認して欲しい。

### 糖衣構文

積と和の式は`binop`なるアトムを使って表していた。これははっきり言って面倒くさい。

LMNtal Compiler はアトム`+`と`*`の結合強度を賢く判断し正しい構文木を生成する。よってこれらを中置記法で用いることもできる。

しかし、あえて`binop`アトムで統一したのはこうすることによって、子ノードをそれぞれ順番に評価してゆく部分をまとめて書けるからだ。今後二項演算は積と和だけでなく、差、論理和、論理積、大小比較、等価性判断、適用など沢山出てくる。これらの共通した処理をまとめて記述することはメンテナンス性を高めるためにも重要である。

従って、`+`と`*`などの中置記号は「糖衣構文（記述を簡単にするための略記法）」ということにして評価前に`binop`を用いた構造へ変換してやることにしよう。

糖衣構文解消用のルールを以下に示す。

```text
 R = X + Y :-
 R = binop(add, X, Y).
 
 R = X * Y :-
 R = binop(mul, X, Y).
```

この変換は非決定的に行われる。もちろんこれを決定的に行うようにすることもできるだろうが、面倒なので「ゼロステップ」ルールということにしてしまう。 ルールを「ゼロステップ」にするには膜を用意して、その中に`'$callback'(zerostep)`アトムと一緒に入れてやれば良い。これを「糖衣構文解消用のルール」と呼ぶことにする。

```text
{'$callback'(zerostep).
 R = X + Y :-
 R = binop(add, X, Y).
 
 R = X * Y :-
 R = binop(mul, X, Y).
}.
```

この膜（私は「ゼロステップ膜」と勝手に呼んでいるが恐らく正式名称ではないだろう）は通常の膜とは異なり、ステップ数を無視すれば基本的に膜がない状態と同じになる（ただし、モジュールと組み合わせて使おうとすると悲惨なことになる）。つまり、この膜の中のルールは膜の外のアトムにも反応する。

先の積と和の式の評価の例に糖衣構文解消用のルールも含めて以下の例を走らせてみて欲しい。

```text
% 1 + 2 * 3
test = eval(1 + 2 * 3).
```

前回の例題と同様の結果が得られるはずだ。state viewerも確認して同じ状態遷移をしていることを確かめて欲しい。

ちなみにコード全文はこんな感じなはずだ。

```text
% 1 + 2 * 3
test = eval(1 + 2 * 3).
 
eval_left_child @@
 R = eval(binop(OP, X, Y)) :-
 R = almost_fireable(OP, eval(X), Y).
 
eval_right_child @@
 R = almost_fireable(OP, val(X), Y) :-
 R = fireable(OP, val(X), eval(Y)).
 
 R = fireable(add, val(X), val(Y)) :- Z = X + Y | R = val(Z).
 R = fireable(mul, val(X), val(Y)) :- Z = X * Y | R = val(Z).
 
 R = eval(Int) :- int(Int) | R = val(Int).
 
{'$callback'(zerostep).
 R = X + Y :- R = binop(add, X, Y).
 R = X * Y :- R = binop(mul, X, Y).
 R = X - Y :- R = binop(sub, X, Y). 
}.
```

## 問題

引き算も追加してみよう

