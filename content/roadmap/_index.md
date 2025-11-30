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

## Static Event Dispatch

### Swamp Event Rules

```swamp
on user_defined_rule_name(EventStructType) { // the fields of EventStructType are automatically bound to variables.
// there can be a Context struct as well, but it is optional to include them as a parameter

}
```

```swamp
on user_defined_rule_name(EventStructType) | health > 10 && player == Friendly -> { // the fields of EventStructType are automatically bound to variables
    player.add_score(10)
}
```

```swamp
on user_defined_rule_name(evt: EventStructType) | evt.health > 10 && evt.player == Friendly -> { // explicit name for the event. you access the fields as normal (evt.player, evt.health)
    evt.player.add_score(10)
}
```

### Emit events

```swamp
event_struct := EventStruct { health: 20, team: Team::Blue }

emit(event_struct) // this will send it out to all rules
```

### Lowered

The rules are lowered at compile time to a special function:

```swamp
fn __emit_eventstruct(event_struct: EventStruct, context: Context) {
    // the guards out side the rules are lowered as an if statement
    if event_struct.health > 10 && event_struct.player == Friendly {
        user_defined_rule_name(event_struct) // context wasn't added since it wasn't used in that rule
    }

    host_call(event_struct, context) // the engine might want to process it as well
}
```
