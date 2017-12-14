---
navhome: /docs/gen
title: generators
sort: 2
---

### generators

A *generator* is a limited Hoon "script", producing a value (`noun`) by 
performing calculations on its arguments, prompting for input, or making HTTP
requests. A generator can't otherwise produce moves or perform asynchronous
computations.

##### prerequisites

For this to make sense, you should have some basic familiarity with the *Hoon* programming language, and with `:dojo`, the Urbit shell. You'll also need an urbit instance to try the examples.

- [install urbit](https://urbit.org/docs/using/install/) and [get started](https://urbit.org/docs/using/setup/)
- [Urbytes](https://urbit.org/docs/byte/): a bottom-up introduction to Hoon
- [:dojo (shell)](https://urbit.org/docs/using/shell/) docs
- [%clay (filesystem)](https://urbit.org/docs/using/filesystem/) docs

##### usage

Generators are run from the `gen/` directory of your current `:dojo` desk. To
run the examples in this tutorial, you'll want to copy them to
`%/gen/test/hoon`, or somesuch.

First, `|mount` your desk:

```
your-urbit:dojo> |mount %
```

> Note: `%` is a `:dojo` shortcut for the current `beak`, our ship, our current
> desk, and our current `case` -- a commit or timestamp. It's already
> formatted as a path, so `+ls %/web` does what it's supposed to.

That command mounts your current desk inside your urbit planet directory,
synced both ways, Dropbox style (`./your-urbit/home`, unless you've switched
cases).

There are four types of generators:

<list/>
