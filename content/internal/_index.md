+++
title = "Language Internals"
description = "Detailed specification of the Swamp programming language"
[extra]
toc = true
+++

## Slice

A continous area of elements of the same type.

```swamp
[10, -4, 20]
```

## SlicePair

A continous area of elements of the same interleaved key- and value-type.

```swamp
[10:"Something", -4:"Another", 20:"Third"]
```

## Delimiters

### Natural Delimiters

If a construct has a natural delimiter (like | in guards or `{...}` in blocks), no additional delimiter is needed:

```swamp
// Guards use | as natural delimiter

guard =
    | x > 0 -> foo()
    | x < 0 -> bar()
    | _ -> baz()


// Blocks use {...} as natural delimiter

match x {
    1 => {
        foo()
        bar()
    }
    2 => {
        baz()
    }
}
```

### Explicit Delimiters

If a construct has no natural delimiters, you must either:

  1. Add a comma after EVERY item (including the last one, even if it is only one item), or

  2. Wrap the content in a scope `{...}`

  3. For fixed-size structures (like struct fields, tuples), trailing comma is optional.
