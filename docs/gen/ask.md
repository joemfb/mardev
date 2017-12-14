---
navhome: /gen
title: %ask
sort: 3
next: true
---

### `%ask`

An `%ask` generator is a *prompt*; it synthesizes a noun from its environment,
arguments, and interactive input.

`%ask` uses the same environment, argument molds, and calling conventions as
`%say`, and also produces a `++cask`. Unlike `%say`, that cask must be wrapped
in `sole-result`, a *console structure* which includes prompt, status, and
error-reporting metadata. `++sole-result`, along with many other console
structures, is defined in `%/sur/sole/hoon`.

Here's a simple example to demonstrate the boilerplate without prompting:

```
/-  sole                                                ::  import sur/sole
:-  %ask                                                ::
|=  *                                                   ::  ignore env/args
^-  (sole-result:sole (cask *))                         ::  cast product
(sole-so:sole [%noun [1 2]])                            ::  produce result
```

```
~your-urbit:dojo> +test
q=[1 2]
```

Ignore the `q` face on the produce, we'll get to that shortly...

The biggest change with `%ask` generators is that our product is not the final
product; we're producing a structure with several levels of intermediate
metadata, to manage the incremental process of presenting a prompt and
accepting input.

> Note: skip to the bottom if you just want working examples.

##### `++sole-result`: `%ask` structures

```
++  sole-result                                         ::  conditional result
  |*  out/$-(* *)                                       ::  output structure
  $@(@ud (sole-product out))                            ::  error position
::                                                      ::
++  sole-product                                        ::  success result
  |*  out/$-(* *)                                       ::
  %+  pair  (list tank)                                 ::  
  %+  each  (unit out)                                  ::  ~ is abort
  (pair sole-prompt (sole-dialog out))                  ::  ask and continue
```

The first thing to note is that these are all high-order molds, that is, molds
that are themselves parameterized by molds (analogous to generic types in other
languages). `$-(* *)` is the mold for a gate accepting any noun, producing any
noun, i.e., it's the mold for any mold. `++sole-result` is a fork between an
atom (`@ud`) and a `++sole-product`, which is a pair of printable output
(`(list tank)`) and ... something.

`++each` is a loobean-tagged fork: given `(each a b)`, if the head is `&`, the
tale is `a`, if the head is `|`, the tail is `b`. In our case it's a fork
between `(unit out)` and `(pair sole-prompt (sole-dialog out))`. `++unit` is
a *nullable*: either `$~` or `{$~ u/out}`.

So we have a fork between an *error-position* and a `pair` of printables
and a fork between nullable output and other stuff... Thankfully, we have some
gates to help us build up this structure:

> TODO: cask, sole-result, high-order molds, tradeoffs ...

- `++sole-so`: produce successful result
- `++sole-no`: produce empty result
- `++sole-yo`: print printable (such as prompt instructions)
- `++sole-lo`: set the prompt
- `++sole-go`: parse input

Let's give 'em a whirl in `:dojo`:

First, import `%/sur/sole/hoon`:

```
~your-urbit:dojo> /-  sole
```

You can see that a reference to that structure has been stored in the `:dojo`
subject:

```
~your-urbit:dojo> .
[ [ sur=[[%sole ~] ~]
    our=~your-urbit
    now=~2016.12.3..07.38.26..5de9
    eny=0v13u.ndar1.r0b2m.fqgop.omrn1.p7pjj.77a2j.1lj5c.6o5sr.ltd4d.eosa7.6c2iv.a0856.45it3.6p9mg.r9d8u.qk93h.6cj87.55jpa.27fua.dem6u
  ]
  ~
  <1.hza 288.dqq 42.esl 409.vxt 110.xht 1.ztu $151>
]
```

