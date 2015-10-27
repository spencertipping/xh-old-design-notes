# Some examples of the integral transform
```
(fn x (fn y x)) = ∫ x dy dx
```

This clearly doesn't work. Infinitesimals are the wrong abstraction.

Well, possibly. We've got dα/dt terms floating around, and the integral is
implicitly with respect to t. This might work after all, since `(fn y x)`
obviously can't introduce any new side-effects.
