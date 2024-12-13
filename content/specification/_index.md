+++
title = "Language Specification"
description = "Detailed specification of the Swamp programming language"
+++

## Comments

Comments help you explain your code. **Swamp** has three types: regular line comments (`//`) for quick notes, block comments (`/* */`) for longer explanations, and documentation comments (`///`) that automatically generate readable documentation for your code. Documentation comments are special because they help others understand how to use your code and appear in tooltips in your editor.

```swamp
// Regular line comments - for implementation notes
player_health = 100      // Start health value

/* Block comments - for longer explanations
   that span multiple lines and can contain
   lists, examples, etc. **Markdown** will be supported in the future. */

/// Documentation comments - generate external documentation
/// These comments should explain the purpose and usage
/// of the following item (struct, function, field, etc.)
struct Player {
    /// The player's current position in the world
    position: Point,
    
    /// Current health points (0-100)
    /// When this reaches 0, the game is over
    health: Int,
    
    /// Movement speed in units per second
    /// Affected by equipment and status effects
    speed: Float,
}
```

## Variables

### Variable Declaration

Declaring a variable in Swamp is simple:

```swamp
player_name = "Hero"
```

You don't need to declare type, it is implied.
Use `snake_case`[^snakecase] for variable names.

When you create a variable, it's immutable by default. This means once you set its value, it can't be changed. If you need to change it later, mark it as mutable with `mut` as you declare it.

```swamp
player_name = "Hero"    // Immutable - cannot be changed
mut health = 100        // Mutable - can be reassigned
health = 101
```

### Variable Reassignment

Only mutable variables can be given new values. This helps prevent accidental changes and makes your code easier to understand.

```swamp
// Mutable variables can be reassigned
mut score = 0       
score = score + 100 
score = score * 2   
```

### Variable Scope and Lifetime

