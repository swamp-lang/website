+++
title = "Roadmap"
description = "Future plans and development roadmap for Swamp"
[extra]
toc = true
+++

## Roadmap

- Truly Random Resource Ids. `@8fad1938` or `path/something/else` that looks up the id during compile time.
- Implicit projection (Desugar Embedded) Types
- Shared memory pools (maybe?)
- SoA support

## Resource IDs

- Resource Ids. `@8fad1938` or `path/something/else` that looks up the id during compile time.
- When working in a bigger team, those ids can be randomly generated if needed.

## Remove/erase keywords in collection loops

Be able to safely remove elements from a collection. Compile lowering will know how to change the index to not miss any iterations (if it is swap-removed, stay on the same index)

```swamp
for space_ship in ships {
    if ship.x < -10 {
        remove space_ship // removes the space_ship in a safe way
    }
}
```

## Named function arguments

Be able to specify function arguments by name. Evaluation order: for ordered arguments it is left to right, but for named arguments I guess it is also left to right?

```swamp
fn my_function(health: Int, damage: Int, modifier: Int) {}

my_function(health: 10, modifier: 10, damage: 5) //don't need to remember the order
my_function(10, 5, 10) //if not using the field names, need to be correct order
```

Parser is already handling it and putting it in AST, so it is prepared for later phases.

Suggested by @catnipped

## ZII arguments (rest operator)

can use rest `..` operator in function arguments, both for named and normal arguments

```swamp
my_function(health: 10 ..)
```

```swamp
my_function(10, ..)
```

will be desugared into zero for each argument not specified:

```swamp
my_function(health: 10, modifier: 0, damage: 0)
```

Suggested by @catnipped

## Implicit projection (Desugar Embedded) Types

you don't have to type the name of the contained types, when it can be inferred from the type. There is no overhead at all:

the syntax is far from decided, this is just to communicate the idea. proposed keyword `embed`

```swamp
struct Position {
    x: Int,
    y: Int,
}

struct MovementLogic {
    speed: Int,
    is_jumping: Bool,
}

fn print_position(position: Position) {
    print('pos: {position}')
}

struct Monster {
    embed position: Position,
    embed movement: MovementLogic,
}

monster = Monster { position: Position { x: 10, y: 20 } }

print_position(monster) // this works since it knows the type, is desugared to: `print_position(monster.position)`
```

## Intra-region references

"Same parent idea"

Goal: Allow raw `&T` fields when the compiler can prove the pointee lives long enough and shares the same memory parent (memory region, or allocation in scratch/arena).

- Useful when you want to iterate through things of the same type (e.g. positions). And also if the number of things can vary a lot and you don't want to allocate the worst-case in each type (e.g. storing 8 positions in each Monster)

- Avoid worst-case duplication: Instead of storing multiple Positions inside every Monster “just in case,” store references to a shared positions collection. Memory scales with the actual number of positions, not a per-monster worst case.

- Fast iteration & cache locality: Keep each type in its own contiguous block (e.g., a SoA/SparseSet for positions). You can iterate positions in tight cache-friendly loops while monsters keep raw `&Position` to the ones they need.

- (Almost) Zero-cost access

Intra-region references are real pointers --- no handles, no lookups, no reference counting.
When the compiler can prove safety, dereferencing is as cheap as reading a field.

The only practical cost is the usual cache miss when following a pointer to a different memory area, but there's no runtime overhead or indirection beyond that.
As long as the referenced data is laid out contiguously, locality stays good and performance remains predictable.

```swamp

struct Monster {
    attack_position: &Position, // inserts lookup information (id) to a position. can only be set if compiler can verify that it is secure
    target_position: &Position, // inserts lookup information (id). can only be set if compiler can verify that it is secure
}

struct World {
    monsters: [Monster; 256],
    positions: [Position; 256],
}

impl World {
    fn create(mut self) -> Monster {
        self.monsters[1] = Monster {
            position: &self.positions[23], // this position pointer is safe to store in monster, since they share parent (`self` is allocated in the same contiguos memory region)
        }
    }

    fn update_all_positions_in_game(mut self) {
        for mut pos in self.positions {
            pos.x += 1
        }
    }
}
```

