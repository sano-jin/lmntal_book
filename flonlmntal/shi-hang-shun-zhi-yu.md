---
description: 実行順を制御します
---

# 実行順制御

LMNtalは非決定性をもつ。しかしこの非決定性を残しておくと、再帰がbase caseで停止しなくなる可能性がある。従って、部分的にであれ、実行順を制御してやる必要がある。

{% hint style="info" %}
この制御は完全に決定的である必要はなく、むしろある程度の非決定性を残したプログラムの方が書きやすいし、より「LMNtalらしい」。しかし非決定性を少しでも残しておくと状態数が爆発的に増えてデバッグが困難になる。従ってとりあえずは非決定性を完璧に排除することにする。
{% endhint %}

ここで、いわゆる「普通の言語」のプログラムは大抵木構造になっている（c.f. 抽象構文木）。よって木構造を逐次的に処理出来ることが重要である。

### 二分木の深さ優先探索

簡単な例として二分木を深さ優先探索しよう。

ノードは`n(Left, Right, Parent)`である。葉は整数値である。 ここでは探索の過程をオイル（石油）が根本からノードを伝わってゆき、木が末端から徐々に燃えてゆくイメージで説明している（もっと良い説明の仕方があるとは思う）。

探索のために根ノードの親は`oil(root, R)`ということにして、その親は`test(R)`と言うことにする。

例

```text
test = oil(n(n(1,n(2,3)),4))
```

確認のために木を辿って葉から得た整数値をスタック`stack([])`へ積んでいくことにしよう。もしも処理がうまく行っているならば先の例であればスタックのリストには整数が降順に並ぶはずである。つまり、`stack([4,3,2,1])`を手に入れれば安心だ。

早速実装を見ていこう。

`oil`は引数のノードを湿らせ、そのノードは`almost_fireable`（あとちょいで発火可能）になる。`almost_fireable`なノードは自身の子ノードへ順に`oil`を滴らせていく。

```text
drip_oil_left @@
 P = oil(n(L, R)) :-
 P = almost_fireable(oil(L), R).
```

`almost_fireable`なノードは左の子ノードが`fired`（発火済み）であれば、右の子ノードへも`oil`を滴らせる。

```text
drip_oil_right @@
 P = almost_fireable(fired, R)) :- 
 P = fireable(fired, oil(R)).
```

ただし、ここでノードは`fireble`（発火可能）に改名する。さもなくば、永遠にこの操作を繰り返す羽目になるからだ。

`fireable`なノードはどちらの子ノードも発火していたら自らも発火する。

```text
fire @@
 P = fireable(fired, fired) :-
 P = fired.
```

 あとは葉の処理だけだ。葉はただ発火するだけでも良いが今回はスタックへ整数値を積む。 

```text
fire_leaf @@
 P = oil(Int), stack(S) :- 
 int(Int) | 
 P = fired, stack([Int|S]).
```

コード全文はこんな感じになる。 

```text
test = oil(n(n(1,n(2,3)),4)). stack([]).

drip_oil_left @@ 
 P = oil(n(L, R)) :- 
 P = almost_fireable(oil(L), R).

drip_oil_right @@ 
 P = almost_fireable(fired, R) :- 
 P = fireable(fired, oil(R)).

fire @@
 P = fireable(fired, fired) :- 
 P = fired.

fire_leaf @@ 
 P = oil(Int), stack(S) :- 
 int(Int) | 
 P = fired, stack([Int|S]).
```

プログラムを実行したら以下のような結果を得られるはずだ。 

```text
test(fired). stack([4,3,2,1]).
```

state viewer なども確認して欲しい。それぞれの状態は一直線に連なっているはずだ。

