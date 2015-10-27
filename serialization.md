# Issues created by serialization
One of the design principles of xh is that any value can be converted to a
string that fully represents it. The exception is machine-local mutable state,
which can also be serialized as a reference that the machine will recognize
modulo various RPC negotiations (like a missed heartbeat signal, which may
invalidate the reference).

This ends up being a big deal for a few reasons:

1. What happens if you try to serialize the unrealized journal tail?
2. The GC is obligated to find all expressions in any way dependent on journal
   values and preserve them, since any of them could be used to create another
   side effect (through reflection).
3. As a joint consequence of (2) and incompleteness, some pathological forms
   will never be garbage-collected. Not actually sure how bad this is, since xh
   can at least swap them out to disk or something.