> Note: `/-` is a `%ford` rune -- a *preprocesser directive*. `%ford` is the
> Arvo build system -- when it "imports" a Hoon library or structure, it
> compiles the import and appends it to the subject.
> 
> Since `:dojo` is an interactive shell, it has to deal with statefulness not
> present in normal compilation. It does so by storing references to imported
> structures in the `sur` variable. You can clear `sur` (or any `:dojo`)
> variable by evaluating `=sur` (the same is true of `lib`, set by `/+`).

##### `++sole-so`: successful output

`++sole-so` is a gate that accepts a noun (TODO: should be a cask) and
produces that noun, wrapped in `++sole-result`:

```
~your-urbit:dojo> (sole-so:sole [%atom 1])
[p=~ q=[%.y p=[~ u=[%atom 1]]]]
```

This is our success case: no error-position, no printables, just our output.
The head of our `pair` (`p`) is null (`~`), an empty list of printables. The
tail (`q`) is a cell, with a head of `%.y` (another way to write `&`) and a
tail of `[~ u=[%atom 1]]` -- a non-null `unit` wrapping our `cask`:
`[%atom 1]`.

We can store our `++sole-result` in a `:dojo` variable, and then deconstruct
it by face or geometry:

```
~your-urbit:dojo> =a (sole-so:sole [%atom 1])
~your-urbit:dojo> p.a
~
~your-urbit:dojo> q.a
[%.y p=[~ u=[%atom 1]]]
~your-urbit:dojo> p.q.a
[~ u=[%atom 1]]
~your-urbit:dojo> -:a
p=~
~your-urbit:dojo> +:a
q=[%.y p=[~ u=[%atom 1]]]
~your-urbit:dojo> +<:a
%.y
~your-urbit:dojo> +>:a
p=[~ u=[%atom 1]]
~your-urbit:dojo> +>+:a
u=[%atom 1]
```

<blockquote>
This is a good opportunity to examine the *lark* addressing syntax in Hoon. A
cell is a binary tree of nouns, and we can navigate it to arbitrary depths
through a sequence of head/tail selections. `-` and `<` mean *head*, `+` and
`>` mean tail: we alternate for visual clarity.

```
~your-urbit:dojo> -:[[0 1] [2 3]]
[0 1]
~your-urbit:dojo> +:[[0 1] [2 3]]
[2 3]
~your-urbit:dojo> -<:[[0 1] [2 3]]
0
~your-urbit:dojo> ->:[[0 1] [2 3]]
1
~your-urbit:dojo> +<:[[0 1] [2 3]]
2
~your-urbit:dojo> +>:[[0 1] [2 3]]
3
~your-urbit:dojo> -:"abc"
i='a'
~your-urbit:dojo> +<:"abc"
i='b'
~your-urbit:dojo> +>-:"abc"
i='c'
```
</blockquote>

##### `++sole-no`: empty output

`++sole-no` is an arm that produces a constant empty result. It's not a gate:
it produces a constant noun and therefore has no need for a sample.

```
~your-urbit:dojo> sole-no:sole
[p=~ q=[%.y ~]]
```

We have the empty list of printables as our head, and a similar cell as our
tail. The head of the tail is still `&`, representing output, but the tail
of the tail is a null unit (`~`).

TODO: reword "output"

TODO: programmatic failure?

TODO: not a cask!!!

TODO: %dy-meal-cage

> TODO: test if `~` is ignored in a poke

##### `++sole-yo`: intermediate printable

`++sole-yo` is a gate that combines a `tank` (a single printable) and a
`++sole-result`:

```
~your-urbit:dojo> (sole-yo:sole [leaf+"foo"] sole-no:sole)
[p=[i=[%leaf "foo"] t=~] q=[%.y ~]]
~your-urbit:dojo> (sole-yo:sole [leaf+"foo"] (sole-so:sole [%atom 1]))
[p=[i=[%leaf "foo"] t=~] q=[%.y p=[~ u=[%atom 1]]]]
```

