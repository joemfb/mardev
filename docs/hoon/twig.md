---
title: How to read the twig docs
navhome: /hoon
sort: 1
next: true
---

### How to read the twig docs

A "twig" is just a Hoon expression. In a fundamental way, learning Hoon is
learning the twigs -- and there's over 100 of them! Most twigs consist of a
*keyword* or *digraph rune* followed by a fixed structure of "sub-twigs".

`twig` is also the name of the `mold` for the Hoon AST -- the data structure
produced by the Hoon parser. Such ambiguity is a key Urbit aesthetic. While
the official [twig reference docs](https://urbit.org/docs/hoon/reference/)
have a pretty consistent structure, they present a chicken-and-egg-type
problem -- they are themselves difficult to understand without first
understanding twigs generally. Learning the twigs by reading the reference
docs can feel like an exercise in unwinding a Rube Goldberg machine; you
catch yourself saying "a `:cons` is just a `%cons` in the category of `$cons`",
and *despair*.

This document is intended to bridge that gap. We'll work through the standard
sections in the official docs, using the simple twig
[`:cons`](https://urbit.org/docs/hoon/twig/col-cell/hep-cons/) as our example.
Of course, we'll introduce a handful of other twigs along the way.

There are easily-runnable `:dojo` examples throughout this document. Run them!
You'll need an urbit instance, of course, to use `:dojo`, the urbit shell.

[Install urbit](https://urbit.org/docs/using/install/) and
[get started](https://urbit.org/docs/using/setup/), or just head straight for
the [urbytes](https://urbit.org/docs/byte/): bottom-up lessons that'll have
you on your feet quickly.

#### title

Twig titles have three parts: keyword, rune, rune pronunciation:

> :cons :- "colhep"

The keyword and the rune are *interchangeable* in a twig -- keywords are
provided to help with initial "ASCII overload", while runes are standard style.
(Elsewhere, the keyword-or-rune that begins a twig is called the "stem" or
"sigil".)

```
~your-urbit:dojo> :cons(1 2)
[1 2]
~your-urbit:dojo> :-(1 2)
[1 2]
```

Prior to the introduction of keywords, rune
[pronunciations](https://urbit.org/docs/hoon/syntax/#-glyphs-and-characters)
were the only twig names: its much nicer to say "colhep" in place of "colon
hyphen", and the distinction becomes more pronounced when you get to "question
mark" or "ampersand". Now, twigs are most often referred to by their keyword.

Runes have a distinct advantage over keywords: the first character represents
a group of related twigs, establishing a class of sorts. For instance, all the
runes starting with `:` construct cells.

As we advance we'll use fewer keywords in the examples. They are training
wheels, and we intend to fly.

#### mold

After the title, we have the twig mold and a brief description of the twig:

> `{$cons p/seed q/seed}`: construct a cell (2-tuple)

In a casual reading, the mold is almost irrelevant -- it shows the structure of
the twig body ("bulb"), but that's described more clearly in the subsequent
*Syntax* section. The mold is really a peek under the hood, defining the
structure of the twig as it's implemented in the compiler. If you understand
that, you understand twigs.

But it's likely that you don't yet understand. Vaguely speaking, the twig mold
is the type-signature of the twig AST. That's not quite right in some subtle
ways, and may even obstruct a deep understanding of Hoon.

For now, we'll just note that the mold generally describes the structure an
expression. A valid `:cons` expressions contains two expressions, `p` and `q`.
In this case, `p` and `q` are *seeds*, meaning twigs that produce values.
*seed* is constrasted with *moss*, meaning a twig that produces a mold (see
[`:cast` or `^-`](http://urbit.org/docs/hoon/twig/ket-cast/hep-cast/) for a
twig with both).

Molds will be explored in detail in the [next section](/docs/hoon/mold).

#### expansion

Hoon twigs can be considered in two categories: *natural* and *synthetic*.
Natural twigs are the primitive capabilities of the language itself, while
synthetic twigs are best thought of as built-in macros. The vast majority
of twigs are, in fact, synthetic.

Synthetic twigs will all have some type of expansion: either a direct "Expands
to" or a *pseudocode* "Expands to" with a "Compiler macro". These all offer
different ways to again peek under the hood, demonstrating how advanced
capabilities are built out of primitive features. Let's analyze a couple
examples:

##### `:scon` or `:_`

[`:scon`](https://urbit.org/docs/hoon/twig/col-cell/cab-scon/) is the direct
inverse of `:cons`: it creates a cell with its second sub-twig as the head and
its first as the tail.

The mold is `{$scon p/seed q/seed}`, and the expansion is simple and direct:

> `:cons(q p)` or `:-(q p)`

```
~your-urbit:dojo> :cons(1 2)
[1 2]
~your-urbit:dojo> :scon(1 2)
[2 1]
~your-urbit:dojo> :_(1 2)
[2 1]
```

This twig exists only for convenience and aesthetics; it's occasionally
clearer to write a cell-constructing expression with the tail first.

##### `:conp` or `:*`

[`:conp`](https://urbit.org/docs/hoon/twig/col-cell/tar-conp/) is the
"n-tuple" version of `:cons`: it creates a cell from an arbitrary number of
expressions. (We've already seen the `:dojo` print cells in its *irregular*
form, `[]`.)

```
~your-urbit:dojo> :*(1 2 3 4)
[1 2 3 4]
```

Its mold is `{$conp p/(list twig)}` -- since it takes an arbitrary number of
sub-twigs, there's no direct expansion for it. Instead, we have some
pseudocode and a compiler macro:

> *Pseudocode*, `a`, `b`, `c`, ... as elements of `p`:
>
> `:cons(a :cons(b :cons(c :cons(... z)))))`

That's fairly straightforward, although probably easier to read in irregular
syntax:

```
[a [b [c [... z]]]]
```

Which is actually the same as `[a b c ... z]`, since cells associate to the
right.

<blockquote>
The fact the cells associate right is a pretty key Hoon insight. To make it
concrete, note that all of these are exactly equivalent:

```
> :conp(1 2 3 4)
[1 2 3 4]
> :cons(1 :cons(2 :cons(3 4)))
[1 2 3 4]
> [1 2 3 4]
[1 2 3 4]
> [1 [2 [3 4]]]
[1 2 3 4]
```
</blockquote>

As for the macro, we have both keyword and rune forms:

```
:loop
:ifno  p
  !!
:ifno  t.p
  i.p
:cons  i.p
:moar(p t.p)
```

```
|-
?~  p
  !!
?~  t.p
  i.p
:-  i.p
$(p t.p)
```

Without knowing Hoon, that appears quite complicated -- in fact, it's a
straightforward list reduction. It takes a null-terminated structure of
arbitrary length (`p`) and converts it to a non-terminated structure.

To try it out, we'll need to create a list of some sort: this creates a list
of atoms and stores it in a `:dojo` variable:

```
~your-urbit:dojo> =p ^-((list @) [1 2 3 4 ~])
```

Reformatting the macro from *tall* to *flat* form (see [syntax](#syntax)
below) makes it easier to copy and evaluate:

```
~your-urbit:dojo> |-(?~(p !! ?~(t.p i.p [i.p $(p t.p)])))
[1 2 3 4]
```

This is a pretty trivial transformation, but we can at least see that it does
what it says on the can. In practice, this macro is evaluated against the AST
of the arbitrary number of sub-twigs in a `:conp` expression.

> Understanding the "Compiler macro" expansions will often require
> intermediate-levels of familiarity with Hoon; they use a wide variety of
> twigs and data-structures, and are implemented in a compact style without
> much exposition. This example alone involves terminated vs non-terminated
> tuples, lists, loops, recursion, pattern-matching, and so on.
>
> This document is intended to explain the structure and purpose of the
> official docs, as much any specific example. As long as the context is clear,
> rest assured that we will build up to these concepts and many more.

#### produces

This section isn't always present -- when it is, it's a normative description
of the product of the twig, as defined by the mold.

Some examples:

###### `:cons` or `:-`

> The cell of `p` and `q`.

###### `:bump` or `.+`

`{$bump p/atom}`: the Nock increment instruction
([docs](https://urbit.org/docs/hoon/twig/dot-nock/lus-bump/))

> `p` plus `1` if `p` is an atom; otherwise, crashes. The product atom has no
> aura.

There are some other documentation sections that may present instead:

##### convention

This is used to document *hint* twigs, used to create a side-effect in the Nock
interpreter without affecting the *result* of the computation itself.

###### `:dump` or `~&`

`{$dump p/seed q/seed}`: a debug printf
([docs](https://urbit.org/docs/hoon/twig/sig-hint/pam-dump/))

> Prettyprints `p` on the console before computing `q`.

##### normalizes/defaults to

These document some advanced mold twigs, especially those that represent a
choice between sub-molds.

###### `:claw` or `$@`

`{$claw p/moss q/moss}`: mold which normalizes a union tagged by depth
([docs](https://urbit.org/docs/hoon/twig/buc-mold/pat-claw/))

> **Normalizes to**
>
> `p`, if the sample is an atom; `q` otherwise.
>
> **Defaults to**
>
> The default of `p`.

In other words, `:claw` represents a choice between an atom or a cell, and
defaults (or "bunts") to an atom. We'll discuss these concepts in much more
depth when we return to molds.

#### syntax

In contrast to other ASCII-heavy languages, Hoon syntax is highly regular and
structured. The vast majority of twigs consist of a keyword or rune "sigil"
followed by a fixed structure of sub-twigs, in either *tall* or *flat* form.

We'll consider the *regular* twig forms first, and then examine some examples
of *irregular* expressions.

##### tall/flat form

All regular twigs can be expressed in either *tall* or *flat* form. We've
already seen flat form in all our `:dojo` examples: it's the twig sigil
followed immediately by parenthesis enclosing the twig body ("bulb"), with each
sub-twig separated by a single space.

For example:

```
~your-urbit:dojo> :-(%a %foobar)
[%a %foobar]
```

The exact same expression can be rewritten in tall form: the sigil and each
sub-twig must be separated by more-than-one space, allowing much greater
flexibility:

```
~your-urbit:dojo> :-  %a  %foobar
[%a %foobar]
~your-urbit:dojo> :-  %a
                  %foobar
[%a %foobar]
```

Tall form is a key syntactic innovation in Hoon, it allows arbitarily deep
nestings of functional expressions, while neither encroaching on the left
margin nor producing a terminator pileup (`))))`).

Tall form twigs can contain tall or flat forms, nested to any depth, while
flat form twigs can only contain other flat forms.

```
~your-urbit:dojo> :-  :cons(%a %b)
                  :scon(%foo %bar)
[[%a %b] %bar %foo]
```

##### bulb structures

There are a handful of different twig body types, with requisite syntax
variation.

The first are fixed-length twigs. While all regular flat forms are
terminated with a closing parenthesis, regular tall forms with a fixed
length are unterminated -- the compiler can obviously tell when they are
"closed". Such twigs are be labeled as *n*-fixed (our dear friend `:cons`,
for instance, is "2-fixed").

A close relative of `:cons` is
[`:cont` or `:+`](https://urbit.org/docs/hoon/twig/col-cell/lus-cont/), which
is 3-fixed:

```
~your-urbit:dojo> :cont(1 2 3)
[1 2 3]
~your-urbit:dojo> :+  %a
                    %b
                  %c
[%a %b %c]
```

In contrast, a twig with an arbitrary number of sub-twigs is said to be
*running*. A running twig in tall form requires an explicit terminator (`==`).

We've already seen `:conp` in flat form:

```
~your-urbit:dojo> :conp  1
                         2
                         3
                         4
                  ==
[1 2 3 4]
```

Finally, we have some twigs which operate on a sequence of paired sub-twigs;
these are said to be *jogging*. In the flat form of jogging twigs, these pairs
are separated by a comma, followed by a single space. The tall form is simply
separated by more-than-one space, again followed by a terminator.

Almost all *jogging* twigs have an initial or final fixed structure, so they
will be described as "1-fixed, then jogging" or "jogging, then 1-fixed". In our
example, we introduce yet another new twig:

[`:make` or `%=`](https://urbit.org/docs/hoon/twig/cen-call/tis-make/)
produces a noun after applying some changes to it. It's an incredibly powerful
"natural" twig, underpining many other capabilities of Hoon. Its regular form
is "1-fixed, then jogging".

First, we need a noun to modify:

```
~your-urbit:dojo> =a [p=1 q=2 r=3]
~your-urbit:dojo> a
[p=1 q=2 r=3]
```

Then, we produce a new noun by referencing the old, along with a list of
changes to be made:

```
~your-urbit:dojo> :make(a p 5)
[p=5 q=2 r=3]
~your-urbit:dojo> %=(a p 5, q 4)
[p=5 q=4 r=3]
```

In tall form, it's a good practice to format the twig in a way that visually
groups the paired sub-twigs:

```
~your-urbit:dojo> %=  a
                    p  5
                    q  4
                  ==
[p=5 q=4 r=3]
```

There's also a fourth twig body type: the *battery*, which is unique to cores.

##### irregularities

In addition to the *tall* and *flat* regular forms we've covered, there are
some twigs that have *irregular* syntactic forms as well. A few twigs have
*only* an irregular form. Irregular syntax doesn't follow the rules we've
layed out (as the name suggests), but they are always *flat* form only.

We've already seen the irregular form of `:conp`: `[%foo %bar %baz]`.

`:make` also has an often-used irregular form:

> Irregular: foo(x 1, y 2, z 3) is :make(foo x 1, y 2, z 3).

Reusing our last example:

```
~your-urbit:dojo> a(p 5, q 4)
[p=5 q=4 r=3]
```

See the official [Hoon docs](https://urbit.org/docs/hoon/syntax/) for a much
more detailed syntax analysis.

#### discussion

This section is only rarely present, usually to provide some general context
to a twig, often relating it to other twigs. Occasionally, it's a discussion of
the compiler implementation strategy. Don't worry if it's not clear initially
-- it'll become so if it's important information.

#### examples

And finally, all the twigs should have some `:dojo` examples, including tall
and flat form with both runes and keywords, as well as irregular syntax
where appropriate. Do try the examples; copy or re-type them into your `:dojo`.
If they're unclear, broken, or incomplete, open an issue (or better yet, a pull
request) on the [github docs repo](https://github.com/urbit/docs). This is an
excellent way to start contributing to Urbit!

---

And there we have it. This should put you well on your way to an epiphany of
Hoonery.

And yet, we glossed over molds. This must be rectified -- they are a unique
feature of Hoon, and one of its most important. *We must go deeper*.
