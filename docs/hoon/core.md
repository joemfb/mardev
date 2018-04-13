---
title: Approaching the core
navhome: /hoon
sort: 5
next: true
---

### Approaching the core

In most (every?) functional programming language, functions are simply
axiomatic -- it's right there in the adjective. But we're well aware by now
that Hoon is not like other languages; neither gates (anonymous functions) nor
their calling constructs are *natural runes*, rather, they are synthetic
compiler macros which implement function-like conventions around a *core*.
Cores are Hoon's primitive capacity for combining code with data;

they're the foundation of gates, and many other patterns in Hoon. Before we look at them generally, let's review the nature of functions in light of the natural capabilies of Hoon.

TODO: reword last ^

#### functions

Mathematically speaking, a function is simply the relation between an input
and an output. Programming languages are both more and less precise.
Informally, a function in a programming language is just a self-contained
piece of code. Stateful languages have a host of related concepts (methods, sub-routines, etc.), but we're only concerned with pure, stateless functions
that return or *produce* a value. While we tend to think of the arguments
to a function as being its input, this is imprecise; a zero-argument
mathematical "function" would just be a constant, but is perfectly normal when
programming. This is because the input to a function is not merely its
arguments, but also the global environment and any other symbols that are in
scope when it's defined.

So we can tighten up our definition a little: a function is a self-contained
piece of code, computed against its arguments and the scope or context in
which it was defined.

> The last part is important! We want to define a function in one context and
> reliably use it in another, i.e., lexical scope. (And, yes, dynamic scope is
> a thing: you could even emulate it in Hoon by surgically manipulating cores.
> But should you really want to?

#### subject-oriented programming

So what does all this mean with regards to Hoon?

*coming soon*

TODO: subject oriented programming

```
> (add 3 2)
5
> (mul 3 2)
6
> (div 3 2)
1
> (sub 3 2)
1
> (gth 3 2)
%.y
```

TODO: faces in spans

late binding
mutual recursion

TODO: code and data

core wraps subject

core is it's own subject

example

trap

loop

gate

```
> .*([-:dec 10 +>.dec] -:dec)
9
> .*([-:dec 10 0] -:dec)
9
> .*([-:dec 100 0] -:dec)
99
> .*([-:add [10 2] +>:add] -:add)
12
> .*([-:add [10 2] 0] -:add)
```

jets, ~fodwyt-ragful http://urbit.org/~~/fora/posts/~2016.7.29..17.25.38..95fb~/