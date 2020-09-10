# まとめ



今まで断片的に記述してきたものを全てまとめた。

```text
% 1 + 2 * 3
//test = eval(binop(add, 1, binop(mul, 2, 3)),[]).

% (not false) && (false || true) --> true 
//test = eval(and(not(false), or(false, true)),[]).        

% if 1 < 2 then 3 else 4 --> 3
// test = eval(if(1 < 2, 3, 4), []). 

% let x = 1 in let x = x + 2 in x + 3 --> 6
// test = eval(let("x", 1, let("x", "x" + 2, "x" + 3)), []). 

% let f = fun x -> x + 1 in f 2 --> 3
// test = eval(let("f", fun("x", "x" + 1), app("f", 2)), []).
 
% let rec fib n = 
% 	if n = 0 then 1 
%		else n * fib (n-1) 
% in fib 5
% -> 120
test = eval(
 let_rec("fib", "n", 
  if("n" =< 0, 1, 
   "n" * app("fib", "n" - 1)),
 app("fib", 5)), 
 []).
 
eval_left_child @@
 R = eval(binop(OP, X, Y), Env) :-
 ground(Env) |
 R = almost_fireable(OP, eval(X, Env), Y, Env).
 
eval_right_child @@
 R = almost_fireable(OP, val(X), Y, Env) :-
 R = fireable(OP, val(X), eval(Y, Env)).
 
eval_value @@
 R = eval(Int, Env) :- int(Int), ground(Env) | R = val(Int).
 R = eval(true, Env) :- ground(Env) | R = val(true).
 R = eval(false, Env) :- ground(Env) | R = val(false).
 
 R = fireable(add, val(X), val(Y)) :- Z = X + Y | R = val(Z).
 R = fireable(mul, val(X), val(Y)) :- Z = X * Y | R = val(Z).
 R = fireable(sub, val(X), val(Y)) :- Z = X - Y | R = val(Z).
 
 R = fireable(and, val(true), val(Y)) :- R = val(Y).
 R = fireable(and, val(false), val(Y)) :- unary(Y) | R = val(false).
 
 R = fireable(or, val(false), val(Y)) :- R = val(Y).
 R = fireable(or, val(true), val(Y)) :- unary(Y) | R = val(true).
 
 R = eval(not(X), Env) :- R = fireable(not, eval(X, Env)).
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
 
eval_let_val @@
 R = eval(let(Var, Val, Body), Env) :- 
 ground(Env) |
 R = fireable(let, Var, eval(Val, Env), Body, Env). 
 
eval_let_body @@
 R = fireable(let, Var, val(Val), Body, Env) :- 
 R = eval(Body, [let(Var, val(Val))|Env]).
 
resolve_val @@
 R = eval(Var1, [let(Var2, Val)|T]) :- 
 string(Var1), Var1 == Var2, ground(T) |
 R = Val. 
 
lookup_val @@
 R = eval(Var1, [let(Var2, Val)|T]) :- 
 string(Var1), Var1 \= Var2, ground(Val) |
 R = eval(Var1, T). 
 
eval_fun @@
 R = eval(fun(Var, Body), Env) :- 
 string(Var) |
 R = val(fun(Var, Body, Env)). 
 
eval_let_rec @@
 R = eval(let_rec(FN, Var, FB, Body), Env) :- 
 string(Var), ground(Env), string(FN) |
 R = eval(Body, [let(FN, val(rec_fun(FN, Var, FB, Env)))|Env]).  
 
app_fun @@
 R = fireable(app, val(fun(Var, Body, Env)), val(Val)) :- 
 R = eval(Body, [let(Var, val(Val))|Env]).  
 
app_rec_fun @@
 R = fireable(app, val(rec_fun(FN, Var, Body, Env)), val(Val)) :- 
 ground(Env), ground(Body), string(FN), string(Var) |
 R = eval(Body, [let(Var, val(Val)), 
			let(FN, val(rec_fun(FN, Var, Body, Env)))|Env]). 
 
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
 
 R = app(X, Y) :- R = binop(app, X, Y).
 
 R = fun([Var|T],  Body) :- 
 R = fun(Var, fun(T, Body)).
 
 R = fun([],  Body) :- 
 R = Body. 
 
 R = let(FN, FV, FB, Body) :- 
 R = let(FN, fun(FV, FB), Body).
 
 R = let_rec(FN, [Var|T], FB, Body) :- 
 R = let_rec(FN, Var, fun(T, FB), Body). 
}.
```

