# The core issue
Inasmuch as the application of the quotation operator is dependent on
evaluation ordering, the evaluation order will dictate the semantics of
quotation. Put differently, code lacks an intrinsic indicator of whether it is
intended as code or data. code = data is an untyped relationship.

No very good solutions to this problem. We have a few mediocre ones:

1. Retroactively un-evaluate forms in response to late-bound quotation.
2. Modify the notation to indicate when indirect-quoted application is
   intended.
3. Use evaluation order-limiting constructs of some kind. Lisp's function and
   macro are two cases of this.

In some ways Clojure deals with this by providing some data structures which
are evaluatively stable: vectors, keywords, maps, numbers, strings, etc. These
patterns structure evaluation; maybe we can do something similar for xh.
