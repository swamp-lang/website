+++
title = "Swamp Documentation"
description = "Official documentation for the Swamp programming language"
sort_by = "weight"
+++

**Swamp** is a compact, deterministic systems language for **games and real-time servers**.

It uses short, clear syntax, explicit memory with **compile-time allocation**, and fully predictable execution.

Swamp code compiles to **Marsh VM** bytecode and, in the future, directly to **native machine code**.

## Key ideas

- **Explicit control.** Nothing is hidden --- memory, lifetimes, and layout are defined by the code, not by a runtime.

- **Compile-time allocation.** All memory sizes are fixed at build time, giving total predictability and eliminating runtime allocation and fragmentation.

- **Strong static typing.** No garbage collection --- safety through structure and compile-time validation, ensuring consistent behavior and memory integrity.

- **Scoped borrowing.** Simple, explicit ownership rules for safe, compile-time memory management.

- **Zero-cost data types.** Built-in aggregates such as enums with payloads, optionals, and collections like `Sparse<T; N>` or `Map<T; N>` map directly to memory layouts with no runtime overhead.

- **Deterministic by design.** Every instruction and branch is reproducible — ideal for lockstep, rollback, and replay-based systems.

- **Hot reloading.** Designed for fast iteration --- Swamp compiles quickly to compact **Marsh VM** opcodes that can be reloaded in-place without losing state, enabling rapid gameplay and logic experimentation.

- **Compact syntax.** Minimal, explicit, and readable at the instruction level.

## Quick Examples

### Functions and String Interpolation

```swamp
fn say_hello(name: String) -> String {
    'Hello, {name}!'
}

print(say_hello("Ossian"))  // Outputs: "Hello, Ossian!"
```

### Pattern Matching

```swamp
enum Result {
    Ok(String),
    Err(String),
}

fn process(result: Result) {
    match result {
        Ok(message) -> print('Success: {message}'),
        Err(error) -> print('Error: {error}')
    }
}
```

### Structs and Member functions

```swamp
struct Point {
    x: Float,
    y: Float,
}

impl Point {
    fn squared_distance(self, other: Point) -> Float {
        ((self.x - other.x) * (self.x - other.x) +
         (self.y - other.y) * (self.y - other.y))
    }
}

p1 = Point { x: 0.0, y: 0.0 }
p2 = Point { x: 3.0, y: 4.0 }

print('Distance: {p1.squared_distance(p2)}')
```

### Mutable Variables

In Swamp, variables are immutable by default and arguments are passed by value (copied when passed to functions).
This design encourages immutability for safer, more predictable code. To make a variable mutable, use the `mut` keyword:

```swamp
mut x = 10
x = 20  // Now you can change the value of x
```

 When passing a mutable variable to a function, you must explicitly mark it as mutable at the call site, even if the variable was already declared as mutable:

```swamp
fn increment(mut x: Int) {
    x += 1
}

mut a = 5 // a will be changed in the future

// even though a is already mutable, we want to make
// it explicit that we allow and see the function might change it.

increment(mut a)

print(a)  // Outputs: 6
```
