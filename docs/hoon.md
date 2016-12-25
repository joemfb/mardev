---
navhome: docs/hoon
title: hoon
sort: 1
---

### hoon

Hoon is one of many true innovations in Urbit: a new, strict, staticly-typed
functional programming language, conceptually and practically independent of
the tradition of lambda calculus, with a host of distinguishing features.

Idiomatic hoon is dominated by its massive suite of digraph ASCII *runes*, yet
still intelligible due to its highly regular syntactic structures. Its only
intrinsic data structures are *atoms* (unbounded unsigned integers) and
*cells* (binary trees of *nouns*: either *atoms* or *cells*), but its rich type
inference easily supports custom structures of arbitrary complexity.

And, strangest of all, Hoon itself is entirely *computationally infeasible*!
The standard-library decrement algorithm is `O(n)`, and most of the other basic
arithmetic is implemented on top of it -- it only works (at all!) because of
the counterintuitive performance strategy (jets) of Urbit's custom
virtual-machine (Nock).

In spite of all this -- or maybe because if it, Hoon is a (mostly)
straightforward and simple language. However, the learning curve is
unquestionably steep. We'll start with a review of the Hoon reference docs,
then dig deeper into some key concepts and data structures.

<list/>