Variables only exist within their scope (the block of code where they're defined). **There are no global variables**, so any variables that you need to access within a function must be given to it as a parameter. However, a variable can be accessed inside a nested scope (such as within an **if** statement or **for** loop).
When the scope ends, the variable is automatically cleaned up.

```swamp
{
    power_up = spawn_powerup()    // Power-up exists in this scope
    if player.collides_with(power_up) {
        mut bonus = 100
        bonus *= 2  // Local multiplication
        player.score += bonus
        {
            {
                mut new bonus cool thing = power_up
            }
        }
    }
    // bonus is not accessible here
}   // power_up is automatically cleaned up here
```

## Functions

### Function Declaration

```swamp
fn my_function(parameter: Type) {
}
```

Functions are declared with ´fn´ followed by the function name (in `snake_case`[^snakecase]) and parameters in parentheses. Each parameter also needs a declared Type (upper `CamelCase`[^camelcase]).

### Functions that Return Values

```swamp
fn add(a: Int, b:int) -> Int {
    a+b
}
```

If the function will return a value, the parameters are followed by a `->`and a Type declaration for the return value. By default, the function will return the *last line* of its declaration.

If you need to, you can write `return` to escape the function with a value before the last line.

```swamp
fn my_function(a: Int, b:int) -> Int {
    if a > b {
        return 100
    }
    a+b
}
```

### Parameter Mutability

Functions can choose whether they want to modify their parameters by using `mut`.This helps make it clear which functions will change the values passed to them and which will just read them.

It is generally recommended to use immutable parameters and return the result, unless there are big types that can cause performance issues.

```swamp
// Mutable parameter example
fn apply_damage(mut target: Entity, damage: Int) {
    target.health -= damage
    if target.health <= 0 {
        target.state = EntityState::Downed
    }
}

// Immutable parameter example
fn calculate_distance(player: Point, target: Point) -> Float {
    dx = target.x - player.x
    dy = target.y - player.y
    (dx * dx + dy * dy).sqrt()
}
```

### Types of Functions

**Swamp** has three types of functions that help you organize your code:

#### Member Functions

These operate on an instance of a type, accessed using a dot notation. They can modify the instance if marked with `mut`.

```swamp
impl Player {
    /// Reduces player health and handles incapacitation
    fn take_damage(mut self, amount: Int) {
        self.health -= amount
        if self.health <= 0 {
            self.state = State::Incapacitated
        }
    }

    /// Calculates distance to target
    fn distance_to(self, target: Point) -> Float {
        dx = target.x - self.position.x
        dy = target.y - self.position.y
        (dx * dx + dy * dy).sqrt()
    }
}

// Usage:
player.take_damage(10)
distance = player.distance_to(enemy.position)
```

#### Associated Function Calls

These belong to the type itself, not instances.
They're called using double colon notation (`::`) and are often used as constructors or utility functions.

```swamp
impl Weapon {
    // Factory method
    fn create_sword() -> Weapon {
        Weapon {
            damage: 10,
            range: 2.0,
            weapon_type: WeaponType::Sword
        }
    }
}

sword = Weapon::create_sword()
```

#### Standalone Functions

In **Swamp**, standalone functions (functions not associated with any type) are rarely used because it's usually better to organize functions as part of a relevant type. However, they can be useful for certain utility operations or when interfacing with system-level features. Or you want to have a short name to call it with.

```swamp
/// Logs a debug message to the console
fn log(message: String) {
    // ... write to console/file
}
```

## Basic Types

**Swamp** provides fundamental types for storing different kinds of data: Integers (Int) for whole numbers, Floating-point numbers (are in fact Fixed Point numbers) for decimal values, Booleans (Bool) for true/false conditions, and Strings (String) for text.

### Integers

```swamp
health = 100     
```

#### Integer Operations

- Add `+`
- Subtract `-`
- Multiply `*`
- Divide `/`
- Remainder `%`
  
#### Integer Comparisons

- Equal `==`
- Not Equal `!=`
- Less Than `<`
- Less or Equal to `<=`
- More Than `>`
- More or Equal to `>=`

### Floats

Floats are always written with one decimal, to keep them apart from Ints.

```swamp
speed = 5.5      
```

#### Float Operations

- Add `+`
- Subtract `-`
- Multiply `*`
- Divide `/`
  
#### Float Comparisons

- Equal `==`
- Not Equal `!=`
- Less Than `<`
- Less or Equal to `<=`
- More Than `>`
- More or Equal to `>=`

### Booleans

```swamp
is_jumping = true 
```

A Boolean can only have two different values,  `true` or `false`.

#### Boolean Operations

- And `&&`
- Or `||`
- Not `!`

### Strings

```swamp
player_name = "Hero" 
```

Strings are written in quotation marks `""`. If you need to use quotation marks within the string, you can use backslashes like this `\"`.

```swamp
dialog = "Guard: \"Stop right there!\""
```

#### String Operations & Member Functions

- `.len()` Returns length (in characters).
- `+` Concatenate two strings into one.

### String Interpolation

String interpolation lets you embed values and expressions directly in your text using curly brackets `{}` in a **single quotation mark** declaration.

```swamp
// Basic interpolation
name = "Hero"
message = 'Welcome, {name}!'
```

Anything within the curly brackets will be handled like regular code: you can include simple variables, complex expressions, and even format them with special modifiers for precise control over how they appear.

```swamp
// Expression interpolation
status = 'HP: {health * 100 / max_health}'
```

#### String Interpolation Formatting

You can specify how the interpolation formats itself using by adding `:` after a variable within the brackets.

- Lowercase hexadecimal `:x`
- Uppercase hexadecimal `:X`
- Binary `:b`
- Floating point precision `:.1f` (number indicates how many decimals to show)
- String padding `:.1s` (number indicates how many digits to show)

```swamp
// Format specifiers
number = 255
hex_lower = 'Item ID: {number:x}'        // "Item ID: ff"
hex_upper = 'Item ID: {number:X}'        // "Item ID: FF"
binary = 'Flags: {number:b}'             // "Flags: 11111111"

// Floating point precision
pi = 3.1415
coords = 'Position: {pi:.2f}'            // "Position: 3.14"

// String padding
score = 12
padded_score = 'Score: {score:.5s}'      // "Score: 00012"
```

## Composite Types

These are more complex types that let you group data together in different ways.

### Arrays

Arrays are ordered lists of items of the **same type**. You can create them, access their elements by position (starting at 0), and modify them if they're mutable.

#### Array Type Declaration

```swamp
fn my_function (my_list: [Int]) {}
```

When declaring a list as a parameter, add square brackets `[]` surrounding the Type that the list will take.

#### Array Member Functions

- Add item to end of list (must have same Type) `.add(item)`
- Remove the item and index `.remove(index)`

#### Array Construction

```swamp
// Initialize positions
spawn_points = [ Point { x: 0, y: 0 }, Point { x: 10, y: 10 }, Point { x: -10, y: 5 } ]
```

#### Array Access

```swamp
waypoints = [ Point { x: 0, y: 0 }, Point { x: 10, y: 10 } ]
next_pos = waypoints[1]
```

#### Array Assignment

```swamp
mut high_scores = [ 100, 95, 90, 85, 80 ]
high_scores[0] = 105
```

### Maps

Maps are collections that store pairs of values, where you use one value (the key) to look up another (the value).

#### Map Declaration

```swamp
fn my_function(my_map: [Key: Value]) {} 
```

A map looks similar to a list, but has two types within the square brackets `[]`. The first type is used as the lookup key.

#### Map Construction

```swamp
// Spawn points for different level names
spawn_points = [
    "Starting Level": Point { x: 0, y: 0 },
    "Second Level": Point { x: 100, y: 50 },   
    "Secret Area": Point { x: -50, y: 75 },  
]
```

Remember that each following Key/Value pair must have the same types as the last.

#### Map Access

```swamp
spawn_points = [ "Starting Level": Point { x: 0, y: 0 }, "Second Level": Point { x: 100, y: 50 } ]
start_pos = spawn_points["Starting Level"]     // Get starting position
```

#### Map Assignment

```swamp
mut spawn_points = [ "Starting Level": Point { x: 0, y: 0 } ]
spawn_points["Starting Level"] = Point { x: 10, y: 10 }    // Update spawn point
```

### Structs

Structs let you create your own data types by grouping related values together.

#### Struct Definition

To define a struct, simply write `struct` followed by the name of your Struct in upper `CamelCase` (as it will become a Type) and some curly brackets `{}`. Inside the brackets, you list each **field** the Struct will contain (and their Types).

```swamp
struct Player {
    position: Point,
    health: Int,
    mana: Int,
    speed: Float
}
```

#### Struct Construction

Once a Struct is defined, you can create a an instance of it. When you do, you have to set **each field** of the Struct to a value.

```swamp
player = Player {
    position: Point { x: 0.0, y: 0.0 },
    health: 100,
    mana: 50,
    speed: 5.0
}
```

#### Struct Field Access

You can access fields like variables, using a period (`struct.field`).

```swamp
// Read field values
current_health = player.health
can_cast = player.mana >= 20
```

#### Struct Field Assignment

Using `mut`, you can assign new values to fields, just like variables.

```swamp
mut player = Player {
    position: Point { x: 0.0, y: 0.0 },
    health: 100,
    mana: 50,
    speed: 5.0
}

// Update fields
player.health -= 10        // Take damage
player.mana -= 20          // Use mana
player.position = Point { x: 10.0, y: 5.0 }  // Move player
```

#### Struct Implementation

Using `impl` you can attach [member functions](#member-functions) to Structs.

```swamp
impl Player {
    /// Handle taking damage and effects
    fn take_damage(mut self, amount: Int) {
        self.health -= amount
        if self.health <= 0 {
            self.health = 0
            self.state = State::Incapacitated
        }
    }
}
```

`impl` can also be used to attach functions used for [associated function calls](#associated-function-calls).

```swamp
impl Player {
    /// Create a new player with default values
    fn new() -> Player {
        Player {
            position: Point { x: 0.0, y: 0.0 },
            health: 100,
            mana: 50,
            speed: 5.0
        }
    }
}
```

### Tuples

Tuples are similar to structs, but they are not constructed as you use them, and do not have Type names or field names. To use a Tuple, write one or more values inside regular parentheses `()`.

```swamp
player_position = (2,1)
```

```swamp
fn get_position() -> (Int, Int) {
    (10, 20)
}

x, y = get_position()
```

### Enums

Enums let you define a Type that can be one of several variants.
Each variant can optionally carry different types of data. They're great for representing things that can be in different states or categories.

#### Enum Definition

To define an Enum, write `enum` followed by its name (in uppercase `CamelCase` as it will become a Type) and a pair of curly brackets `{}`. Inside the brackets, you list each Type the Enum can be.

```swamp
enum Item {
    // Simple variants (no data)
    Gold,
    Key,
    
    // Tuple variants with data
    Weapon(Int, Float),    // damage, range
    Potion(Int),           // healing amount
    
    // Struct variants with named fields
    Armor {
        defense: Int,
        weight: Float,
        durability: Int
    },
}
```

#### Enum Declaration

```swamp
item = Item::Armor { defense: 3, weight: 3.8, durability: 99 }
```

#### Enum Pattern Matching

You can [pattern match](#pattern-matching) an Enum and output different outcomes depending on what Type an Enum.

```swamp
match item {
    // Simple variant matching
    Gold => {
        player.money += 100
    },
    
    Key => open_nearest_door(),
    
    // Tuple variant destructuring
    Weapon(_, range) => {           // Ignore damage
        set_attack_range(range)
    },

    Potion(amount) => {
        player.health += amount
    },
    
    // Struct variant destructuring
    Armor { defense, weight, .. } => {
        if player.strength >= weight {
            equip_armor(defense)
        } else {
            show_message("Too heavy!")
        }
    }
}
```

### Type Aliases

You can make an alias for any Type. This is mostly useful for Tuples, and in most cases you are better of defining a Struct.

```swamp
type MyAlias = (Int, Int)
```

### Optional Types

Optional types handle values that might or might not exist.

#### Type Declaration

```swamp
target: Entity?            // Current target
```

The `?` suffix indicates that these variables might not have a value.

#### Usage Examples with Default Values

{% note(type="unimplemented") %}
Default value operator `??` is not implemented yet.
{% end %}

```swamp
// Using ?? to provide default values
damage = equipped_weapon?.damage ?? 1     // Default to 1 if no weapon
name = target?.name ?? "No Target"        // Default to "No Target" if no target
x = spawn_point?.x ?? 0.0                 // Default to 0.0 if no spawn point
```

#### Optional Binding

```swamp
// Using if to bind and check optionals
if equipped_weapon? {
    // weapon is now bound and available in this scope
    weapon.attack()
}

// Can be used with else
if target = find_nearest_enemy()? {
    target.take_damage(10)
} else {
    player.search_area()
}
```

#### Chaining Optionals

```swamp
guild_name = player.guild?.get_name() ?? "No Guild"

leader_rank = player.guild?.get_leader()?.get_rank() ?? "No Rank"

spell_power = equipped_weapon?.get_enchantment()?.calculate_power() ?? 0
```

## Control Flow

Control flow determines how your program runs. Swamp provides several ways
 to control the flow of your game.

### If Expressions and Statements

In **Swamp**, every block is an expression that returns a value. This means you can use them on the right side of assignments. When an `if` doesn't have an `else` block, the missing path automatically returns Unit `()` (representing "nothing").

#### If Expression

```swamp
// Both paths return Int
damage = if is_critical_hit {
    base_damage * 2    // Returns Int
} else {
    base_damage       // Returns Int
}
```

#### If Expression with Implicit Unit

```swamp
// Type mismatch example
value = if has_powerup {
    100              // Returns Int
}                   // Implicit else returns ()
// 'value' type is unclear: Int or ()
```

### While Loops

While loops keep running their code block as long as a condition is true.

```swamp
mut projectile = spawn_projectile()
while projectile.is_active {
    projectile.update()
    projectile.check_collisions()
}
```

### For Loops

```swamp
// Update all entities
for enemy in enemies {
    enemy.update()
    enemy.check_player_distance()
}

// Update all entities
for id, enemy in map_of_enemies {
    println("Updating enemy {id}")
    enemy.update()
    enemy.check_player_distance()
}


```

### Break and Continue

```swamp
// Find first vulnerable enemy
for enemy in enemies {
    if enemy.has_shield {
        continue  // Skip to next enemy
    }
    if enemy.is_in_range(player) {
        enemy.attack()
        break     // Stop searching
    }
}
```

### Return

```swamp
fn find_health_potion(inventory: Inventory) -> Item? {
    for item in inventory.items {
        if item.is_health_potion() {
            return item    // Early return when found
        }
    }
    none                 // Return none if not found
}
```

## Pattern Matching

Pattern matching is a powerful way to handle different cases in your code. It's like a super-powered if statement that can look inside complex types and handle multiple cases clearly.

### Basic Patterns

```swamp
match game_state {
    Playing => update_game(),
    Paused => show_pause_menu(),
    GameOver => show_final_score(),
    _ => show_main_menu()
}
```

### Multiple Patterns

```swamp
match item {
    // Simple variant matching
    Gold => {
        player.money += 100
    },
    
    // Tuple variant destructuring
    Weapon damage, range => {
        player.equip_weapon(damage, range)
    },
    
    // Struct variant destructuring
    Armor defense, weight => {
        if player.can_carry(weight) {
            player.equip_armor(defense)
        }
    }
}
```

### Literal Patterns

```swamp
// Numeric and string literals
match player.health {
    100 => ui.show_status("Full Health"),
    1..10 => ui.show_status("Low Health!"),
    _ => ui.show_health(player.health),
}

// Tuple patterns with literals and variables
match position {
    0, 0 => player.spawn_at_origin(),
    0, y => player.spawn_at_height(y),
    x, 0 => player.spawn_at_width(x),
    x, y => player.spawn_at(x, y),
}

// Struct patterns with literals
match entity {
    Player health: 100, mana: 100 => ui.show_status("Full Power!"),
    Player health: 0 => {
        player.die()
    },
    Enemy health: 1 => {
        enemy.enter_rage_mode()
    }
}

// Enum patterns with data
match item {
    Gold => {
        player.money += 100
        ui.show_pickup("Gold")
    },
    Weapon 0, _ => ui.show_status("Broken Weapon"),
    Weapon damage, range => player.equip_weapon(damage, range),
    Armor defense: 0 => ui.show_status("Broken Armor"),
}
```

### Guard Patterns

{% note(type="unimplemented") %}
Guard patterns are not implemented yet.
{% end %}

```swamp
match player_state {
    Attacking damage if has_power_up => 
        apply_damage(damage * 2),
    Attacking damage => 
        apply_damage(damage),
    _ => ()
}
```

## Operators

### Binary Operators

```swamp
// Arithmetic: +, -, *, /, %
remaining_health = health - damage

// Logical: &&, ||
can_attack = in_range && has_ammo

// Comparison: ==, !=, <, <=, >, >=
if player.mana >= spell.cost {
    cast_spell(spell)
}

// Range: ..
for frame in 0..animation.frame_count {
    render_frame(frame)
}
```

### Unary Operators

```swamp
// Negation (-)
velocity.x = -velocity.x  // Reverse direction

// Logical NOT (!)
if !inventory.is_full {
    pickup_item()
}
```

### Suffix Operators

```swamp
// Optional unwrap (?)
if equipped_weapon?.can_fire {
    fire_weapon()
}

current_target = find_nearest_enemy()?
```

## Iterable Sequences

**Swamp** provides several ways to work with sequences of values in your game.
You can loop through ranges of numbers, collections of items, or any other sequence using for loops.

### Exclusive Range

```swamp
// Countdown timer
for i in 3..0 {
    display_number(i)
}
```

### Inclusive Range

{% note(type="coming_soon") %}
Inclusive ranges are not implemented yet. Use exclusive ranges for now.
{% end %}

```swamp
for hp in 1..=max_health {
    draw_health_pip(hp)
}
```

### Array Iteration

{% note(type="coming_soon") %}
Array iteration is not implemented yet.
{% end %}

```swamp
for item in inventory {
    item.draw()
}
```

### Map Iteration

{% note(type="coming_soon") %}
Map iteration is not implemented yet.
{% end %}

```swamp
for player_id, score in high_scores {
    display_score(player_id, score)
}
```

## Type Inference

**Swamp** automatically determines types from context, so you rarely need to write them explicitly.

You only need to declare types explicitly when:

- Declaring function parameters and return types
- Creating struct or enum definitions
- When the compiler needs help understanding your intent

The compiler will tell you when explicit types are needed.

[^snakecase]: [Snake_case Wikipedia](https://en.wikipedia.org/wiki/Snake_case)
[^camelcase]: [CamelCase Wikipedia](https://en.wikipedia.org/wiki/Camel_case)
