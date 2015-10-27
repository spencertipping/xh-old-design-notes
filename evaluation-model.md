# xh evaluation model
xh programs cannot be efficiently executed top-down since the evaluation order
of subexpressions is unknown. Instead, programs must be evaluated bottom-up;
for example:

```
# top-down evaluation
(eval '(if (= 4 $x) 'a 'b)) -> (if (eval '(= 4 $x)) 'a 'b)
                            -> (if false 'a 'b)
                            -> (eval 'b)

# bottom-up evaluation
(if (= 4 $x) 'a 'b) -> {$x       [(= 4 $x)]
                        (= 4 $x) [(if (= 4 $x) 'a 'b)]}
```

The difference is the graph ordering: top-down evaluations require graphs with
child pointers, while bottom-up use pointers from children to parents. The
bottom-up model is useful because it provides flexible scheduling and the
ability to exhaustively propagate knowledge; the latter is important for
garbage collection.
