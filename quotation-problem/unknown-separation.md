# Quotation as separation of unknowns
If we have something like `(quote (f $x))`, the purpose of the `quote` operator
is to prevent `(f $x)` from being rewritten into a functional result. So we're
reintroducing `f` and `x` as unknowns.

Now suppose we have a function closure designed to re-scope a set of variables:

```
(let [fn-template '(fn y $x)
      outer       '(fn x)]
  (eval (append outer [fn-template])))
```

Here, the `$x` in `fn-template` is intended to be an unknown; it can be given a
value only by tying it into some lexical scope somewhere. This, in turn, is
done internally by the `fn` construct.

Another way to say this is that `fn` quotes with respect to a defined set of
variables, whereas `quote` undefines all of them. It's probably possible to
come up with macro-definitions for both of these things in terms of lower-level
evaluation primitives -- though if shadowing is going to work, it means that
inner evaluation blocks on outer. That's a problem.

If we don't base `fn`, at least, on lower-level primitives, then we have
another problem in that destructuring binds must be evaluated at compile-time.

## Proof of quotation problem
Suppose we have something that looks like this:

```
b = (fn [a x] (a fn [x] $x))
((b (fn [f x y] (f $x $y)) 5) 6)        # what should this be?
```

Two possible evaluations:

```
((fn [f x y] (f $x $y)) fn [x] 5)       # eager argument substitution
= ... = (fn [x] 5)

((fn [f x y] (f $x $y)) fn [x] $x)      # delayed argument substitution
= ... = (fn [x] $x)
```

This means that the quotation happening by `fn` is entangled with the order of
evaluation; _evaluation does not preserve the semantics of expressions_.
