---
title: Iconography of a type system
navhome: /hoon
sort: 2
next: true
---

### Iconography of a type system

**_Molds: Part 1_**

The previous section describes the twig molds as "the type-signature of the
twig AST", then immediately disavows that description. We'll take a second
crack at the `:cons` twig mold: `{$cons p/seed q/seed}`, after a quick detour
through the Hoon type system.

#### type

The notion of "type" in a programming language is heavily contested and
overloaded; nearly every language uses it to mean something at least somewhat different from the others. The inestimable Gary Bernhardt (of
[*Destroy All Software*](https://www.destroyallsoftware.com/screencasts) fame)
has written the best
[summary of type systems](https://gist.github.com/garybernhardt/122909856b570c5c457a6cd674795a9c)
that I've ever found.

Hoon aims to make its type system as simple as possible, but no simpler: the
type of a twig is a *span*, the set of nouns the expression can produce. There
is no syntax for a span, it is always inferred by the Hoon compiler. We can
cast any noun *A* to the span of any other noun *B*, as long as the span of *A*
is provably a subset of the span of *B* (we say that span *A* *nests* in span
*B*). Fundamentally, this is type-by-example.

You might expect that it would be very tedious to always need to create
examples for every type of noun you need ... and you would be right! This
is the job of a mold.

A mold is a gate (anonymous function:
[`:gate` or `|=`](https://urbit.org/docs/hoon/twig/bar-core/tis-gate)) that
accepts any noun as its sample (argument) and produces a structured noun from
it. Molds are *data validators* and *type constructors*. The span of the
product of a mold will always be the same, regardless of the sample -- if a
mold can't normalize its sample, it is said to *bunt*, producing a *default*
noun. So molds are the mechanism which makes type-by-example practical.

> The "span of the bunt of a mold" is a bit of a mouthful; we can refer
> instead the **icon** of a mold.

In spite of their definition as normalizing functions, production of example nouns is by far the most frequent use of molds. Calling them as functions
is generally bad style, outside of validating untrusted data. This is a very
important distinction, and more than a little paradoxical ... We'll examine this duality more closely in Part 2.

#### base molds

Because molds are used so frequently, they are privileged with quite a bit
of special syntax. Here are the basic forms:

- `*` normalizes to any noun
- `^` normalizes to any cell
- `@` normalizes to any atom
- `?` normalizes to a *loobean*, a Hoon boolean
- `$~` normalizes to null

`*` does double duty; it also produces the *bunt* of a mold:

```
~your-urbit:dojo> **
0
~your-urbit:dojo> *^
[0 0]
~your-urbit:dojo> *@
0
~your-urbit:dojo> *?
%.y
~your-urbit:dojo> *$~
~
```

We can can cast a noun to the *icon* of a mold with
[`:cast` or `^-`](https://urbit.org/docs/hoon/twig/ket-cast/hep-cast):

```
~your-urbit:dojo> :cast(* 0)
0
~your-urbit:dojo> ^-(* [0 0])
[0 0]
~your-urbit:dojo> ^-(^ 0)
nest-fail
~your-urbit:dojo> ^-(^ [0 0])
[0 0]
~your-urbit:dojo> ^-(@ 0)
0
~your-urbit:dojo> ^-(@ [0 0])
nest-fail
```

> If the span of the noun does not **nest** within the icon of the mold, the
> type-cast expression crashes and prints `nest-fail`.

We can also test a noun against a mold with the pattern-match twig
[`:fits` or `?=`](https://urbit.org/docs/hoon/twig/wut-test/tis-fits):

```
~your-urbit:dojo> :fits(* 0)
%.y
~your-urbit:dojo> ?=(* [0 0])
%.y
~your-urbit:dojo> ?=(^ 0)
%.n
~your-urbit:dojo> ?=(^ [0 0])
%.y
~your-urbit:dojo> ?=(@ 0)
%.y
~your-urbit:dojo> ?=(@ [0 0])
%.n
```

We've skipped over `?` and `$~`, but for good reason. We haven't yet discussed
*auras*, the type-system for atoms, and both `?` and `$~` are special cases
within atoms generally. For now, suffice to say that the product of a `:fits`
twig is always `?`, a *loobean* of either *yes* (`%.y`, `&`) or *no* (`%.n`,
`|`).

We'll come back to atoms -- but first we return to our old friend
`:cons`. Let's build up an understanding of its mold (`{$cons p/seed q/seed}`),
along with the others that we see defining twigs in the reference docs.

#### leaf molds

The first is of the form `$foo`: a constant, atomic mold. The mold `$foo`
matches the constant atom `%foo`, and *nothing else*.

```
~your-urbit:dojo> *$foo
%foo
~your-urbit:dojo> :cast($foo %foo)
%foo
~your-urbit:dojo> ^-($foo %bar)
nest-fail
~your-urbit:dojo> :fits($foo %foo)
%.y
~your-urbit:dojo> ?=($foo %bar)
%.n
```

Leaf molds are an excellent opportunity to demonstrate that molds are gates. A
gate ([`:gate` or `|=`](https://urbit.org/docs/hoon/twig/bar-core/tis-gate))
is an anonymous function built from two sub-twigs: a mold to normalize the
sample (argument), and an expression producing the product.

We already know that the sample of a mold must be any noun (`*`), and that
leaf molds produce a constant atom. The gate equivalent of `$foo` is thefore
obviously `|=(* %foo)`.

We can call a gate with
[`:call` or `%-`](https://urbit.org/docs/hoon/twig/cen-call/hep-call):

```
~your-urbit:dojo> :call(:gate(* %foo) 1)
%foo
~your-urbit:dojo> %-(|=(* %foo) [1 2])
%foo
```

Or with the Lisp-like irregular syntax:

```
~your-urbit:dojo> (|=(* %foo) [1 2])
%foo
```

We can prove the equivalence of `$foo` and our gate with the equality-test
twig [`:same` or `.=`](https://urbit.org/docs/hoon/twig/dot-nock/tis-same):

```
~your-urbit:dojo> :same(1 1)
%.y
~your-urbit:dojo> .=(1 2)
%.n
~your-urbit:dojo> .=($foo |=(* %foo))
%.y
```

And we can use our gate like a mold; bunting, casting, and pattern-matching:

```
~your-urbit:dojo> *|=(* %foo)
%foo
~your-urbit:dojo> ^-(|=(* %foo) %foo)
%foo
~your-urbit:dojo> ^-(|=(* %foo) %bar)
nest-fail
~your-urbit:dojo> ?=(|=(* %foo) %foo)
%.y
~your-urbit:dojo> ?=(|=(* %foo) %bar)
%.n
```

Leaf molds on their own are so trivial as to seem unnecessary; their real
utility comes in conjunction with other molds.

#### cell molds

We've already seen the mold for any cell (`^`), next we have the mold for a
*specific* cell:
[`:bank` or `$:`](https://urbit.org/docs/hoon/twig/buc-mold/col-bank).
`:bank($foo $bar)` matches a cell with a head of `%foo` and a tail of `%bar`:

```
~your-urbit:dojo> *:bank($foo $bar)
[%foo %bar]
~your-urbit:dojo> :cast(:bank($foo $bar) [%foo %bar])
[%foo %bar]
~your-urbit:dojo> ^-($:($foo $bar) [%foo %baz])
nest-fail
~your-urbit:dojo> ?=($:($foo $bar) [%foo %baz])
%.n
```

> `:bank` doesn't have to wrap multiple molds, or even match a cell, but it
> almost always does. `:bank(@)` is valid Hoon -- entirely equivalent to just
> `@`.

Switching to the *irregular* syntax for `:bank` brings us most of the way home:

```
~your-urbit:dojo> ^-({$foo $bar} [%foo %bar])
[%foo %bar]
```

There's an obvious and helpful visual symmetry between the cell itself
(`[]`, irregular `:conp`) and the mold that matches it (`{}`, irregular
`:bank`).

> The logically-equivalent gate for `{$foo $bar}` is `|=(* [%foo %bar])`, but
> it's not literally equivalent. While the Hoon compiler can automatically
> *cons* multiple parallel twigs, there's no equivalent syntax, so our gate
> has to explicitly construct the cell.

Leaf and base molds are often combined in a `:bank` for flexible pattern
matching with `:fits`:

```
~your-urbit:dojo> ?=({$foo *} [%foo 123])
%.y
~your-urbit:dojo> ?=({$foo *} [%foo 123 456])
%.y
~your-urbit:dojo> ?=({$foo @} [%foo 123])
%.y
~your-urbit:dojo> ?=({$foo @} [%foo 123 456])
%.n
~your-urbit:dojo> ?=({$foo ^} [%foo 123])
%.n
~your-urbit:dojo> ?=({$foo ^} [%foo 123 456])
%.y
```

This is where the expressive power of molds starts to shine through, combining
specific and generic molds to match arbitrary patterns within nouns. We'll come
come back to this approach when we're unpacking complex structures.

#### labeled surfaces

Introducing one last mold will bring us to enlightment:
[`:coat` or `$=`](https://urbit.org/docs/hoon/twig/buc-mold/tis-coat). This
creates a mold which wraps a `face` around another mold.

```
~your-urbit:dojo> :cast(:coat(a $foo) %foo)
a=%foo
~your-urbit:dojo> ^-($=(a *) [1 2])
a=[1 2]
```

The irregular form of `:coat` should already be familiar:

```
~your-urbit:dojo> ^-(a/$foo %foo)
a=%foo
~your-urbit:dojo> ^-(a/* [1 2])
a=[1 2]
```

Casting with a `:coat` is labeling our product noun, but if we compare the unlabeled and labeled nouns, we see that they're exactly the same!

```
~your-urbit:dojo> .=([1 2] [1 2])
%.y
~your-urbit:dojo> .=([1 2] ^-(a/* [1 2]))
%.y
```

This is a bit strange: the label hasn't changed our noun, instead, it appears
to be metadata of some sort. In fact it's type information, part of our span.

We can apply a label (called a `face`) directly to a noun with another casting
twig: [`:name` or `^=`](https://urbit.org/docs/hoon/twig/ket-cast/tis-name).

```
~your-urbit:dojo> :name(a 1)
a=1
~your-urbit:dojo> ^=(a [1 2])
a=[1 2]
```

And the `:dojo` is actually using the irregular form of `:name` when it prints
our product:

```
~your-urbit:dojo> a=1
a=1
~your-urbit:dojo> a=[1 2]
a=[1 2]
```

> It's important to understand that *faces* are not part of nouns themselves,
> but only exist in the span of a noun. This is why `:name` is a type-casting
> twig; casting a noun can only change the span.
> 
> The realization that labels are types is  key to a general understanding of
> Hoon, which has no traditional environment or symbol table. Every expression
> in Hoon is simply evaluated against a noun (the "subject" of the
> expression), so "declaring" a "variable" is constructing a new subject,
> usually with a labeled noun, and evaluating another expression against that
> subject.
> 
> The subject-oriented nature of programming in Hoon is the focus of
> [Urbyte 2](https://urbit.org/docs/byte/2/).

So `:coat` is to a mold as `:name` is to a noun: instead of modifying the span
of a noun to include a face, we're modifing a mold so it will add a face to the span of its product. Bunting some example molds (producing their default, example products) should make this clear:

```
~your-urbit:dojo> *@
0
~your-urbit:dojo> *a/@
a=0
~your-urbit:dojo> *^
[0 0]
~your-urbit:dojo> *a/^
a=[0 0]
```

Since a `:bank` is made out of molds, we can use `:coat` inside it to apply
faces to specific portions of a normalized noun:

```
~your-urbit:dojo> *{@ @}
[0 0]
~your-urbit:dojo> *{a/@ b/@}
[a=0 b=0]
~your-urbit:dojo> ^-({@ @} [1 2])
[1 2]
~your-urbit:dojo> ^-({a/@ b/@} [1 2])
[a=1 b=2]
```

#### twig structures

Let's now bring back the original twig mold we are trying to understand:
`{$cons p/seed q/seed}`. Fully expanded, it looks something like this:

```
:bank($cons :coat(p seed) :coat(q seed))
```

```
$:($cons $=(p seed) $=(q seed))
```

This is a mold that matches a cell with a head of `%cons` and a tail of two
seeds: `p` and `q`. `seed` is documentation-only terminology; it means a `twig`
that produces *any* value, as opposed to `moss`, which signifies a `twig` that
produces a mold. This distinction becomes clear when we examine the mold
for `:cast`, which casts a noun to the icon of a mold: `{$cast p/moss q/seed}`.

> It bears repeating that `moss` and `seed` are simply aliases for `twig`.
> You can look up the definition of `twig` in `hoon.hoon` (search for
> "++&nbsp;&nbsp;twig") and find the actual mold for `:cons`:
> 
> `{$cons p/twig q/twig}`

These twig molds match the structure of the Hoon abstract syntax tree (AST).
An AST is the structure produced by a parser, so let's parse some Hoon.

`ream` is the name of a standard library gate that applies the Hoon parser to
a `cord` (an atomic string), producing a `twig`.

```
~your-urbit:dojo> (ream '1')
[%sand p=%ud q=1]
~your-urbit:dojo> (ream '2')
[%sand p=%ud q=2]
~your-urbit:dojo> (ream ':cons(1 2)')
[%cons p=[%sand p=%ud q=1] q=[%sand p=%ud q=2]]
~your-urbit:dojo> (ream ':-(1 2)')
[%cons p=[%sand p=%ud q=1] q=[%sand p=%ud q=2]]
```

[`sand`](https://urbit.org/docs/hoon/twig/atom/sand/) is the twig for any
literal atom. So, just by looking, we can see that these nouns match the
structure of our mold. Let's cast anyway, just to make sure:

```
~your-urbit:dojo> ^-({$cons p/twig q/twig} (ream ':-(1 2)'))
nest-fail
```

Wait ... What?

```
~your-urbit:dojo> ^-({$cons *} (ream ':-(1 2)'))
nest-fail
~your-urbit:dojo> ^-({$cons * *} (ream ':-(1 2)'))
nest-fail
```

????!!1!

We can see the noun, we can read the mold. Everything we've learned up to this
point tells us that they should match.

In desperation, we copy and paste:

```
~your-urbit:dojo> ^-({$cons p/twig q/twig} [%cons p=[%sand p=%ud q=1] q=[%sand p=%ud q=2]])
[%cons p=[%sand p=%ud q=1] q=[%sand p=%ud q=2]]
```

And, somehow, this works -- which should be confusing. It *is* confusing. We
are confuse.

---

I was, in fact, quite confused while writing this section. And unexpected
`nest-fail` errors can be quite frustrating when learning Hoon. There's an
easy workaround for cases like this, which we'll get to. But before we do, we
need to understand why this has happened.
