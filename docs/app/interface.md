---
navhome: /app
title: interface
sort: 2
next: true
---

### interface

Apps interact with the Outside by accepting several types of interactions:

- `%poke` -- remote-procedure call
- `%peer` -- subscription
- `%peek` -- one-time request for data (namespace de-reference)

An app registers handlers for these interactions by providing arms of the same
name. These interactions can all be specialized by mark (logo??) or wire.

TODO: explain moves (both apps and vanes?)

(Userpace) moves are always represented as interactions between two apps on
two ships -- for instance app A on ship A *pokes* app B on ship B. This holds
if the apps are on the same ship, or are even the same app -- distributed
interactions are the same as local interactions. An app on a ship is `++gill`:
`(pair ship term)`.

NOTE: src-dab is *NOT* in the bowl!!!

Moves are processed transactionally, with the *sending* gill receiving a
success/failure acknowledgment. This acknowledgement is produced automatically
by `%gall` -- the receiving gill simply crashes to indicate failure, or
produces a list of moves along with the app core (optionally updated) to
indicate success.

For clarity, we'll refer to the gill that initiates an interaction as Alice,
and the target of that interaction as Bob.

##### poke/coup: remote-procedure call

Alice *pokes* Bob. The `%poke` move includes a `wire` (context) and an
arbitrary marked noun.

Bob can accept *any* type of `%poke` with a `++poke` arm -- he'll receive the
payload as a `++cask`: `(pair mark noun)`. 

Here's the simplest possible `++poke`:

```
++  poke
  |=  arg/(cask)
  ~&  arg
  [~  ..poke]
```

Alternately, Bob can restrict the type of accepted `%poke` moves by
*specializing* his `++poke` arm by mark (logo???), in which case the payload
will be typed:

```
++  poke-atom
  |=  arg/@
  ~&  arg
  [~  ..poke]
```

Apps can provide any number of such specializations.

After Bob has finished processing the `%poke`, `%gall` sends Alice a `%coup`
acknowledgement. If Bob crashed while processing the `%poke`, the `%coup`
payload will contain a printable stack trace (`++tang`, which is
`(list tank)`); otherwise, the payload will be null.

`++coup` accepts a wire (the context of the `%poke`) and a nullable payload,
and produces a list of moves and the app core (as usual, optionally updated).

```
++  coup
    |=  {a/wire b/(unit tang)}
    :-  ~
    ?~  b
      ..coup
    ((slog u.b) ..coup)
```

TODO: coup specialized by wire "segment"

##### peer/diff/reap: subscription

`++peer`

Handles incoming subscriptions.

TODO: reap

TODO: diff

##### pull

TODO: quit

##### peek: namespace de-reference

.^/:wish

.^(tape %gx /=test=/foo)

TODO: peer-scry :dojo vs gen case discrepancy(???)


others:

lame???

deal??
onto??

see ++club and ++cuft in zuse:

```
++  club                                                ::  agent action
  $%  {$peel p/mark q/path}                             ::  translated peer
      {$peer p/path}                                    ::  subscribe
      {$poke p/cage}                                    ::  apply
      {$puff p/mark q/noun}                             ::  unchecked poke
      {$pull $~}                                        ::  unsubscribe
      {$punk p/mark q/cage}                             ::  translated poke
      {$pump $~}                                        ::  pump yes+no
  ==                                                    ::
++  cuft                                                ::  internal gift
  $%  {$coup p/(unit tang)}                             ::  poke result
      {$diff p/cage}                                    ::  subscription output
      {$doff p/mark q/noun}                             ::  untyped diff
      {$quit $~}                                        ::  close subscription
      {$reap p/(unit tang)}                             ::  peer result
  ==
```

### `++pull`

Handles dropping subscribers.

TODO: quit??

### `++pour`

Handles responses to `%pass` moves.

### `++park`

Save state on update.

TODO: unimplemented?

### `++prep`

Load state on update.
