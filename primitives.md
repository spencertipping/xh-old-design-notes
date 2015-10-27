# Primitive types
- Function (fn body is accessible through introspection)
- Function args reference (De Bruijn index)
- Delayed value (introspectable)
- List
- Symbol (supports character-level introspection)
- Variable reference

## Functions
The primitive representation of functions is in terms of De Bruijn indexes:

```
(fn x $x)        -> fn(@0)
(fn x (fn y $x)) -> fn(fn(@1))
```

Note the different syntax: references to function arguments are not related to
variable dereferencing. If they were, then it would be possible to introduce a
new reference after the function were compiled, forcing additional live
references where they most likely aren't needed.

Argument references are self-evaluating if treated as values; this makes it
possible to construct them and compile arbitrary functions.
