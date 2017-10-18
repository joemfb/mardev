---
title: Atomic ambience
navhome: /hoon
sort: 4
next: true
---

### Atomic ambience

And now we turn our full attention to the lowly atom. We've seen *atom*
defined as "an unsigned integer of any size", and, for the most part, that's how we've used them thus far. But there have been hints of something more --
mention of "atomic strings", the unexplained oddities of *null* and *loobean*,
and you may have detected an atomic odor in our symbols themselves. So let's
dig in.

But first, a test: what do `0w0`, `~zod`, `~`, `''`, `&`, and `%$` have in
common?

Obviously they're all atoms (why else would we ask?), but there's more:

```
> `@`0w0
0
> `@`~zod
0
> `@`~
0
> `@`''
0
> `@`&
0
> `@`%$
0
```

This is at least disorienting, and perhaps disturbing. So let's back up ...

Most programming languages have syntactic support for various forms of literal
numbers: at least decimal and hexadecimal, often binary and octal as well.
This is pretty straightword, decimal `27` is octal `33` is hexadecimal `1b` is
binary `11011` is base64 `r`; we're all comfortable using different forms to represent the same conceptual number. And that's the first layer of atoms in
Hoon, different representations of numbers *as numbers*. But we can go much
further.

We know that computers work by assigning agreed-upon meanings to numbers --
that is always at the bottom of all our code and formats and encodings. But we
rarely (if ever) see a programming language that transparently works in this
way; instead we typically manipulate these numbers through towers of
abstractions.

Hoon supports both interpretations: various forms to represent numbers
*as numbers*, and standard approaches for encoding other types of data as
numbers. Despite always *being* unsigned integers, atoms are used to
*represent* signed integers, floats, dates, and even strings*.

> \* And there's more: while we're used to considering an arbitrary binary
> blob as a sequence of bytes (or a list of numbers), that's just a useful
> perspective, and not in any way a fundamental property of digital data. It's
> just as accurate to consider a binary blob as *one large number*. Indeed,
> Urbit encodes arbitrary nouns as atoms for persistence and network
> transmission.
>
> Any digital data is
> [already on the number line](https://github.com/armhold/infinity)! (My
> thanks to to `~barpub-tarber` for that link.)

You can even do math on them, all at one:

```
> (div (mul %bar (add 'foo' ~nec)) (sub 0xaf %.y))
312.820.849.957
```

Of course, you probably don't want to. That example has no semantic meaning
whatsoever as it ignores the representational significance of those atoms.
Instead, you want to differentiate the semantic meanings of those atoms that
are used *as numbers* from those that *encode non-numeric data*. In other
words, you want a type system.

#### auras

The mold twigs we've examined so far are not sufficient to the task of
constructing a type system for atoms. They're mostly concerned with the shape
of cells, but atoms are shapeless. Instead, Hoon atoms are tagged with an
*aura*: a symbol that signifies its type. And that type can only be
*advisory*: it documents intentions, but can't enforce semantics. Hoon has no
dependant types.

We've been using `@` as the mold for atoms -- it means the *empty* aura, and
no specific semantics. But we've still been creating atoms with auras in our
examples: any literal decimal number has the aura `@ud` (unsigned decimal):

```
> 1
1
> ? 10
  @ud
10
```

There are other unsigned numeric auras: for instance, `@ub` is binary, `@ux`
is hexadecimal and `@uw` is base64:

```
> `@ub`17
0b1.0001
> `@ux`17
0x11
> `@uw`17
0wh
```

#### casts

The characters of the aura form a hierarchy, used by the *nest* algorithm in
type-casts. All auras nest within `@`, all unsigned auras nest within `@u`, and so on. And casting atoms is more flexible than cells; you can cast from
general to specific, as well as the reverse:

```
> ^-(@ux ^-(@u 1))
0x1
> ^-(@u ^-(@ 1))
1
```

To cast between auras that do not nest in either direction, you must first
cast to `@`:

```
> ^-(@ux 10)
nest-fail
> ^-(@ux ^-(@ 10))
0xa
```

Conveniently, the irregular form of `:cast` does this for us, but only for
individual atoms:

```
> `@ux`10
0xa
> `{@ux @ux}`[10 20]
nest-fail
> `{@ux @ux}``{@ @}`[10 20]
[0xa 0x14]
```

#### standard auras

