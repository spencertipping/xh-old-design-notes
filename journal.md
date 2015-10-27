# Execution journaling
Evaluation in xh is always pure; there are no side effects. The only way to
modify the state of the rest of the world is by adding an element to the
journal, an increasingly-defined list of state modifications. All global
variable references are rewritten to refer to the journal, often in a
head-losing way (so early effects can be garbage-collected).

For example, a reference to the global variable "foo" might be encoded like
this:

```ml
let rec resolve name = function
  | Def (k, v) :: _ when name = k -> v
  | _ :: xs                       -> resolve name xs in
resolve "foo" journal
```

`resolve` will descend through the list immediately, either returning a
definition (at which point it can be removed by evaluation) or holding a
reference only to the end of the journal. This journal end is a cons cell with
an unrealized head and tail; as a result, any expression (like the match above)
will block until it becomes defined by a corresponding side effect.

## Committing to the journal
Because evaluation happens independently of side effect ordering, order of side
effects must instead be achieved using value dependencies. Specifically, the
constructors that represent state transitions require a prior side effect
object; this dependency forces ordering. For example:

```
# beginning of program: journal = (cons j0 ...)
(def j1 (posix.rm j0 "foo"))
(def j2 (posix.rm j1 "bar"))
```

Sometimes you don't want to force an ordering because it doesn't matter. In the
example above, for example, you usually don't care about file deletion order.
If that's the case, you can depend on an earlier journal entry:

```
(def j11 (posix.rm j0 "foo"))
(def j12 (posix.rm j0 "bar"))
(def j2  (max j1 j2))
```

Notice that if we want some other side effect to block on both unordered side
effects, we need to recombine them into a consistent timeline at some point.
This is done using `max`, which returns whichever occurred later once their
ordering is established (and blocks until it knows).

## Quotation
Lisp-style quotation is entangled with the interpreter's state of knowledge,
which in turn depends on the side effects that have happened. As a result, the
`quote` function takes a journal state as input; this constrains its evaluation
to sometime after the journal contains a given commit.

In this sense, `quote` is intended to mean, "quote X with respect to Y" rather
than "quote X" -- this removes the need for constructs like Common Lisp's
quasiquote operator, since xh allows you to control the evaluation ordering
more precisely.

## Side-effecting definition
`def` commits a new entry to the journal upon evaluation, provided that its
first argument is a literal. This usually happens shortly after a program is
parsed and before any other side effects are committed, but it can also happen
later. For example:

```
(def foo $bar)          # pushes Def (foo, $bar) definition
(def bar $bif)          # pushes Def (bar, $bif) definition, resolving $bar
```

This definition mechanism has the advantage that it allows globals to be
garbage-collected, since everything ends up being rewritten into calls against
the journal.

## Program garbage collection
Because the program is evaluated bottom-up, the only live object is the
unrealized pseudo-cons cell at the end of the journal. Anything not depending
on this is provably immutable, which means that any side effects they will
commit are reachable strictly by evaluation, which can be delayed arbitrarily
(since side effects are only partially ordered until they happen). Once a form
has no references to either the journal tail or the journal commit function, it
no longer has any impact and can be deleted.

## Transactions, distribution, and locking
Distribution can happen transparently. Transactions and locking are non-issues
because all side effects happen constructively; an expression must hold a
reference to something that's been committed in order to itself commit
something. This makes xh a CP system. It can be made into an AP system by
referring to the `min` of two remote journal entries:

```
(def j11 (some-remote-effect j0))
(def j12 (another-remote-effect j0))
(def j2 (min j11 j12))          # whichever happens first
```

This allows you to proceed as soon as either machine acknowledges the effect,
which introduces the potential for inconsistency (i.e. disagreements about the
true order of effects) but allows for node failure.
