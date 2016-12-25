---
title: Mold it again, Sam
navhome: /hoon
sort: 3
next: true
---

### Mold it again, Sam

**_Molds: Part 2_**

*[Read part 1 first](/docs/hoon/mold).*

#### span, once more, with feeling

We know that a *span* is just a set of nouns, inferred by the compiler in its
analysis of an expression. We know that we can cast a given noun to the span of
an example noun. And we've been exploring *molds* as a standard approach for
producing example nouns. But we may have missed some of the implications.

Recall that noun *A* can only be cast to the span of noun *B* if the spans
*nest*, i.e., if the span of *A* is provably a subset of the span of *B*
(and `nest-fail` means we can't prove that they nest). A type-cast can either leave the span of *A* unchanged (identical spans also nest), or broaden it to
the span of *B*. In other words, a successful type-cast *throws away* type
information.

That bears repeating: in Hoon, a type-cast works by *throwing away* type
information! And it has nothing to do with the noun itself -- it's purely a
comparison of (and operation on) the *span* of the noun.

`ream` parses *any* Hoon expression, and produces any twig. The span of it's
product is a *fork* with many different branches: one for each type of twig.
And we're trying to cast it to the mold of a specific twig -- we're trying to
narrow the span with a cast, so of course we have failed.

> My thanks to `~fodwyt-ragful` for leading me to enlightenment!

To make this concrete, we'll throw away all our type information by casting to
the broadest possible span: `*`, any noun. Once we've done so, it's impossible
to cast with a more specific mold.

```
> ^-(* 0)
0
> ^-(@ ^-(* 0))
nest-fail
> `*`[0 0]
[0 0]
> `^``*`[0 0]
nest-fail
```

> We've also introduced the irregular form of `:cast`: `` `a`b`` is `^-(a b)`.

We know that `0` is an atom and `[0 0]` is a cell, but the compiler doesn't.
We've told it that they could be any noun at all, and it believes us.

Narrowing the span (by adding more specific type information) is obviously a
very useful operation, but, to do so, we have to prove to the compiler that we
actually have more specific information than what is already in the span. To
do that, we'll need to inspect the noun itself.

We can inspect a noun with `:fits`, the pattern-matching twig that we used
earlier. It ignores the span entirely, simply telling us whether a mold matches
a noun:

```
> ?=(@ 0)
%.y
> ?=(@ `*`0)
%.y
> ?=(^ [0 0])
%.y
> ?=(^ `*`[0 0])
%.y
```

This tells us whether the mold fits the noun, but we want to use that
information. *If* the mold fits, we want to cast the noun. So we need the
conditional twig: [`:if` or `?:`]().

We also need to refer to the same noun multiple times (in the `:fits` and the
`:cast`), so we put a face on it and add it to the subject:

```
> =+(a=`*`[0 0] ?:(?=(^ a) `^`a %nope))
[0 0]
```

This is a pretty gnarly looking expression! We've cast our cell to `*` and put
the label `a` in the span. Then we've pushed that onto the subject (with
[`:pin` or `=+`]()) and evaluated our conditional: if `a` is a cell, produce
it as a cell, otherwise produce the symbol `%nope`.

Let's try again in tall form:

```
> =+  a=`*`[0 0]
  ?:  ?=(^ a)
    `^`a
  %nope
[0 0]
```

Better! But we really don't care about the non-matching case -- we just want to
prove that `a` is a cell. So we can just crash with `!!` if it's not:

```
> =+  a=`*`[0 0]
  ?:(?=(^ a) `^`a !!)
[0 0]
```

And, in fact, there's a twig for that: [`:sure` or `?>`]() is a positive
assertion:

```
> =+  a=`*`[0 0]
  ?>(?=(^ a) `^`a)
[0 0]
```

As it turns out, our inner cast is completely irrelevant. The compiler performs
type inference over pattern-matches in conditional expressions and adjusts the
span accordingly (this is called branch specialization in the official docs).

We can also combine our `:pin`, `:name`, and `:cast` expressions with
[`:var` or `=/`]():

```
> =/  a/*  [0 0]
  ?>(?=(^ a) a)
[0 0]
```

