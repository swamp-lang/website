+++
title = "Roadmap"
description = "Future plans and development roadmap for Swamp"
[extra]
toc = true
+++

## Modules

### External packages?

```swamp
use @external_package
```

## Iterators

### Inclusive Range

```swamp
0..=10
```

## Traits

### Default trait implementation

### Operator implementation trait

```swamp
impl Add for MyOwnType {
    fn add(self, other: MyOwnType) -> MyOwnType {
        // ...
    }
}
```

## `Self`

Support for **Self** (as an alias for the type itself) in implementation blocks.

## String Escape codes

- `\n` new line escape code
- `\t` tab escape code
- `\"` quotation and single quotation mark escape codes
