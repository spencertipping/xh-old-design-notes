# The quotation problem
Quoted values in any form are sometimes difficult to deal with; in particular,
the sense of quotation can be ambiguous. This requires gensyms in some
languages, but xh can't support them. The problem arises like this:

```
((fn x (fn y x)) 'y) = (fn y 'y)        <- if symbols remain quoted
                     = (fn y y)         <- if symbols become unquoted
```

Making dereferencing explicit doesn't solve the problem because it undoes
shadowing:

```
       ((fn x (fn y $x))  y)     = (fn y y)
       ((fn x (fn y $x)) $y)     = (fn y $y)
((fn y ((fn x (fn y $x)) $y)) 5) = (fn y 5)
```

A function always rewrites things, but the question is whether the things it's
rewriting should be treated as opaque objects. Maybe it suffices to
partially-order the _evaluator_ in addition to the side-effect chain. The only
syntax necessary should be something like derivative-notation:

```
((fn' x (fn y $x)) $y) = (fn y $y)
((fn  x (fn y $x)) $y) = (fn _ $y)
```

So maybe functions need to be their own syntactic element? In theory that works
around the list parsing issue ... but adds a new layer of complexity for
macroexpanders. This may be ok, particularly if it's possible to create
function objects using list calls (just like the `cons` function, we could have
a `fn` function).

This isn't quite good enough because we need functions to modify the quotation
level of other functions; what is _their_ quotation level?

```
(quotify   (fn  x $x)) = (fn' x $x)
(unquotify (fn' x $x)) = (fn  x $x)
```

Functions are always stable under evaluation, though, so the context of a
`quotify` or `unquotify` call is unambiguous. The real problem is that its
moment of application isn't:

```
((fn' x (quotify (fn y $x))) $y) = (fn' _ $y)
```

This retroactive re-quotifying divorces semantics from evaluation order, which
I think is what we want here. Can we abstract over `quotify`?

```
(((fn' f (fn' x (f (fn y $x)))) quotify) $y) = (fn' _ $y)
```

**Does this create the potential for space leaks, since any given function
  might end up in any state of quoted-ness?** It actually does worse than that.
Because a quotified function must "remember" its less-quoted forms,
serialization will break requoting. That is, the semantics of quoting are in
general lost across the serialization barrier.

Ok, so we know the following at this point:

1. Variable scopes must be determined either at read-time (typical lexical
   scoping implementation), or through some type of rewriting stage that
   behaves correctly with shadowed quantities.
2. All scoping configurations must have serialized forms that re-evaluate to
   them. Inner functions must be accessible in two ways: for uninvoked outers
   closure variables are underspecified terms, and for invoked outers the
   values themselves must be rewritten.

I suspect that both shadowing and lack of shadowing end up causing problems. We
must shadow in situations like this:

```
f = (fn  x $x)
g = (fn' x $f)
```

The reason is that otherwise expressions inside `f` are forced to keep track of
their unevaluated state, which violates the serialization property.
