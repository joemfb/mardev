---
navhome: /gen
title: 'naked'
sort: 1
next: true
---

### 'naked' generator

The simplest type of generator is just a `gate`.

##### `gate`

A gate is an anonymous function. It takes a *sample* (an argument noun) and
produces a noun. A gate expression has two parts, a mold for the sample, and an
expression producing a *product*.

An "identity" gate, producing it's sample, unchanged:

```
~your-urbit:dojo> (|=(a/* a) 1)
1
~your-urbit:dojo> (|=(a/* a) [1 2])
[1 2]
```

##### generator

We can convert this gate to a trivial generator by saving the Hoon source to a
file at `%/gen/test/hoon`:

> note about |mount, copy, or edit directly
> 
> note about trailing newlines!!!

```
|=(a/* a)
```

We can then evaluate the generator from the `:dojo`:

```
~your-urbit:dojo> +test 1
1
~your-urbit:dojo> +test [1 2]
[1 2]
```

Let's add some type-safety and restrict our generator to atoms:

```
|=(a/@ a)
```

```
~your-urbit:dojo> +test 1
1
~your-urbit:dojo> +test [1 2]
/~your-urbit/home/~2016.11.1..17.02.53..4f0b/arvo/ford:<[1.411 24].[1.411 38]>
nest-fail
-want.a/@
-have.{@ud @ud}
```

> Reminder: *nest* is a type-compatibility test, checking if one `span` is
> provably a subset of another. The `:dojo` helpfully shows us the span we want
> (our generator's sample mold), and the incompatible span we have.

Our examples are just producing their argument -- let's compute something:

```
|=  a/@                                                 ::  atomic argument
~&  [%arg a]                                            ::  print the arg
`@p`(sham a)                                            ::  produce a hash
```

> Note that we've switched from *wide* to *tall* form Hoon

```
~your-urbit:dojo> +test 1
[%arg 1]
~racdyn-dispyx-todwed-mignus--mapweb-mopbyn-halryg-lonfet
~your-urbit:dojo> +test 'foo'
[%arg 7.303.014]
~darpes-batpec-pasnyt-racsef--labwed-dozlug-sonlup-parhet
```

`++sham` is a standard-library gate that produces the 128-bit hash of an atom.
We're casting that hash to the aura `@p`, the phonetic base used for urbit
ships, tickets, passwords, etc. We're also printing our original argument with
the `~&` hint; this is an interpreter instruction that prints the product of an
expression *as a side-effect* -- it doesn't change the overall computation.

You might find the second invocation a little disorienting if you're not
familiar with Hoon. A full analysis of atoms is out-of-scope here, but it's
worth unpacking a little. An atom is always just a number -- more specifically, it's an unsigned integer of any size with an *aura*, an advisory
type. We've already seen `@ud` (unsigned decimal) and `@p` (phonetic). `'foo'`
is a `cord` (aura `@t`), an *atomic string*, and it indeed equals `7.303.014`
(itself the germanic way of writing 7,303,014 -- we print that because our sample mold is `@`, which prints the same as `@ud`). That number is a pretty
unusual way to represent strings, at least in my experience. A more typical
structure is a `tape`: a linked-list of UTF-8 bytes (`"foo"`). The
relationship between a `cord` and a `tape` is best illustrated in hexadecimal
(aura `@ux`):

```
~your-urbit:dojo> `@`'foo'
7.303.014
~your-urbit:dojo> `@`'abc'
6.513.249
~your-urbit:dojo> `@ux`'abc'
0x63.6261
~your-urbit:dojo> `(list @ux)``(list @)`"abc"
~[0x61 0x62 0x63]
```

##### `mark` and `cask`

So far our generator has produced nouns directly; this is fine for standalone
use, but we'd run into trouble if we wanted to pass our noun to an app. At the
OS level, urbit uses *marks* to tag data with a semantic type. A `mark` is just
a name, ranging from the generic data (`%noun`, `%atom`) to standard formats
(`%json`, `%xml`) to app-specific data structures (`%talk-command`,
`%womb-balance`). The name is a symbolic reference to a formal description of
the type (itself a Hoon file in `%/mar` containing parse/diff/transform/etc.
implementations). A `mark` is similar to a MIME type, or a file extension.

> TODO: should i note that some marked data isn't backed by an actual mark?

"Marked" data is a `++cask`, a cell where the head is the `mark` and the tail
is the data itself. All generators should produce a `++cask`; this is an
informal requirement for "naked" generators, but formally enforced for all others.

`[%noun 0]` is a basic cask.

> TODO: redo examples with casks

> TODO: `:dojo` &mark examples

> TODO: mention app pokes??

##### restrictions

We can embed arbitrarily complex computations inside of our gate, but we still
have some restrictions.

We *always* have to call it with one (and only one) argument:

```
|=  {a/@ b/@}
:-  %noun
[a b]
```

```
~your-urbit:dojo> +test [1 2]
[%noun 1 2]
~your-urbit:dojo> +test 1 2
...
/~your-urbit/home/0/app/dojo:<[611 11].[611 31]>
%one-argument
/~your-urbit/home/0/app/dojo:<[611 28].[611 30]>
[%dojo-lame %made]
~your-urbit:dojo> +test
...
/~your-urbit/home/0/app/dojo:<[611 11].[611 31]>
%one-argument
/~your-urbit/home/0/app/dojo:<[611 28].[611 30]>
[%dojo-lame %made]
```

Additionally, the generator can't tell *where* or when it's being run,
doesn't have access to entropy for pseudorandom or cryptographic
computations, and can't even pretty-print its product.

For any of these use-cases, we'll need a `%say` generator.
