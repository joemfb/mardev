---
navhome: /gen
title: %say
sort: 2
next: true
---

### `%say`

Most generators are `%say` generators -- they produce a noun from their
arguments, non-interactively. That's the same thing "naked" generators do, but
`%say` removes most of their restrictions, at the cost of a minor increase in
complexity.

A `%say`, generator is a cell of `[%say gate]`, where the gate *must* produce
a `++cask`, a cell containing a noun tagged by its `mark`.

> Reminder: to run these examples, copy them to `%/gen/test/hoon`, or a 
> similar path:
> 
> `$ cp example-gen.hoon ~your-urbit/home/gen/test.hoon`

##### no arguments

The first advantage of a `%say` generator is that it can be called without an
argument:

```
:-  %say
|=  *
[%noun [0 1]]
```

```
~your-urbit:dojo> +test
[0 1]
```

> Note: If your generator produces a `noun` that isn't a `cask` (ie, isn't
> tagged), `:dojo` will print the error `%dojo-lame %made`.

##### pretty-print

The next advantage is pretty-printing: since the product of a `%say` generator 
*must* be a `cask`, `:dojo` can provide special handling for known marks. One
such `mark` is `tang`, a list of `tank` printables:

```
:-  %say
|=  *
[%tang [%leaf "foo"]~]
```

```
~your-urbit:dojo> +test
foo
```

##### environment

Those gates all have a sample mold of `*` (any noun) and don't use their arguments in any way. However, the actual sample they are called with contains three nouns:

- a triple of the current time, 64 bytes of entropy, and the current `++beak` (ship, desk, and case)
- a list of arguments
- a list of named arguments

Of course, `*` matches that. A more specific mold for 0-arguments is a `cell`, followed by two nulls (empty lists):

```
{^ $~ $~}
```

The full 0-argument sample mold includes faces for the environment triple. Here's a generator that prints its environment, each face on its own line:

```
:-  %say                                            ::  a %say generator
|=  {{now/@da eny/@uvJ bec/beak} $~ $~}             ::  with zero arguments
:-  %tang                                           ::  producing a tang
%-  flop  %-  limo                                  ::  reversed
:~  leaf+"timestamp: {<now>}"                       ::  (tangs print bottom-up)
    leaf+"entropy: {<eny>}"                         ::
    leaf+"ship: {<p.bec>}"                          ::  a+b is [%a b]
    leaf+"desk: {<q.bec>}"                          ::  "{}" is interpolation
    leaf+"case: {<r.bec>}"                          ::  <> creates a tape
==                                                  ::
```

```
timestamp: ~2016.12.1..06.16.40..c1c2
entropy: 0v2en.k7t53.u0v9o.v9e8a.q5a5e.korqi.8q8a0.htd3t.s0jhh.akf77.i8rd2.osfaj.2abmq.sd3k3.p6qm9.9em4s.gdhi1.o9627.hnk2j.bhil1.k5hhv
ship: ~your-urbit
desk: %home
case: [%da p=~2016.12.1..06.16.40..c1c2]
```

##### arguments

Generators inherit type-safety from Hoon; invoking that last generator with an argument fails with a specific error:

```
~your-urbit:dojo> +test %foo
nest-fail
-want.{{now/@da eny/@uvJ bec/{p/@p q/@tas r/?({$da p/@da} {$tas p/@tas} {$ud p/@ud})}} $~ $~}
-have.{{now/@da eny/@uvJ bec/{p/@p q/@tas r/?({$da p/@da} {$tas p/@tas} {$ud p/@ud})}} {$foo $~} $~}
```

We're only interested in the end of those lines, since the environment triple is the same in every case:

```
-want.{... $~ $~}
-have.{... {$foo $~} $~}
```

Combine that with our original 0-argument mold, and we have the mold for an argument-accepting generator:

```
:-  %say
|=  {^ {arg/$foo $~} $~}
[%noun arg]
```

```
~your-urbit:dojo> +test %foo
%foo
```

Of course, the mold `$foo` only matches `%foo`:

```
~your-urbit:dojo> +test %bar
nest-fail
-want.{... {arg/$foo $~} $~}
-have.{... {$bar $~} $~}
```