It'd be nice to *see* that our span is now actually `^`; after all, `[0 0]`
looks the same regardless of the span. We can tell the `:dojo` to print the span of our product by prefacing our expression with `?`:

```
> ? =/(a/* [0 0] ?>(?=(^ a) a))
  {* *}
[0 0]
> ? `*`[0 0]
  *
[0 0]
```

> Spans have their own pretty-print format -- it's usually mold-like, except
> for some well-known spans which have their own conventions. For now, just
> remember that it's *presentational*, and not necessarily syntactically valid.

Returning to our twig mold example, we now know what we need: in order to
narrow the span from any twig to a specific one, we need to inspect the noun
itself:

```
> =+  a=(ream ':-(1 2)')
  ?>  ?=({$cons *} a)
  ^-({$cons p/twig q/twig} a)
[%cons p=[%sand p=%ud q=1] q=[%sand p=%ud q=2]]
```

As before, the final cast is irrelevant: there's only one twig with a head
that matches `$cons`.

```
> =+  a=(ream ':-(1 2)')
  ?>(?=({$cons *} a) a)
[%cons p=[%sand p=%ud q=1] q=[%sand p=%ud q=2]]
```

(You can check the span yourself with `?` -- it's too long to reproduce here.)

Pattern-matching has a very important limitation: it won't match through
recursive molds. If, `:fits` is given a noun that matches a recursive branch
in a mold, it will crash with `fish-loop`:

```
> `twig`(ream ':-(1 2)')
[%cons p=[%sand p=%ud q=1] q=[%sand p=%ud q=2]]
> ?=(twig (ream ':-(1 2)'))
-axis.5
fish-loop
```

We can see that recursive spans like `twig` are no obstacle to the compiler:
in fact, they're used quite frequently, and there are many recursive molds.
But `:fits` is not the same operation as `nest`; it ignores the span and
inspects the noun itself, and recursive inspections could have unbounded
complexity. Instead, we use a minimal mold to pattern-match -- we only need
to distinguish between branches and the compiler will infer the rest.

#### call me, maybe

Pattern-matching isn't the only way to add type information. After all, we do
have these molds lying around -- and they're gates. We can just call them!

```
> `^``*`[0 0]
nest-fail
> (^ `*`[0 0])
[0 0]
> ? (^ `*`[0 0])
  {* *}
[0 0]
```

This is the quick and dirty workaround that we mentioned earlier:

```
> ^-({$cons p/twig q/twig} (ream ':-(1 2)'))
nest-fail
> ({$cons p/twig q/twig} (ream ':-(1 2)'))
[%cons p=[%sand p=%ud q=1] q=[%sand p=%ud q=2]]
```

When we call a mold, we're throwing away our type information (by casting to
`*`, the sample mold for every mold gate) and then *molding* the sample into a
specific shape. This is the first reason it's ill-advised to call a mold
directly: we never want to unnecessarily discard the span.

But that's not the only reason! A mold always produces the same span -- if it
can't *normalize* a noun, it *bunts*. Calling a mold can produce something entirely unrelated to the sample. What's more, compound molds can partially
normalize and partially bunt, producing a sort of franken-noun:

```
> `{^ @}`[[0 1] 2]
[[0 1] 2]
> ({^ @} [[0 1] 2])
[[0 1] 2]
> `{^ @}`[[0 1] [2 3]]
nest-fail
> ({^ @} [[0 1] [2 3]])
[[0 1] 0]
```

`[2 3]` clearly doesn't nest within the icon of `@`; when we call `@` with an incompatible sample, it bunts and produces `0`:

```
> (@ [2 3])
0
```

We can guard against partially-normalized nouns by comparing the mold sample
to the normalized product:

```
> =+  a=[[0 1] [2 3]]
  =+  b=({^ @} a)
  ?>(=(`*`a `*`b) b)
(crash)
> =+  a=[[0 1] 2]
  =+  b=({^ @} a)
  ?>(=(`*`a `*`b) b)
[[0 1] 2]
```

