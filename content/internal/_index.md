+++
title = "Language Internals"
description = "Detailed specification of the Swamp programming language"
[extra]
toc = true
+++

## Initializer List

A continous area of elements of the same type.

```swamp
[10, -4, 20]
```

## Initializer Pair List

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
    1 -> {
        foo()
        bar()
    }
    2 -> {
        baz()
    }
}
```

### Explicit Delimiters

If a construct has no natural delimiters, you must either:

  1. Add a comma after EVERY item (including the last one, even if it is only one item), or

  2. Wrap the content in a scope `{...}`

  3. For fixed-size structures (like struct fields, tuples), trailing comma is optional.

### Memory-Safe Semantic Confusion (Identity confusion)

**Guarantees (Always):**

- ✅ Memory safe --- no buffer overflows, no out-of-bounds writes
- ✅ No dangling pointers --- references can't outlive their memory
- ✅ No wild pointers --- references can't point to arbitrary/uninitialized memory
- ✅ No process corruption --- can't write outside your process space
- ✅ Correct alignment --- types maintain their alignment requirements
- ✅ No crashes or panics from memory errors

**Possible Issues (And can *only* happen within a scoped borrow):**

- ⚠️ Logic errors --- reading/writing data that's semantically wrong
- ⚠️ Identity confusion --- reference points to different logical entity than expected
- ⚠️ Stale data --- seeing old values after logical deletion
- ⚠️ But still deterministic --- bugs are reproducible, not random

Every reference in Swamp is constructed through controlled binding expressions (`with`, `when`, `loops`), so you literally cannot create a wild pointer. You can't:

- Cast an integer to a pointer
- Take an address of arbitrary memory
- Create uninitialized pointers
- Have null pointers
