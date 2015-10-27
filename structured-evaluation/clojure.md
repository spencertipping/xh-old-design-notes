# Everything wrong with Clojure's quasiquotation mechanism
Clojure has it mostly right for trivial macro cases, but complex macros quickly
devolve into chaos as the structure of the generated code diverges from the
code that generates it. Piecewise destructuring functions would probably help,
and things like splicing-interpolation could be generalized more nicely.

The other thing is that data structures and semantics begin crossing in ugly
ways. `(conj [x] y)` isn't the same result as `(conj (list x) y)`, so you end
up needing to specify the type of collection _separately from the process you
used to construct it_.

Clojure does weird things with namespaces, and I'm not particularly convinced
that its strategy always makes sense. The most noticeable form of this is the
gensym-weirdness that makes it awkward to introduce shadowing variables. All
things considered, this isn't particularly egregious.

Clojure's data structures are a minor annoyance when walking through code. The
language is unhelpful about this and clearly wasn't intended to be used this
way. It would be nice to have an abstract evaluator that could be used to
reverse-engineer the language, perhaps. The evaluator should be a first-class
entity rather than something that always creates side-effects. It seems like
the language should also support some way to describe local scopes.

Put differently, _you should be able to find out anything the compiler knows_.
Clojure is not very good at this.
