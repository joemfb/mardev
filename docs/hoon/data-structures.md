---
title: Common data structures
navhome: /hoon
sort: 6
---

### Common data structures

*coming soon*

#### higher-order mold

a la generic type

#### standard cell faces

lone/pair/trel/quad

#### unit

#### list

`list` is a higher-order mold for a null-terminated linked-list. (It's "higher-ordered" because it takes another
mold as its sample, like a generic type.)


A `list` is a null-terminated and recursive; it's either null (empty) or a
cell with a head of the first *list **i**tem* (`i`) and a **t**ail (`t`)
containing the rest of the list, recursively.

an empty list of atoms:

```
> ^-((list @) ~)
~
```

a list containing one atom (note that it has to be null-terminated):

```
> ^-((list @) [1 ~])
~[1]
```

a list containing two atoms:

```
> ^-((list @) [1 2 ~])
~[1 2]
```

This [`:loop`]() crashes (`!!`) if the list is null ([`:ifno`]()), produces the
singular list item `i` if the tail is null, or produces a cell with a
head of the first list item and a tail of *the same operation applied to the*
*rest of the list*. In other words, it takes a null-terminated structure of
arbitrary length an converts it to a non-terminated, fixed-length structure.

Let's test it out. First, assign our list to a `:dojo` variable:

```
> =p ^-((list @) [1 2 ~])
```

Then, convert the macro from *tall* to *wide* form (see [syntax]() below) and
evaluate that:

```
> |-(?~(p !! ?~(t.p i.p [i.p $(p t.p)])))
[1 2]
```

This is a trivial example, producing a trivial result. In practice, this macro
is evaluated against the AST of the arbitrary number of sub-twigs in a `:conp`
expression.

Note: the [`:ifno` (`?~`)]() branches in the macro are mandated by Hoon's
type-safe compiler. We can't "reference" any of the faces of the list until we
know that it's not null:

```
> i.p
-find.i.p
find-fork-d
> ?~(p ~ i.p)
1
```

#### set

#### map

