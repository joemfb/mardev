---
navhome: /app
title: state
sort: 1
next: true
---

### state

An app is a `door`, a `core` with a sample. The sample is the app state; for 
obvious reasons, it must be a `cell`.

> Reminder: to run these examples, copy them to `%/app/test/hoon`, or a 
> similar path:
> 
> `$ cp example-app.hoon your-urbit/home/app/test.hoon`
> 
> To start the app, run the following from your `:dojo`
>
> `your-urbit:dojo> |start %test`
> 
> Note: this is only needed the first time; subsequent updates will run
> automatically.

Here's roughly the simplest possible app:

```
|_  ^
++  poke
  |=  a/(cask)
  ~&  a
  [~ ..poke]
--
```

We print our argument (a `cask`, a noun tagged by its `mark`), and we produce
`[~ ..poke]` (the phrase to remember is *a list of moves and our context*).

To interact with this app, poke it from the `:dojo`:

```
your-urbit:dojo> :test 1
[p=%noun q=1]
```

Pokes and other interactions will be examined in detail later -- for now,
we're concerned only with the app state.

##### `bowl`

Our app state, the core sample, is always a `cell` (`^`), but we should be more
specific. The head of the cell is always a `bowl` containing standardized,
event-specific state, while the tail is app-specific. We'll leave the tail
null for now.

> Note: it's a good practice to put a face on the bowl. While some of the
> standard Urbit apps use `hid` for some inscrutable historical reason, we'll
> use the more obvious `bow`.

Our updated app prints the bowl on every `poke`:

```
|_  {bow/bowl state/$~}
++  poke
  |=  a/(cask)
  ~&  bow
  ~&  a
  [~ ..poke]
--
```

But updating our `:test` app fails!

```
gall: %test: prep mismatch
```

We're getting ahead of ourselves! `%gall` is protecting us from
indescriminately throwing away our existing app state. If we had
pre-existing state, we'd want to convert it to our new state mold in the
`++prep` arm -- for now, we'll just explicitly throw it away:

```
|_  {bow/bowl state/$~}
++  poke
  |=  a/(cask)
  ~&  bow
  ~&  a
  [~ ..poke]
::
++  prep
  |=  *
  [~ ..prep]
--
```

> Note: starting the app with a non-conforming sample would produce
> the following error message:
>
> `test: bogus core`

Poking our app now prints the `bowl`:

```
your-urbit:dojo> :test 1
[ [our=~zod src=~zod dap=%test]
  [wex={} sup={}]
  ost=3
  act=30
  eny=0v12f.g2iro.46nt6.8dehr.mljj8.r2flv.es5p6.7rq8q.kmuhm.ljgqq.vljc8.n7g6r.cod1v.n7ce4.mo351.buigb.f662s.sr9km.dii0k.4kd09.34ntm
  now=~2016.10.5..05.09.46..b69e
  byk=[p=~zod q=%home r=[%da p=~2016.10.5..05.09.43..e014]]
]
[p=%noun q=1]
```

There's a lot to unpack here. Let's start with the `bowl` mold definition,
from `%zuse`:

```
++  bowl                                                ::  standard app state
        $:  $:  our/ship                                ::  host
                src/ship                                ::  guest
                dap/term                                ::  agent
            ==                                          ::  
            $:  wex/boat                                ::  outgoing subs
                sup/bitt                                ::  incoming subs
            ==                                          ::
            $:  ost/bone                                ::  opaque cause
                act/@ud                                 ::  change number
                eny/@uvJ                                ::  entropy
                now/@da                                 ::  current time
                byk/beak                                ::  load source
        ==  ==                                          ::
```

- `our` is us, our current ship
- `src` is the ship from which our current event originated (also us, since we poked `:test`)
- `dap` is our app name, `%test`
- `wex` is the apps that we've subscribed to
- `sup` is the apps that have subscribed to us
- `ost` is the numeric identifier of the `duct` representing the current event
- `act` is TODO ???
- `eny` is entropy
- `now` is the current timestamp
- `byk` is the `beak` from which we are running

> Note: the last three faces form the standard generator sample.

We'll use this stuff later, when we're producing or responding to moves.

#####  updating state

Let's start keeping some custom app state: the set of `cask`s we've
been poked with:

```
|_  {bow/bowl state/(set (cask))}
++  poke
  |=  a/(cask)
  ~&  [%poked-with state]
  ~&  [%new a]
  :-  ~
  ..poke(state (~(put in state) a))
::
++  prep
  |=  *
  [~ ..prep]
--
```

```
your-urbit:dojo> :test 1
[%poked-with {}]
[%new p=%noun q=1]

your-urbit:dojo> :test 2
[%poked-with {[p=%noun q=1]}]
[%new p=%noun q=2]

your-urbit:dojo> :test 1
[%poked-with {[p=%noun q=1] [p=%noun q=2]}]
[%new p=%noun q=1]

your-urbit:dojo> :test 3
[%poked-with {[p=%noun q=1] [p=%noun q=2]}]
[%new p=%noun q=3]

your-urbit:dojo> :test 1
[%poked-with {[p=%noun q=1] [p=%noun q=3] [p=%noun q=2]}]
[%new p=%noun q=1]
```

=(. (|=(* +>)))

TODO: abet

TODO: prep