This is the exact same structure we've seen before, but now we have printable
list. `++list` is itself a higher-order mold: a recursive fork that's either
null or a cell of a list item (`i`) and the rest of the list (`t`).

Our final `cask` is still sitting at `+>+`:

```
~your-urbit:dojo> +>+:(sole-yo:sole [leaf+"foo"] (sole-so:sole [%atom 1]))
u=[%atom 1]
```

##### `++sole-lo`: prompt

Now we get to the main event: interactive input!

`++sole-lo` accepts a `++sole-prompt` and a gate that produces a
`++sole-result`. It produces an intermediary `++sole-result` from those, so
it's a type of deferred recursive structure. The `++sole-prompt` sets the
actual `:dojo` prompt, while the gate parses the input and produces the final
`++sole-result`.

```
~your-urbit:dojo> =a (sole-lo:sole [& %foo "bar"] |=(a/tape sole-no:sole))
~your-urbit:dojo> a
[p=~ q=[%.n p=[%.y %foo "bar"] q=<1.xfd {a/"" {sur/{{@tas $~} $~} our/@p now/@da eny/@uvJ} $~ <1.hza 288.dqq 42.esl 409.vxt 110.xht 1.ztu $151>}>]]
```

Here we have the same empty list of printables as our head, but a very
different structure as our tail. The head of our tail is `%.n` (another way of
writing `|`), so we're in the second fork of the `++sole-result` `each` mold:
`(pair sole-prompt (sole-dialog out))`.

```
~your-urbit:dojo> p.a
~
~your-urbit:dojo> q.a
[%.n p=[%.y %foo "bar"] q=<1.xfd {a/"" {sur/{{@tas $~} $~} our/@p now/@da eny/@uvJ} $~ <1.hza 288.dqq 42.esl 409.vxt 110.xht 1.ztu $151>}>]
~your-urbit:dojo> -.q.a
%.n
~your-urbit:dojo> p.q.a
[%.y %foo "bar"]
~your-urbit:dojo> q.q.a
<1.xfd {a/"" {sur/{{@tas $~} $~} our/@p now/@da eny/@uvJ} $~ <1.hza 288.dqq 42.esl 409.vxt 110.xht 1.ztu $151>}>
```

When the `:dojo` encounters this structure, it resets the prompt, waits for
input and then calls our `sole-dialog` gate with the input -- a `tape`. We can
simulate that process by just calling the gate:

```
~your-urbit:dojo> (q.q.a "foo")
[p=~ q=[%.y ~]]
```

In fact, that gate is called on every keypress! And intermediate produces are
*discarded*, only the final one (produced after RETURN, ie, enter) is actually
produced by the generator. This allows us to perform *input validation*.

Recall that `++sole-result` is a fork between an atom (`@ud`) and the
`++sole-product` structures we've been examining. That atom is an *error position* -- a 0-based index into our input. When the `:dojo` receives an
error position, it moves the cursor to the position and blocks further input
until the error has been resolved.

TODO: confirm ^

Let's write a gate that rejects any input containing a lowercase "z":

```
~your-urbit:dojo> =x |=  a/tape
                     =+  i=(find "z" a)
                     ?^(i u.i (sole-so:sole [%noun a]))
~your-urbit:dojo> (x "abc")
[p=~ q=[%.y p=[~ u=[%noun "abc"]]]]
~your-urbit:dojo> (x "abcz")
3
```

Back at `p.q.a`, we have our `++sole-prompt`:

```
~your-urbit:dojo> p.q.a
[%.y %foo "bar"]
> `sole-prompt:sole`p.q.a
[vis=%.y tag=%foo cad=~['b' 'a' 'r']]
```

Here's the mold definition:

```
++  sole-prompt                                         ::  prompt definition
  $:  vis/?                                             ::  command visible
      tag/term                                          ::  history mode
      cad/styx                                          ::  caption
  ==                                                    ::
```

