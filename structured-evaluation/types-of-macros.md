# Types of macros
Macros serve a few main purposes in a language like Clojure:

1. Notational convenience -- this is where most of the application-level
   leverage happens.
2. Serialization, e.g. pigpen -- generally considered a misuse of the language
   because macro-quoted forms won't have let-bindings dereferenced properly.
3. Semantic modification -- also generally considered a misuse of the language
   unless you're specifically defining a DSL (and usually then also).
4. Reflection -- fully addressable by an appropriately introspective runtime.

Arguably (2) is really a special case of (4).

Workarounds in other languages: OCaml has campl4, C/C++ has cpp, Java has
nothing, Haskell has nothing. It's a recognized problem, but not egregious
enough to force a language designer to consider it (and arguably Haskell's
abstraction is good enough that it doesn't matter too much).

(1) happens because of quoting, not because of macros per se. The macro
structure is just an expedient to support better compilation; with an
appropriately aggressive evaluation model, macros can be approximated by
functions that call `eval` and take quoted arguments.

(3) is generally considered a bad idea, but enables some interesting ideas like
Cascalog. The problem here is modifications don't tend to compose well; you
could have something like Clojure's `for` loop that fails to leverage `pmap` or
some other asynchronous model. In some sense we want a type system for this
stuff, at least so the conversions can be specified more formally.

(3) also feels like a unit-specificity problem. Numbers are untyped in the
unitary sense, but a tag-based type system would probably solve that. This
raises a whole host of questions though: are types reified? They must be. But
no logic can diverge based on the type of something, since otherwise it could
implicitly be converted prior to introspection. It's the quotation problem all
over again unless all logic involves explicit conditioning on the type.

Let's look at this just a bit differently. Let's suppose for a moment that the
entire program _describes_ a process, and that we have ML-style union types
that comprise the list heads that make up special forms in each of several
macro layers. Then lists are nested tag structures -- but in particular, we can
look at a list and figure out which macro consumes it. (Unless the macros
themselves consume arbitrary head-forms, which is easily possible.)

This isn't quite right. Macros are fine being untyped, particularly if we have
good introspection. The real issue is that it's sometimes difficult to specify
what is happening to what code. Maybe we should have some trivial IoC thing
ship with the default library to support code-walking insertion points. (Also
good because we then solidify this "code = data by default" idea.) Then
macroexpansion isn't a property of invocation, it's a property of the language
you're constructing. You extend the compiler.

I think this has a serious chance of working. It's trivial to build
general-purpose macros on top of this, so the usual use cases end up working
out as normal. It's also trivial to define a more situation-specific macro
language, which would probably address some pattern-matching deficiencies in
Clojure and Lisp's `defmacro`.

Do we still assert core semantics the same way? Universal evaluation etc? Maybe
the core layer is lower-level than that; in some sense the language almost
doesn't need a core layer at all. If the compiler is self-hosting, we can
parameterize the code generator by the host architecture and code straight to
the metal (or other suitable compilation target). That's an appealing way to do
it, particularly because it opens up the possibility of replacing or bypassing
the runtime in certain cases. Otherwise that stuff really is walled off, which
is appropriate only in the vast majority of cases.

All of this sounds reasonable enough in theory, but does it fix the quotation
problem in practice? At best it sounds like we have a much clunkier syntax with
all of the same problems.
