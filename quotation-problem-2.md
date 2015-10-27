# Quotation problem (second try)
Suppose every quoting construct is gated on the definedness of something. For
example, functions really only make sense when the formal is defined. We can
add an argument to `quote` to delay quotation until realized. Does that solve
the problem?

I'm not yet sure it does. Part of the problem has to do with the order in which
the evaluator _chooses_ to realize terms; if we did things this way, we'd need
limit expansion. For example:

```
((fn x (fn y $x)) $y) = ((fn x λy.$x) $y)
                      = (λx.λy.$x $y)
                      = λy.$x | x = $y
```

I think this is the right interpretation. The symbol `y` was realized, so no
macro-stuff could happen at that point. If we wanted the inner function to
become the identity, we'd need to delay the evaluation of its formal like this:

```
((fn x (fn (delay $x y) $x)) $y) = (λx.(fn (delay $x y) $x) $y)
                                 = (fn y $y)
                                 = λy.$y
```

This requires `delay` to be a special form, but I think that's ok.