## Shared Memory Pools Pointers (maybe? unsure)

Description:

- A pool is a 'static, non-moving region of T slots (Sparse slots).
- A field like position: Pool(Position)? holds a permanent, non-movable, non-copyable pointer to one slot.
- `remove()`: frees the slot and clears the field to null.

After that, the pool may reuse the slot for someone else.

the syntax is far from decided, this is just to communicate the idea.

```swamp

pool cool_positions: [Position; 512] // the pool is always alive

struct Monster {
    attack_position : Pool(cool_positions) , // internally keeps a pointer to an element in `cool_positions`
    target_position : Pool(cool_positions), // internally keeps a pointer to an element in `cool_positions`
}


fn spawn_a(mut a: Monster) {
    a.position = positions.alloc_slot()   // binds pointer to a.position (until remove() is called)
    a.position.x = 10
}

fn despawn_a(mut a: Monster) {
    a.position.remove() // frees slot, field pointer is set to zero
}

```

## Call Guard

A call guard is a boolean expression that controls whether a function executes. The guard is checked at each call site *before* the function is invoked. If the condition fails, the call is skipped entirely --- no arguments are evaluated, no registers are saved or restored, and the function body never runs.

### Syntax

Guards are declared using the `|` operator after the function parameters:

```swamp
// Only cares about Elite Celestial Wizards
fn enemy_defeated(evt: EnemyDefeated, mut loot: LootTable)
  | evt.enemy == CelestialWizard && evt.is_elite {
    loot.spawn_rare(evt.position, ItemId::AstralGate)
}
```

### Why Use Call Guards?

**Clarity**: The precondition is declared in the function signature, making it immediately obvious when the function will run. No need to dig through the implementation to understand the entry requirements.

**Performance**: Guards are desugared at compile time into conditional checks at each call site. Failed guards skip the function call entirely, eliminating overhead from argument evaluation and register saving/restoring.

### How It Works

At compile time, each call site is desugared into a guarded invocation:

```swamp
// Original call
enemy_defeated(evt, &table)

// Desugared to
if evt.enemy == CelestialWizard && evt.is_elite {
    enemy_defeated(evt, &table)
}
```

This transformation happens at every call site, ensuring zero runtime dispatch overhead.

## Parameter Field Binding

When a struct parameter is declared without a name, its fields are automatically bound as local variables:

```swamp
fn check(PlayerSpawned) {
    // PlayerSpawned fields (health, team, etc.) are directly accessible
    if health > 25 {
        print('strong player with health {health}')
    }
}
```

This is purely syntactic sugar. The function still receives the full struct; the compiler simply generates field access code for you. It's equivalent to:

```swamp
fn check(evt: PlayerSpawned) {
    if evt.health > 25 {
        print('strong player with health {evt.health}')
    }
}
```

## Static Event Dispatch

### Event Rules (Handlers)

Event handlers (also called rules) are defined using the `on` keyword instead of `fn`. The compiler tracks all `on` functions that listen to a specific struct type (the first parameter) and generates a dispatch function for each event type. You don't call event handlers directly --- they're invoked through the generated dispatch function.

