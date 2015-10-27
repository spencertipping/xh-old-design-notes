# Old xh design notes
These are from when I was trying to write a Lisp that evaluated in an
unspecified order (guided by term realization), and used a global side-effect
journal value rather than allowing evaluation itself to issue side effects. I
still think this language is possible to write, though `delay` is trickier than
it looks here and you can't serialize the journal future as I've implied.
