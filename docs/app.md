---
navhome: /docs/app
title: apps
sort: 3
---

### apps

An Urbit app is a named, permanent, stateful service. Apps can make Arvo syscalls and interact with other apps by sending (or receiving) typed,
transactional RPC calls, subscriptions, and namespace de-references. They are
permanent; once started they *cannot* be stopped, only upgraded.

An app is a [`:door`](https://urbit.org/docs/hoon/twig/bar-core/cab-door/), a
many-arm'd `core` with a sample. The sample is a cell containg request-specific
metadata and the app state; the arms of the core provide the interface.

Apps are run and managed by the app-server kernel module `%gall`.

#### prerequisites

For this to make sense, you should have some basic familiarity with the *Hoon* programming language, and with `:dojo`, the Urbit shell. You'll also need an urbit instance to try the examples.

- [install urbit](https://urbit.org/docs/using/install/) and [get started](https://urbit.org/docs/using/setup/)
- [Urbytes](https://urbit.org/docs/byte/): a bottom-up introduction to Hoon
- [:dojo (shell)](https://urbit.org/docs/using/shell/) docs
- [%clay (filesystem)](https://urbit.org/docs/using/filesystem/) docs

note: arvo docs

#### usage

Apps are run from the `app/` directory of your current `:dojo` desk. To run
the examples in this tutorial, you'll want to copy them to `%/app/test/hoon`,
or somesuch.

**NOTE:** since apps are *permanent*, using an expendable urbit is highly
recommended. You can use a fake `~zod` (which can't connect to the network) or
comets.

To start a new comet, simply run `bin/urbit -c my-comet`. The [Urbit
Contributors guide](https://github.com/urbit/urbit/blob/master/CONTRIBUTING.md)
has instructions for creating a fake `~zod`.

First, `|mount` your desk:

```
your-urbit:dojo> |mount %
```

> Note: `%` is a dojo shortcut for the current `beak`, our ship, our current
> desk, and our current `case` -- a commit or timestamp. It's already
> formatted as a path, so that something like `+ls %/web` does what it's
> supposed to.

That command mounts your current desk inside your urbit planet directory (at `./your-urbit/home` by default) -- that directory is synced both ways, Dropbox style.

Then, copy in your example app:

```
$ cp example-app.hoon your-urbit/home/app/test.hoon
```

(Or simply open `your-urbit/home/app/test.hoon` directly in your editor of
choice)

Finally, start the app:

```
your-urbit:dojo> |start %test
```

<list/>