Event handlers commonly combine [Call Guards](#call-guard) for conditional execution and [Parameter Field Binding](#parameter-field-binding) for concise field access.

```swamp

enum Team {
    Red
    Blue,
}

struct PlayerSpawned {
    health: Int,
    team: Team,
}

// fields of `PlayerSpawned` are automatically bound to variables
// in almost all cases you need one or more context parameters, but it
// is optional.
on player_spawned(PlayerSpawned) {
    print('a player spawned in with health {health}')
}

// fields of PlayerSpawned` are automatically bound to variables
on healthy_blue_player_joins(PlayerSpawned)
  | health > 10 && team == Blue -> {
    print('a healthy blue player spawned in with health {health}')
}

// explicit name for the event; access fields
// normally (evt.team, evt.health)
on cool_red_player_enters(evt: PlayerSpawned, mut battle: Battle)
  | evt.health > 45 && evt.team == Red -> {
    print('a healthy red player spawned in with health {evt.health}')
    battle.message("strong player joined for team red!")
}
```

### Emit Events

Events are dispatched using the `emit()` macro, which triggers all matching event handlers (rules). The compiler automatically generates the appropriate dispatch function based on the event type.
The emit macro is replaced at compile time with a call to the generated dispatch function, e.g. `__emit_player_spawned(player_spawned, battle)`

```swamp
player_spawned := PlayerSpawned { health: 20, team: Blue }

// sends the event to all matching handlers
// internally this will be replaced with:
// __emit_player_spawned(player_spawned, battle)

emit(player_spawned, battle)
```

### Lowered at Compile Time

At compile time, event handlers are lowered into a specialized dispatch function. Guard expressions become conditional branches, ensuring zero runtime overhead for routing the events --- all dispatch logic is resolved statically.

```swamp
fn __emit_player_spawned(evt: PlayerSpawned, mut battle: Battle) {
    // guards from the rules are lowered to if statements
    if evt.health > 10 && evt.team == Blue  {
        // context omitted since this specific rule doesn't use it
        healthy_blue_player_joins(evt)
    }

    if evt.health > 45 && evt.team == Red {
        // this rule required the battle context
        cool_red_player_enters(evt, &battle)
    }

    some_event_handler_with_no_guard(evt)

    // optional host call for notification
    host_call(evt, context)
}
```

### Intercept any Function

Inspired by @catnipped, this feature enables intercepting *any* function for event dispatch. The function must have a clear first parameter that is a struct (not counting `self`). `self` will serve as an automatic context. A potential issue might be when `mut` is needed by the event handler but not used by the function itself.

At compile time (or when patching the `.swim` file for [Marsh VM](https://marshvm.com/)), a dispatch code block is inserted before the function body.

Mark functions to intercept as events using a keyword. The syntax is not clear at this time, assume it is an `intercept` keyword for now:

```swamp
intercept some_game::logic::Battle::spawn_unit
```

The `spawn_unit()` function is then patched with a dispatch block at the start:

```swamp
fn spawn_unit(mut self, unit_info: UnitInfo) {
    // Injected dispatch block
    {
        if unit_info.type == SpaceWizard && unit_info.faction == Friendly {
            // context parameter omitted since this rule doesn't use it
            user_defined_rule_name(unit_info)
        }

        if unit_info.starting_health > 45 && unit_info.faction == Enemy {
            // this rule requires the self context
            another_user_defined_rule_name(unit_info, self)
        }
    }

    // original code:
    ...
}
```

## Optimization: Optional Aggregates Using Zero Pointer

An optimization for optional aggregate types where the pointer itself serves as the discriminant. Instead of wrapping in a union with a separate tag, a zero pointer represents `none` and a non-zero pointer represents `some`.

```swamp
// Without optimization (wrapped in union)

// is lowered as: Alloc Position?, then set union tag to 1
p: Position? = Position { x : 0, y : 0 }

// is lowered as: Alloc Position?, then set union tag to 0
p: Position? = none

// With optimization (pointer as discriminant)

// is lowered as: Alloc Position, store pointer, no wrapping
p: Position? = Position { x : 0, y : 0 }

// is lowered as: store null pointer (0), no wrapping
p: Position? = none
```

This optimization eliminates wrapping and unwrapping overhead, making `when` expressions and `if` conditionals a lot faster:

```swamp
p: Position? = none

when p { // lowered as: if p != 0, no unwrapping needed

}

if p { // literal pointer check, zero overhead, no unwrapping

}
```

## Optimization: Payload-Free Enums as Scalars

Enums without payloads can be lowered directly to integer primitives (`u8`, `u16`, or `u32`) based on the number of variants, eliminating the need for stack frame allocation.

**Trade-off**: These optimized enums cannot be borrowed because the Swamp ABI passes scalars by value, never indirectly.
