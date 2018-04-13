---
navhome: /app
title: console
sort: 3
next: true
---

### console

basic do-nothing app:

```
!:                                                      ::
::::  sample console app                                ::
  ::                                                    ::
|_  {bow/bowl $~}
::                                                      ::  ++poke-noun
++  poke-noun                                           ::  generic input
  |=  a/*
  ^-  (quip move +>.$)
  ~&  a
  [~ +>.$]
--
```

`++quip` is a generic mold for apps: `(quip a b)` is `{(list a) _b}`

start and poke

```
~your-urbit:dojo> |start %test
activated app home/test
~your-urbit:dojo> :test 1
10
```

now, subscribe, and log subscribers:

```
!:                                                      ::
::::  sample console app                                ::
  ::                                                    ::
|_  {bow/bowl $~}
::                                                      ::  ++poke-noun
++  poke-noun                                           ::  generic input
  |=  a/*
  ^-  (quip move +>.$)
  ~&  [%poke a]
  ~&  sup.bow
  [~ +>.$]
::                                                      ::  ++peer
++  peer                                                ::  generic subscribe
  |=  a/*
  ^-  (quip move +>.$)
  ~&  [%peer a]
  ~&  sup.bow
  [~ +>.$]
--
```

poke

```
~your-urbit:dojo> :test 1
[%poke 1]
{}
```

link

```
~your-urbit:dojo> |link %test
[linked to [p=~zod q=%test]]
```

and dumped into an empty prompt

ctrl-x back to `:dojo`

```
~your-urbit:dojo> :test 1
[%poke 1]
{[p=2 q=[p=~zod q=/sole]]}
```

ctrl-x back to `:test` (empty prompt), hit RETURN

```
[unlinked from [p=~zod q=%test]]
gall: %test: no poke arm for sole-action
[%drum-coup-fail ~zod 1 p=~zod q=%test]
```

poke again

[%poke 1]
{[p=2 q=[p=~zod q=/sole]]}

BUG!!!!

autolink WTF???

```
> |unlink %test
>=
[%poke 1]
{[p=2 q=[p=~zod q=/sole]]}
> :test 1
>=
[%peer [1.701.605.235 0]]
{[p=2 q=[p=~zod q=/sole]]}
> |link %test
>=
[linked to [p=~zod q=%test]]
[%poke 1]
{[p=2 q=[p=~zod q=/sole]]}
> :test 1
>=
> |unlink %test
>=
[unlinked from [p=~zod q=%test]]
[%poke 1]
{}
> :test 1
```

TODO: peer-sole

lets fix that error:

```
/-  sole                                                ::  import sur/sole
!:                                                      ::
::::  sample console app                                ::
  ::                                                    ::
|_  {bow/bowl $~}
::                                                      ::  ++poke-noun
++  poke-noun                                           ::  generic input
  |=  a/*
  ^-  (quip move +>.$)
  ~&  [%poke a]
  ~&  sup.bow
  [~ +>.$]
::                                                      ::  ++poke-sole-action
++  poke-sole-action                                    ::  console input
  |=  act/sole-action:sole
  ^-  (quip move +>.$)
  ~&  act
  [~ +>.$]
::                                                      ::  ++peer
++  peer                                                ::  generic subscribe
  |=  a/*
  ^-  (quip move +>.$)
  ~&  [%peer a]
  ~&  sup.bow
  [~ +>.$]
--
```


```
~your-urbit:dojo> |link %test
[linked to [p=~zod q=%test]]
```

```
[%ret ~]
[%det ler=[own=0 his=0] haw=0v4.7j7tl.qs19r.9fa4k.3tc3j.r825p ted=[%mor p=~[[%ins p=0 q=~-a]]]]
[%det ler=[own=0 his=1] haw=0vrrjd9.u7ur7.gq7ok.oea2v.mirno ted=[%mor p=~[[%ins p=1 q=~-b]]]]
[%det ler=[own=0 his=2] haw=0v5.i8k78.ms04l.hna4p.nfsbt.jv3i0 ted=[%mor p=~[[%ins p=2 q=~-c]]]]
[%det ler=[own=0 his=3] haw=0v5.urc9d.5582e.473l2.mt29n.n1vop ted=[%mor p=~[[%ins p=3 q=~-d]]]]
abcd
```



TODO: sole-share state *has* to be synced!