> We're casting `a` and `b` to `*` in the equivalence test because we just
> want to compare the nouns themselves, not consider their spans.
> 
> Also, we're using the irregular syntax for `:same`: `=(a b)` is `.=(a b)`.

And in fact, there's a twig for this exact pattern! [`:fry` or `;;`]() calls a
gate and asserts that it's a *fixpoint*: that its sample equals its product.
So it's a great fit for molds:

```
> ;;({^ @} [[0 1] 2])
[[0 1] 2]
> ;;({^ @} [[0 1] [2 3]])
(crash)
```

You can also wrap a mold in a fixpoint assertion with `hard`:

```
> ((hard {^ @}) [[0 1] 2])
[[0 1] 2]
> ((hard {^ @}) [[0 1] [2 3]])
%hard
(crash)
```

Or test if a mold is a fixpoint with `soft`:

```
> ((soft {^ @}) [[0 1] 2])
[~ [[0 1] 2]]
> ((soft {^ @}) [[0 1] [2 3]])
~
```

> A `soft` mold produces a `unit`, Hoon's approach to a nullable type: null is
> null, and not-null is a null-prefixed cell. This lets us introduce
> nullability explicitly, and only where we want it.

So it's clear that calling a mold is a pretty blunt instrument -- you'll
almost never need to do it, and you'll know if you do. (At the same time, it's
a handy tool for exploratory programming.)

#### fork molds

As we said earlier, `twig` is a mold for normalizing *any* twig; it's span is
a *fork* representing a choice between its many *branches*. Spans are always produced by inference, so lets see how we can lead the compiler to infer a
fork.

A conditional expression is the obvious starting place:

```
> ?:(.=(1 1) %a %b)
%a
> ?:(.=(1 2) %a %b)
%b
> ? ?:(.=(1 1) %a %b)
  ?($a $b)
%a
> ? ?:(.=(1 2) %a %b)
  ?($a $b)
%b
```

The compiler can't infer which branch we'll take (equality tests aren't
inferred and neither branch references the subject), so the span is a fork
between our two constant symbols.

If we wrap that expression in a gate, we can use it to bunt, cast, or
pattern-match. We'll put our gate in a `:dojo` variable for readability:

```
> =foo |=(* ?:(=(1 1) %a %b))
> *foo
%a
> ^-(foo %a)
%a
> ^-(foo %b)
%b
> ^-(foo %c)
nest-fail
> ?=(foo %b)
%.y
> ?=(foo %c)
%.n
```

> Note: we can clear that `:dojo` variable by evaluating `=foo`.

Building a fork manually is another peek behind the curtain -- it's type
inference all the way down! You can't stop it; you can only hope to channel it.

Of course, we use molds to channel it in practice, instead of writing explicit
gates. There are four mold twigs for creating forks between different molds.

##### `pick`

