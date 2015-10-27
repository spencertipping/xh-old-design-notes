# Macros and evaluation
Macros lift an evaluative context over quoted code. Lisp draws a line between
this context and the one that executes your code, but there's no particular
principle that demands this. Given the notational similarity, macros and
functions should have as similar contexts as possible.

In practice I don't think this is a big deal. It means macros are free to
predicate things on runtime values without quoting that predication. It's then
up to the runtime to figure out how to optimize the result.

This approach might have deeper consequences. The mechanism of action here is
unquoting, so a macro will refer to `$x` to mean, "the runtime value of x" --
presumably from x's surrounding scope. But this conflicts with the more obvious
definition, "the value of x within the body of the macro." The possibility of
this conflict requires us to forcibly separate the two namespaces somehow (or
to take Lisp's approach and quote any interaction).

The underlying issue we're trying to fix is just that the code to generate code
doesn't really look like the code being generated. This is jointly a matter of
pattern matching and of templating, though arguably more the former than the
latter. Macros aren't at liberty to execute things arbitrarily, though; that
unquoting eliminates the possibility of macro composition.

In xh it seems artificial to draw this line between functions and macros.
There's not a strong preference between a quasiquoted thing and an unrealized
expression -- in practice they behave the same until you try to inspect them.
Only the next macro along will notice the difference, since it will try to
inspect the code (most likely).

Do we even want this level of control over this stuff? I'm not remotely
confident it's a feature.
