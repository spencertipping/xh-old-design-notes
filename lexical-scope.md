# Lexical scope implementation
Lexical scope is awkward to implement. It's fairly trivial to identify the
references involved, but issues like rewriting and shadowing complicate the
picture. It gets especially awkward if you have a value that incorporates two
different partially-expanded results of a function call.

These problems are compounded by xh's fexpr model, which allows you to write
things like this:

```
((partial fn x) (get $x 0))     # the identity function
```

This is a big problem. The `$x` in `(get $x 0)` needs to end up being a lexical
reference, but it's possible we'll rewrite it into a journal expression because
its scope isn't evident at read-time. Here's an example where the contradiction
comes up:

```
(def x [5])
(def f ((partial fn x) (get $x 0)))     # (f a) should be 5 (NOPE)
(def g (fn x (get $x 0)))               # (f a) should be a
(def h ((fn x (fn x (get $x 0))) 5))    # (f a) should be a
```

What should `f` be? It could be either `(fn x (get $x 0))` or `(fn x 5)`,
depending on the order of evaluation. This ends up forcing a contradiction if
we want `fn` to be a first-class function:

1. `f` should always evaluate as `(fn x 5)`; `fn` must force its arguments
   fully. (Otherwise we have no way to introduce a reference into the body of a
   function, and all evaluation becomes blocked on invocation.)
2. `g` should always evaluate to the identity function; locals must shadow
   globals.
3. `h` should also evaluate to the identity function; inner locals must shadow
   outer locals.

No consistent evaluation model satisfies these constraints and allows `fn` to
remain first-class.

## `fn` as a quoting construct
The problem arises from the fact that `fn`, like `quote`, is entangled with the
state of knowledge of its arguments. One solution is to forcibly disentangle
them by using separate namespaces for globals and locals, but this is awkward
and doesn't address other cases like shadowing. The underlying issue still
remains: `fn` quotes its arguments, so it interacts with the journal.

Ok, so what if we do this: `fn` always fully evaluates its body argument; that
is, at every moment its body will be as evaluated as possible. This takes care
of shadowing, but not lexical/global conflicts. Maybe worth a thought.

## The real problem: breaking contracts
We can't go and say that `((partial fn x) ...)` isn't the same thing as
`(fn x ...)`; this violates the contract of `partial`. xh must be willing to
treat them the same way, because in every other imaginable situation they would
be.

If we say this, then we're back to the evaluation model above: `fn` must force
its arguments, but globals still shadow locals. The complicating factor is that
in some cases we actually do want global substitution. _If a global exists,
then every form using it will have already been rewritten and the original
lost._ Our hand really is forced here unless we assign lexical rewriting
semantics to `fn`.

## Degrees of freedom in the evaluator
Currently the evaluator is free to expand subforms in any order it would like.
Things like `fn` that guarantee forced-ness do that extra evaluation on their
own. Suppose instead that we constrain evaluation ordering in the following
ways:

1. The head of a list is always evaluated before the tail.
2. Invocation always occurs against _unevaluated_ arguments (problem!).
3. All argument evaluation ultimately happens by native functions.

(2) is a problem for two reasons:

1. `apply` takes an array as its final argument, and non-arrays will need to be
   evaluated. This means that any `fn` invocation happening through `apply`
   will see its arguments in an arbitrarily-evaluated state.
2. Lexical closure becomes impossible because rewriting is how we did that.

We must instead guarantee that every term is always seen in its limit form
(even quotation should work this way)... but that in turn has its own issues.

## Limit forms
The limit form of an expression is any state where you've removed dependencies
on the journal. In particular it means that all relevant journal knowledge has
already been integrated into the expression. It can't mean that the form is
stable under evaluation because some forms never will be. While this doesn't
rule out nonterminating forms, it means that operations over them will have
results that aren't well-defined.

In fact, limit forms can't be asserted to exist specifically because of
nonterminating forms and journal reflection. The closest we can get is to say
that the journal is read-reflective only for definitions; side effects require
constructively-specified and opaque information (which itself could be a
problem if we assert that any value can be expressed as a string).