The simplest is
[`:pick` or `$?`](https://urbit.org/docs/hoon/twig/buc-mold/wut-pick); a
choice between a set of molds.

```
> *$?($foo $bar)
%foo
> ^-($?($foo $bar) %bar)
%bar
> ?=($?($foo $bar) %baz)
%.n
```

> Note: like all the other fork molds, `:pick` *bunts* to its first mold.

The `:dojo` uses the irregular form of `:pick` to print fork spans: `?(a b)`
is the same as `$?(a b)`.

```
> ? *$?($foo $bar)
  ?($bar $foo)
%foo
```

`:pick` is designed for simple union types, so it can fail to accurately differentiate more complex nouns. And it works by simply picking the first mold
with an icon that nests, so the order of molds is very important.

Here's two different molds that can match the same cell:

```
> `{@ @ @}`[1 2 3]
[1 2 3]
> `{@ ^}`[1 2 3]
[1 2 3]
```

The second mold can match many other cell structures as well:

```
> `{@ ^}`[1 2 3 4]
[1 2 3 4]
> `{@ ^}`[1 2 3 4 5]
[1 2 [3 4 5]]
```

Let's compare the differences when those molds are re-ordered in a `:pick`
mold. We'll use faces to easily see which mold was picked, and `:dojo`
variables for readability:

```
> =foo ?(a/{@ ^} b/{@ @ @})
> =bar ?(a/{@ @ @} b/{@ ^})
> ^-(foo [1 2 3])
a=[1 2 3]
> ^-(foo [1 2 3 4])
a=[1 2 [3 4]]
> ^-(bar [1 2 3])
a=[1 2 3]
> ^-(bar [1 2 3 4])
b=[1 2 [3 4]]
```

`{@ ^}` is the more general mold, so it matches everything when it's first.
These types of distinctions can be difficult to spot, so you should only use
`:pick` for simple choices, and be sure to consider the order of the molds.
Upgrade to a more specific fork mold for more complex choices.

##### `:claw`

Next we have
[`:claw` or `$@`](https://urbit.org/docs/hoon/twig/buc-mold/pat-claw), which
distinguishes nouns by their *depth*, i.e., it represents a fork between an
*atom* and a *cell*.

```
> =foo $@(@ {$foo *})
> *foo
0
> ? *foo
  ?(@ {$foo *})
0
> ^-(foo 123)
123
> ^-(foo [%foo 1])
[%foo 1]
> ^-(foo [%bar 1])
nest-fail
```

`:claw` is pretty straightforward, and it's always the right choice for the
union of an atom or a cell.

##### `:book`

But we often want a union of specific cells, which brings us to
[`:book` or `$%`](https://urbit.org/docs/hoon/twig/buc-mold/cen-book), the
tagged-union. Disambiguating cells can be difficult with the other fork molds,
but `:book` makes it simple and reliable by explicitly tagging each branch.

A `:book` is a union of *pages*, each a `:bank` with a leaf mold as its head.
Since each page is explicitly tagged, a `:book` can have an endless number of arbitrarily complex pages without ambiguity.

```
> =foo $%({$a @} {$b ^} {$c *})
> *foo
[%a 0]
> ? *foo
  ?({$a @} {$b * *} {$c *})
[%a 0]
> ^-(foo [%a 1])
[%a 1]
> ^-(foo [%b [2 3]])
[%b 2 3]
> ^-(foo [%c 4])
[%c 4]
> ^-(foo [%c [5 6]])
[%c [5 6]]
```

These tagged nouns probably look familiar; indeed, it's the same type of
structure as the twigs themselves: `twig` is a `:book`, and each specific twig
mold is a page in it.

##### `:bush`

Our final forking twig mold is
[`:bush` or `%^`](https://urbit.org/docs/hoon/twig/buc-mold/ket-bush),
which distinguishes cells based on their head-depth. A `:bush` is a fork
between a cell with an atomic head and a cell with a cellular head.

Let's reuse the trick of putting faces in our fork mold for clarity:

```
> =foo $^(a/{^ ^} b/{@ *})
> *foo
a=[[0 0] 0 0]
> ? *foo
  ?({@ *} {{* *} * *})
a=[[0 0] 0 0]
> ^-(foo [1 2 3])
b=[1 [2 3]]
> ^-(foo [[1 2 3] [4 5 6]])
a=[[1 [2 3]] 4 [5 6]]
```

This is a toy example, but it's hard to come up with a better one; the utility of a mold like this isn't obvious. And `:bush` isn't really used too
frequently -- branching based on the depth of the head of a cell is not a
broadly applicable pattern.

But one very clear use-case is supporting "autocons" behavior in a recursive
`:book` structure. Each page has to have an atomic head, so introducing a
`:bush` allows a cell of sibling pages to be a separate branch of the fork.

#### `twig` redux

This is the actual structure of `twig` -- it's a `:book` in a `:bush`. Here's a
severely truncated rendering:

```
$^  {p/twig q/twig}
$%  {$cast p/twig q/twig}
    {$cons p/twig q/twig}
    ...
==
```

This goes on for over 100 pages, with entries that match the structure of every
possible combination of Hoon syntax. The outer `:bush` provides the *autocons*
feature that we've previously discussed: a cell of sibling twigs is itself a
valid branch of `twig`.

We set out to understand the twig molds, as they're represented in the docs.
Now we know the purpose of the leaf-mold heads and `p`/`q`/`r` faces in their
structures, and understand how they all fit together into a coherent mold and
span. We've also establed `seed` and `moss` as documentary aliases: "a twig
that produces any value" and "a twig that produces a mold", respectively.

There's a handful of other molds that are used in `twig`. Some are specialized
molds, only used in `twig` and `span`. Examples include `taco` (soon to be
`taro`) and `toga` -- these can be safely ignored.

But others involve broadly applicable concepts: there are atoms, for instance -- `@` and specific auras (or aura-aliases), which we'll cover in the next
section. And there are generic data-structures: `pair`, `list`, `set`, `map`,
etc., which we'll address in later.

And then there's `wing`: a list of *limbs*. In broad terms, a `limb` is a
reference to a face, and a `wing` is nesting of limbs.

`a` is a `limb`; `b.a` is a `wing`:

```
> =+(a=[b=1 c=2] a)
[b=1 c=2]
> =+(a=[b=1 c=2] b.a)
2
```

#### abnormal molds

So now we grok `twig`, can read the molds in the docs, and have learned most
of twigs for creating molds. Let's run through the last two, which both have
the property of *not* normalizing their sample.

##### `:shoe`

[`:shoe` or `$_`](https://urbit.org/docs/hoon/twig/buc-mold/cab-shoe) creates
a mold from an example noun. That mold discards its sample and produces the
example noun: `$_(a)` is equivalent to `|=(* a)`.

```
> *@
0
> ^-(@ 123)
123
> *$_(1)
1
> ^-($_(1) 123)
123
> *^
[0 0]
> ^-(^ [123 456])
[123 456]
> *$_([1 1])
[1 1]
> ^-($(_[1 1]) [123 456])
[123 456]
```

The `:shoe` of a constant atom is effectively the same as a leaf mold. Here
we'll use the irregular syntax: `_`.

```
> *$foo
%foo
> ^-($foo %bar)
nest-fail
> *_%foo
%foo
> ^-(_%foo %bar)
nest-fail
```

But they're not literally the same; the `:shoe` implementation has guardrails
so that it can safely wrap any example noun:

```
> .=($foo _%foo)
%.n
```

Directly casting with a `:shoe` is redundant and obfusticatory -- we're
*always* casting by example. When we discussed *span* at the beginning, we introduced the term *icon*: the "span of the bunt of a mold". That's how
`:cast` always works; it bunts the mold, and then casts by example.
[`:like` or `^+`](https://urbit.org/docs/hoon/twig/ket-cast/lus-like) is the
natural cast-by-example twig on which `:cast` is built.

In other words, these expressions all perform the same operation:

```
> ^+(1 2)
2
> ^+(*_1 2)
2
> ^-(_1 2)
2
```

`:shoe` is most often used in conjunction with other molds. For instance, we can change part of the bunt of a `:bank`:

```
> *{@ @}
[0 0]
> ^-({@ @} [1 2])
[1 2]
> *{@ _1}
[0 1]
> ^-({@ _1} [1 2])
[1 2]
```

But remember: a `:shoe` is not a normalizer -- it only produces its example
noun. So we definitely don't want to call it:

```
> ({@ @} [1 2])
[1 2]
> ({@ _1} [1 2])
[1 1]
```

##### `:lamb`

[`:lamb` or `$-`](https://urbit.org/docs/hoon/twig/buc-mold/hep-lamb) creates
a mold for matching a gate from two sub-molds, one for the sample and the
other for the product. It's also not a normalizer, but just a `:shoe` wrapped
around an example gate created from the molds.

- `$-(* *)` matches any gate
- `$-(@ *)` matches a gate with an atomic sample
- `$-(@ @)` matches a gate with an atomic sample and product

... and so on.

The example gate accepts the first mold, and produces the bunt of the second:

`$-(^ @)` bunts to `|=(^ *@)`.

So `:lamb` is really just for casting or type-checking gates. It's used almost
exclusively in the sample molds of higher-order gates (gates that take gates
in their sample; `hard` and `soft` are two that we've already seen).

---

And that's it -- we've comprehensively covered cell molds! It's finally time
to turn our attention to atoms. So far, we've seen them only as unsigned
integers, but there's much, much more.
