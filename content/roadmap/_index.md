+++
title = "Roadmap"
description = "Future plans and development roadmap for Swamp"
[extra]
toc = true
+++

## Modules

### `use` statement

```swamp
use external_module
use external_module::{my_function, MyStruct}
use external_module::*
use external_module::{my_function as my_function_alias, MyStruct as MyStructAlias}
```

### `mod` statement

```swamp
mod local_module
```


## Iterators

### Inclusive Range

```swamp
0..=10
```

### Pipes `<|` and `|>` ?
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
