+++
title = "Roadmap"
description = "Future plans and development roadmap for Swamp"
+++

- [ ] `use` statement.

```swamp
use external_module
use external_module::{my_function, MyStruct}
use external_module::*
use external_module::{my_function as my_function_alias, MyStruct as MyStructAlias}
```

- [ ] `mod` statement.

```swamp
mod local_module
```

- [ ] Add support for inclusive ranges.

```swamp
0..=10
```

## Iterators

- [ ] Inclusive Range. `0..=10`
- [ ] Arrays
- [ ] Maps. Key value pairs.

- [ ] Pipes `<|` and `|>` ?
- [ ] Operator implementation trait

```swamp
impl Add for MyOwnType {
    fn add(self, other: MyOwnType) -> MyOwnType {
        // ...
    }
}
```

- [ ] support for `Self` (as an alias for the type itself) in implementation blocks.

## Strings

- [ ] `\n` new line escape code
- [ ] `\t` tab escape code
- [ ] `\"` quotation and single quotation mark escape codes