As best I can tell, `tag` is entirely ignored, so we'll just use the empty
symbol `%$` (`term` is an alias for `@tas`) and focus on the other two. `vis`
toggles a hidden prompt (`&` is visible, `|` is hidden), and `cad` is the
actual prompt text (`styx` is *styled text* -- a superset of `tape`, which
we'll use for now).

`[& %$ "foo: "]` renders as

```
~your-urbit:dojo: foo: INPUT HERE
```

`[| %$ "bar: "]` renders as

```
~your-urbit:dojo: bar: <~hash> *******
```

You can hit *backspace* to exit a prompt and return to the `:dojo` at any time.

##### `++sole-go`: parse input

TODO: parser combinators, error position

invoked directly:

```
> ((star low) [0 0] "foo")
[p=[p=0 q=3] q=[~ [p="foo" q=[p=[p=0 q=3] q=""]]]]
> ((star low) [0 0] "fooZ")
[p=[p=0 q=3] q=[~ [p="foo" q=[p=[p=0 q=3] q="Z"]]]]
```

produces an `edge`, or in this case `(like tape)`

with scan:

```
> (scan "foo" (star low))
"foo"
> (scan "fooZ" (star low))
{1 4}
'syntax-error'
ford: build failed ~[/g/~zod/use/dojo/~zod/inn/hand /g/~zod/use/hood/~zod/out/dojo/drum/phat/~zod/dojo /d //term/1]
```

with `++sole-go`

```
> ((sole-go:sole (star low) |=(a/tape a)) "foo")
"foo"
> ((sole-go:sole (star low) |=(a/tape a)) "fooZ")
3
```

parsing idioms:

- `++alf`: alphabetic
- `++aln`: alphanumeric
- `++alp`: alphanumeric and -
- `++prn`: printable (excludes control characters)

parsing composers:

- `++star`: match 0-or-more times

boss constructs an atom?

```
> (;~(pfix (star lus) (star alf)) [0 0] "+foo")
[p=[p=0 q=4] q=[~ u=[p="foo" q=[p=[p=0 q=4] q=""]]]]
> (;~(pfix (star lus) (star alf)) [0 0] "+++foo")
[p=[p=0 q=6] q=[~ u=[p="foo" q=[p=[p=0 q=6] q=""]]]]
> (;~(pfix (star lus) (star alf)) [0 0] "+++foo+")
[p=[p=0 q=6] q=[~ u=[p="foo" q=[p=[p=0 q=6] q="+"]]]]
```

##### examples

Prompts for *name*, then prints it:

```
/-  sole                                                ::
:-  %ask                                                ::
|=  *                                                   ::
%+  sole-yo:sole                                        ::  print instructions
  leaf+"enter your name"                                ::
%+  sole-lo:sole                                        ::  prompt for input
  [& %$ "name: "]                                       ::
|=  a/tape                                              ::  process input
%-  sole-so:sole                                        ::  produce result
:-  %tang
[leaf+"your name is '{a}'"]~
```

Prompts for *name* and *password* (hidden), then prints them:

```
/-  sole                                                ::
:-  %ask                                                ::
|=  *                                                   ::
%+  sole-yo:sole  leaf+"enter your name"                ::
%+  sole-lo:sole  [& %$ "name: "]                       ::  prompt 1
|=  a/tape                                              ::  process input 1
%+  sole-yo:sole  leaf+"enter your password"            ::
%+  sole-lo:sole  [| %$ "password: "]                   ::  prompt 2 (hidden)
|=  b/tape                                              ::  process input 2
%-  sole-so:sole                                        ::  produce result
:~  %tang
  leaf+"your password is '{b}'"
  leaf+"your name is '{a}'"
  leaf+~
==
```

TODO: with error reporting
TODO: branch on something for sole-no/sole-yo, crash?

TODO: with parser combinators

TODO: with args

TODO: with args helper

*generator arguments are defined as `++sole-args` in `sur/sole/hoon`*