There are many auras built into Hoon that can be represented with literal
syntax. We'll examine a few here, and you can find the rest in the
[official docs](https://urbit.org/docs/hoon/twig/atom/sand/).

##### `@u`

The print formats for the unsigned integers we've seen are the same as the
literal syntax:

```
> ? 1
  @ud
1
> ? 0b1
  @ub
0b1
> ? 0x1
  @ux
0x1
> ? 0w1
  @uw
0w1
```

If you've tried to use a 4-digit decimal number, you were probably surprised
to find the `:dojo` preventing you. Hoon requires the German-style `.`
separator, and the `:dojo` enforces this with per-character syntax validation
(via the parser, not the typesystem).

```
> ? 1.234
  @ud
1.234
> `@ux`1.234
0x4d2
> `@ud`0x1.0000
65.536
> `@ux`0w1.00000
0x4000.0000
```

The unsigned auras (`@u`) are the easiest to understand, as their semantics map
precisely to the underlying atoms.

##### `@p`

Urbit also has `@p`, a special representation of unsigned integers:

```
> `@p`0
~zod
> `@p`1
~nec
> `@p`123.456.789
~tagtyv-micdyt
```

`@p` is a phonemic base, representing arbitrary atoms as pairs of
pronounceable syllables. It's a base-65536 system designed for
[Urbit identities](http://urbit.org/posts/address-space/), but it's also
useful for memorably representing arbitrary data, such as hashes or passcodes.
While the system is somewhat complicated (documented in detail on the
[Urbit forum](http://urbit.org/fora/posts/~2016.7.27..21.03.03..f2da~/)), it
remains a notation for unsigned integers.

##### `@s`

In contrast, we have `@s` for signed integers. Since the domain of signed
integers is obviously different from that of unsigned integers, `@s` requires
some kind of encoding or transformation. It's approach is quite simple: for any absolute value `n`, the positive integer is `2n` and the negative is
`2n-1`:

```
> ? --5
  @sd
--5
> ? -5
  @sd
-5
> `@`--5
10
> `@`-5
9
> `@sx`10
--0x5
```

There are also auras for floats (`@r`) and dates (`@d`). As you might
expect, these have much more complicated encodings onto the underlying
unsigned atoms. We'll leave their mysteries to the industrious ...

#### atomic strings

We've already seen atomic strings in our Hoon parsing examples. This is the
aura `@t`, aliased to `cord`:

```
> ? 'foo'
  @t
'foo'
> ? `cord`'foo'
  @t
'foo'
```

There are also specializations of `@t`: `@ta` is ASCII text, and `@tas` is
ASCII with the symbol constraint (lower-case alpha-numeric with hyphens):

```
> `@ta`'foo'
~.foo
> `@tas`'foo'
%foo
```

And these strings are indeed atomic:

```
> `@`'foo'
7.303.014
> `@t`7.303.014
'foo'
```

This approach to strings is not at all typical, but it's not too complicated.
The encoding of multi-character strings is non-obvious in decimal:

```
> `@`'a'
97
> `@`'b'
98
> `@`'c'
99
> `@`'abc'
6.513.249
```

But switching to hexadecimal make the association clear:

```
> `@ux`'a'
0x61
> `@ux`'b'
0x62
> `@ux`'c'
0x63
> `@ux`'abc'
0x63.6261
```

The encoding is simply UTF-8, with the least-significant byte first.

#### custom auras

The atomic type system is also entirely customizeable. You can extend built-in
auras, or simply create your own:

```
> `@pat`1
~nec
> `@user`1
1
> `@xyz`0w1.00000
0x40000000
```

The same hierarchical prefix rules apply to type-casts: `@pat` nests within
`@pa` within `@p`. Extensions of built-in auras inherit their print formats,
while custom auras are printed in hex without separators.

Custom auras are perfectly suited to represent binary data with specific
semantics. For instance, here's a single-pixel PNG (`#000000` and zero
opacity, from [1x1px.me](http://www.1x1px.me/)):

```
$ xxd ~/Downloads/000000-0.png
0000000: 8950 4e47 0d0a 1a0a 0000 000d 4948 4452  .PNG........IHDR
0000010: 0000 0001 0000 0001 0804 0000 00b5 1c0c  ................
0000020: 0200 0000 0b49 4441 5478 9c63 6260 0000  .....IDATx.cb`..
0000030: 0009 0003 1911 d9e4 0000 0000 4945 4e44  ............IEND
0000040: ae42 6082                                .B`.
```

```
> `@xpng`0x8950.4e47.0d0a.1a0a.0000.000d.4948.4452.0000.0001.0000.0001.0804.0000.00b5.1c0c.0200.0000.0b49.4441.5478.9c63.6260.0000.0009.0003.1911.d9e4.0000.0000.4945.4e44.ae42.6082
0x89504e470d0a1a0a0000000d4948445200000001000000010804000000b51c0c020000000b49444154789c6362600000000900031911d9e40000000049454e44ae426082
> `@ud`0x8950.4e47.0d0a.1a0a.0000.000d.4948.4452.0000.0001.0000.0001.0804.0000.00b5.1c0c.0200.0000.0b49.4441.5478.9c63.6260.0000.0009.0003.1911.d9e4.0000.0000.4945.4e44.ae42.6082
30.888.123.700.406.672.710.585.050.819.411.191.846.508.756.935.685.128.902.505.913.043.957.563.447.763.594.157.311.970.366.253.837.430.070.238.871.299.383.053.686.528.112.690.739.299.183.785.021.162.390.842.926.850.178
```

You *could* use `@png`, but then it would print, confusingly, in the phonemic
base (and nest with `@p`). But `@xpng` prints as an undifferented blob, and
enables you to write type-safe code to operate on it. (Of course, you'd
probably want a cellular structure for operating on a parsed PNG. But you'd have
to serialize it eventually ...).

#### bitwidth suffix

There's another piece of type information that can be embedded within an aura:
an optional uppercase suffix letter representing an atom's bitwidth as a binary
exponent. `A` is 2<sup>0</sup> and `Z` is 2<sup>25</sup>, so auras can
represent atoms from 1 bit to 4MB. (You can have larger atoms of course, but
the type-system can't express their size.)

The suffix factors into type-casts as well -- intuitively, smaller bitwidths
nest within larger.

```
> `@A`0
0
> ^-(@B `@A`0)
0
> ^-(@A `@B`0)
nest-fail
```

The bitwidths of the Urbit identity classes provide additional clarity:

- `@pD`: galaxy, 2<sup>3</sup>
- `@pE`: star, 2<sup>4</sup>
- `@pF`: planet, 2<sup>5</sup>
- `@pG`: moon, 2<sup>6</sup>
- `@pH`: comet, 2<sup>7</sup>

#### constant atoms

There is final piece of type information that can describe Hoon atoms ...

TODO:

#### dynamic vases

TODO: instrument the compiler at run-time

```
=>(!>(%~zod) ?>(?=({$atom *} -) ,.-))
```
