The concept of a cask is worth exploring in more detail. A noun is an atom
or a cell (a binary tree of nouns). Every noun has a `span`, an inferred type
which can range from infinite (`*`, any noun) to constant (ex: `$foo`, the
atomic symbol `%foo`). While that's vital information within a Hoon expression,
it's not necessarily for (or even available to) a network request or syscall.
Both are asynchronous computations, with necessarily *separate* execution
contexts. And even if the original span is available to both contexts, it's
effectively overspecified for the task of naming what type of data we have. In
such cases, a *mark* is used to provide a formal description of the noun
in question, much like a MIME type. `++mark` is a symbolic reference to that
formal description (itself a Hoon file in `/mar` containing parse/diff/transform/etc. implementations). `++cask` is a high-order mold, analogous to a
generic type in other languages.

`[%noun 0]` is a basic cask. We can print its *span* in `:dojo` using `?`:

```
your-urbit:dojo> ? [%noun 0]
  {$noun @ud}
[%noun 0]
```

We have a *mold* matching a `cell`, with a constant head and an
unsigned-integer tail.

Compare to the span once cast to `++cask`:

```
~your-urbit:dojo> ? ^-((cask *) [%noun 0])
  {p/@tas q/*}
[p=%noun q=0]
```

A `++cask` is a pair, so it applies standard p/q faces. Its head can be any
symbol, and in our case, its tail can be any noun.

Let's try an atomic `++cask`:

```
~your-urbit:dojo> ? [%atom 0]
  {$atom @ud}
[%atom 0]
~your-urbit:dojo> ? ^-((cask @) [%atom 0])
  {p/@tas q/@}
[p=%atom q=0]
```

TODO: overspecified, vase





Hopefully this illuminates the distinction between *span* and *mark*...
