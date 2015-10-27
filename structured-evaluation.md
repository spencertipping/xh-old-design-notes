# Structured evaluation as a solution to the quotation problem
Let's assume some things:

1. Every code form requires a quoted version of itself that occupies the same
   amount of space, and ideally that looks similar.
2. No quotation form can block all evaluation inside it.
3. There exist two functions, `quote` and `unquote`, which act on quoted code.

`quote` and `unquote` are subject to the following:

```
for all $x:
  (unquote $x) == (unquote (quote (unquote $x)))
  (quote $x) is defined
```
