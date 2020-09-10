# まとめ

## まとめ

今までに登場したものを全てまとめたものが以下になる。

```text
 % 1 + 2 * 3
 test = eval(binop(add, 1, binop(mul, 2, 3))).
 
 % (not false) && (false || true) --> true 
 test = eval(and(not(false), or(false, true))).
 
 % if 1 < 2 then 3 else 4 --> 3
 test = eval(if(1 < 2, 3, 4)).
 
 eval_left_child @@
 R = eval(binop(OP, X, Y)) :-
 R = almost_fireable(OP, eval(X), Y).
 
 eval_right_child @@
 R = almost_fireable(OP, val(X), Y) :-
 R = fireable(OP, val(X), eval(Y)).
 
 R = fireable(add, val(X), val(Y)) :- Z = X + Y | R = val(Z).
 R = fireable(mul, val(X), val(Y)) :- Z = X * Y | R = val(Z).
 R = fireable(sub, val(X), val(Y)) :- Z = X - Y | R = val(Z).
 
 R = eval(Int) :- int(Int) | R = val(Int).
 R = eval(true) :- R = val(true).
 R = eval(false) :- R = val(false).
 
 R = fireable(and, val(true), val(Y)) :- R = val(Y).
 R = fireable(and, val(false), val(Y)) :- unary(Y) | R = val(false).
 
 R = fireable(or, val(false), val(Y)) :- R = val(Y).
 R = fireable(or, val(true), val(Y)) :- unary(Y) | R = val(true).
 
 R = eval(not(X)) :- R = fireable(not, eval(X)).
 R = fireable(not, val(true)) :- R = val(false).
 R = fireable(not, val(false)) :- R = val(true).
 
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
 
 eval_condition @@
 R = eval(if(B, X, Y)) :- R = fireable(if, eval(B), X, Y).
 
 eval_then @@
 R = fireable(if, val(true), X, Y) :- ground(Y) | R = eval(X).
 
 eval_else @@
 R = fireable(if, val(false), X, Y) :- ground(X) | R = eval(Y).
 
 {'$callback'(zerostep).
 R = X + Y :- R = binop(add, X, Y).
 R = X * Y :- R = binop(mul, X, Y).
 R = X - Y :- R = binop(sub, X, Y).
 
 R = and(X, Y) :- R = binop(and, X, Y).
 R = or(X, Y) :- R = binop(or, X, Y).
 
 R = '='(X, Y) :- R = binop(eq, X, Y).
 R = '\='(X, Y) :- R = binop(neq, X, Y).
 
 R = '<'(X, Y) :- R = binop(lt, X, Y).
 R = '>'(X, Y) :- R = binop(gt, X, Y).
 
 R = '=<'(X, Y) :- R = binop(le, X, Y).
 R = '>='(X, Y) :- R = binop(ge, X, Y).
 }.
```

引き算も追加した。割り算はゼロ除算が発生する可能性があるため、あえて追加していない。

