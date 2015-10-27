# Reordering of parameter bindings
A variable dependency forces ordering:

```
(let [x 5]
  (let [y x]
    ...))
```

But in the absence of such dependency, we're at liberty to reorder things
arbitrarily:

```
(let [x 5]            (let [y 6]
  (let [y 6]     ==     (let [x 5]
    ...))                 ...))
```

This suggests multiplication should be commutative:

```
∫ f(x, y) x' y' dt = ∫ f(x, y) y' x' dt
```

In the absence of directional multiplication, what does this say about the
relationship between the chain rule and the order of operations? Maybe nothing;
all we're really concerned about is the _side effects_, not the flow of
information. It's possible this all works out.

TODO: tie this in with differential reordering in general. I think I'm missing
something here.
