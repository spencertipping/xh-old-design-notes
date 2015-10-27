# Hello world
Basic IO happens as a function of `t`, which the program's main function takes
as an argument. The initial state `t0` contains the system-provided argv and
env:

```
main t = (-> t
             (print "hello world\n")
             (exit 0))
```

The threading forces side effect ordering; as a result the following code
produces the same output:

```
main t = (-> t
             (print "hello ")
             (print "world\n")
             (exit 0))
```