We can use the symbol mold (aura `@tas`) to accept any symbol:

```
:-  %say
|=  {^ {arg/@tas $~} $~}
[%noun arg]
```

```
~your-urbit:dojo> +test %bar
%bar
```

##### optional arguments

To make `arg` optional, we need to craft a mold that accepts it or the empty list, and then handle the case where it's empty:

```
:-  %say
|=  {^ args/?($~ {arg/@tas $~}) $~}
:-  %noun
?~(args %foo arg.args)
```

```
~your-urbit:dojo> +test
%foo
~your-urbit:dojo> +test %bar
%bar
```

If we also want to use faces from our environment, we'll need the full argument mold. It's pretty long, so we'll switch to the regular mold form (`$:`):

```
:-  %say
|=  $:  {now/@da eny/@uvJ bec/beak}
        args/?($~ {arg/@tas $~})
        $~
    ==
:-  %tang
=+  arg=?~(args %foo arg.args)
:~  leaf+<arg>
    leaf+"{<p.bec>} says: "
==
```

```
~your-urbit:dojo> +test
~your-urbit says:
%foo
~your-urbit:dojo> +test %bar
~your-urbit says:
%bar
```

##### named arguments

Another approach for optional arguments is to use *named* arguments, the third slot in our argument mold.

```
:-  %say
|=  {^ $~ arg/@tas}
:-  %noun
?~(arg %null arg)
```

That's much simpler than our earlier optional argument -- the tradeoff is that named arguments require a different calling convention:

```
~your-urbit:dojo> +test
%null
~your-urbit:dojo> +test, =arg %foo
%foo
```

They compose quite easily:

```
:-  %say
|=  {^ $~ arg1/@tas arg2/@tas}
:-  %noun
?:  &(?=($~ arg1) ?=($~ arg2))
  %null
?~  arg1  arg2
?~  arg2  arg1
[arg1 arg2]
```

```
~your-urbit:dojo> +test
%null
~your-urbit:dojo> +test, =arg1 %foo
%foo
~your-urbit:dojo> +test, =arg2 %bar
%bar
~your-urbit:dojo> +test, =arg1 %foo, =arg2 %bar
[%foo %bar]
```

And, of course, can be combined with our environment and normal arguments:

```
:-  %say
|=  $:  {now/@da eny/@uvJ bec/beak}
        {arg/@tas $~}
        {ship/? desk/? case/?}
    ==
:-  %tang
%-  flop  %-  limo
;:  welp
    :~  leaf+"arg: {<arg>}"
        leaf+"timestamp: {<now>}"
        leaf+"entropy: {<eny>}"
    ==
    ?.(ship ~ [leaf+"ship: {<p.bec>}"]~)
    ?.(desk ~ [leaf+"desk: {<q.bec>}"]~)
    ?.(case ~ [leaf+"case: {<r.bec>}"]~)
==
```

> Note that `ship/?` is a loobean: `&` is *yes*, `|` is *no*. `ship`, `desk`,
> and `case` will all be *yes* by default -- bonus points if you can figure
> out why ... ;)

```
~your-urbit:dojo> +test %foo
arg: %foo
timestamp: ~2016.12.1..06.16.40..c1c2
entropy: 0v2en.k7t53.u0v9o.v9e8a.q5a5e.korqi.8q8a0.htd3t.s0jhh.akf77.i8rd2.osfaj.2abmq.sd3k3.p6qm9.9em4s.gdhi1.o9627.hnk2j.bhil1.k5hhv
ship: ~your-urbit
desk: %home
case: [%da p=~2016.12.1..06.16.40..c1c2]

~your-urbit:dojo> +test %foo, =ship |, =desk |, =case |
arg: %foo
timestamp: ~your-urbit
entropy: 0v2en.k7t53.u0v9o.v9e8a.q5a5e.korqi.8q8a0.htd3t.s0jhh.akf77.i8rd2.osfaj.2abmq.sd3k3.p6qm9.9em4s.gdhi1.o9627.hnk2j.bhil1.k5hhv
```
