+++
title = "The Swamp Way"
description = "Idioms, best practices, and philosophy for writing Swamp code."
[extra]
toc = true
+++

## Prefer Single Assignment

Avoid:

```swamp
mut a = 0
if something {
  a = 4
} else {
  a = 5
}
```

Prefer:

```swamp
a = if something 4 else 5
```

Avoid:

```swamp
mut direction = Vec2 { .. }
if input == Input::Left {
    direction = Vec2::new(-1, 0)
} else if input == Input::Right {
    direction = Vec2::new(1, 0)
} else {
    direction = Vec2::new(0, 0)
}
```

Prefer:

```swamp
direction = match input {
    Left  -> Vec2::new(-1, 0),
    Right -> Vec2::new(1, 0),
    _     -> Vec2::new(0, 0),
}
```

{% note(type="principle") %} One assignment tells the whole story — mutation
muddies it. {% end %}

## ZII = Zero Is Initialization

> Zero should always be safe. Zero means nothing, and nothing is safe.

Embrace zero. The value `0`, `none`, or the _first_ variant of an enum
(discriminant is zero for the first) should always be safe and well-defined.
This makes it easy to check for "empty" or "non-existent" without introducing
optional types.

The benefit is not just simpler code, but also leaner code generation --- the
compiler doesn't need to emit extra instructions for wrapping and unwrapping
optionals.

Avoid:

```swamp
struct Avatar {
  id: Int,
}

fn find_avatar(distance: Int) -> Avatar? {
    ...
}


avatar = find_avatar(20)
when avatar {
    render_avatar(avatar)
}
```

Prefer:

```swamp
struct Avatar {
  id: Int,
}

// id will be zero if avatar is not found
fn find_avatar(distance: Int) -> Avatar {
    ...
}

avatar = find_avatar(20)
if avatar.id != 0 {
    render_avatar(avatar)
}
```

### Enums and ZII

The first variant of an enum is always the zero value --- and it should always
be safe. This way, an uninitialized or default value never causes harm.

```swamp
enum Action {
  Idle,
  DrinkingPotion,
}

fn avatar_do_action(avatar: Avatar, action: Action) {
    match action {
        Idle -> {} // nothing "unsafe" happens
        DrinkingPotion -> avatar.health += 100,
    }
}
```

### When Zero Can't Mean "Nothing"

Sometimes zero is a valid value: coordinates, vectors, matrices, and other
numeric types where 0 has real meaning. In these cases, you cannot reserve zero
to signal "empty."

For these types, use an optional (`T?`) instead.

## Use "rest" `..` Operator

> Fill in what matters, zero the rest.

When creating struct literals, the `..` operator automatically sets all
unspecified fields to their zero value. This makes code shorter, clearer, and
consistent with the ZII principle.

Instead of manually filling in every field, you can focus only on the values
that matter --- the rest are guaranteed safe because zero is safe.

Avoid (explicit zeroes):

```swamp
struct Player {
    id: Int,
    score: Int,
    gold: Int,
    last_checkpoint: Int,
}

player = Player {
    id: 42,
    score: 0,
    gold: 0,
    last_checkpoint: 0,
}
```

Prefer (rest operator):

```swamp
struct Player {
    id: Int,
    score: Int,
    gold: Int,
    last_checkpoint: Int,
}

player = Player {
    id: 42,
    ..
}
```

## Only Use up to Four Parameters

> Functions should be simple to call and easy to read.

Functions with many parameters are confusing: the order is easy to mix up, the
meaning of each argument is unclear, and the code becomes harder to maintain.

There's also a **performance cost**:

- Each parameter must be stored to a temporary register, moved into ABI order,
  and sometimes copied back for scalars.
- On many platforms, when the number of parameters grows, extra ones must be
  spilled to the stack, introducing memory traffic and alignment adjustments.
- With a ready struct, the overhead is minimal --- just passing the pointer to
  the struct.

Instead, if you must have more than four arguments, group related values into a
struct and pass that as a single argument. This improves readability, makes the
code self-documenting, and lets you extend later without breaking call sites.

Functions with many parameters are harder to use correctly. Research suggests
developers struggle with more than four parameters [RamaKak2013], and psychology
shows that people can juggle around four items in working memory [Cowan2010].

Avoid:

```swamp
fn draw_avatar(x: Int, y: Int, rotation: Float, scale: Float, opacity: Float) {
    ...
}

draw_avatar(120, 200, 0.0, 1.0, 0.8)
```

Prefer:

```swamp
struct DrawParams {
    x: Int,
    y: Int,
    rotation: Float,
    scale: Float,
    opacity: Float,
}

fn draw_avatar(params: DrawParams) {
    ...
}

draw_avatar(DrawParams {
    x: 120,
    y: 200,
    rotation: 0.0,
    scale: 1.0,
    opacity: 0.8,
})
```

## Use Anonymous Structs for Rare Cases

> Don't name what you only use once.

If a group of fields is only used in one or a few places, there's no need to
define a named struct for it.

Anonymous structs are perfect for bundling values together for a single call
site or a temporary grouping.

```swamp
fn draw_special_button(special_params: {
    width: Int,
    color: Color,
    tab_count: Int,
} )


draw_special_button( {
    width: 10,
    color: Color::new(1.0, 0.8, 0.8),
    tab_count: 1,
} )
```

## Prefer Scopes or `with` Blocks Over Single-Use Functions

> Functions are for reuse. Scopes are for structure (and speed).

It's good practice to group related code into scopes `{}` or `with` blocks for
readability. But don't create named functions that are only ever called once or
twice. If the code isn't reused, keep it inline --- it makes the flow easier to
follow and avoids cluttering your code with unnecessary names.

Single-use helpers force the compiler to juggle calling conventions: shuffle
args into temps, reorder for the ABI, push/pop callee-saved registers, emit
prologue/epilogue, branch to `call` and back on `ret`. Inline scopes skip all of
that, yielding fewer instructions and better cache/branch behavior.

Inline code is easier to refactor. You don't have to update function signatures
or chase down call sites when adding or removing a local variable.

Avoid:

```swamp
fn draw_main_menu() {
    draw_new_game_button() // new_game_button will only be called from here
    draw_quit_game_button() // quit_game_button will only be called from here
}
```

Prefer:

```swamp
fn draw_main_menu(gfx: Gfx) {
    // New Game Button
    {
        draw_button_border(gfx, BLUE)
        draw_text("new game")
    }

    // Quit Game Button
    {
        draw_button_border(gfx, GREEN)
        draw_text("quit game")
    }
}
```

or

```swamp
fn draw_main_menu() {
    // New Game Button
    with gfx {
        draw_button_border(gfx, BLUE)
        draw_text("new game")
    }

    // Quit Game Button
    with gfx {
        draw_button_border(gfx, GREEN)
        draw_text("quit game")
    }
}
```

## Use Associated Functions

> Keep behavior with the type it belongs to.

When a function conceptually belongs to a type, make it an associated function
(`impl`) instead of a free function.

This has two big advantages:

- **Discoverability:** Code completion and intellisense will show all available
  functions when you type value and a dot `.`. You don't have to remember global
  function names.

- **Documentation:** The impl block becomes the single place to look when you
  want to know what operations a type supports.

Avoid (free function)

```swamp
fn length_of_name(p: Player) -> Int {
    p.name.len()
}
```

Prefer:

```swamp
impl Player {
    fn name_length(self) -> Int {
        self.name.len()
    }
}
```

## Do Not Use Strings

> Strings are for people, not for game code.

In game code you should rarely use strings. Strings are very heavy: they take
more memory, require extra allocations, and slow down comparisons.

Worse, strings lose information. If you turn structured data (like an ID or a
health value) into a string, you throw away its type and meaning. Getting it
back requires error-prone parsing (never ever do that) --- and usually ends up
slower and less flexible.

Use identifiers, enums, or numeric handles instead. Strings should only appear
**as late as possible** in rendering --- that way you can support localization,
keep your game code lightweight, and preserve information in its structured
form.

- **Memory:** numbers and enums are _far_ cheaper to store than strings.

- **Performance**: string comparisons and hashing are _much_ slower than
  integers.

- **Information**: strings flatten structured data and throw away semantics.

- **Flexibility**: separating data from presentation is a good general rule
  anyway, and makes localization and content changes easy.

{% note(type="principle") %} Strings belong at the edge of your game —-- for the
player, not the game code. Inside the game, keep information structured.
{% end %}

**Wrong:**

```swamp
if action == "DRINK_POTION" {
    avatar.health += 100
}
```

**Right:**

```swamp
enum Action {
    Idle,
    DrinkPotion,
}

if action == Action::DrinkPotion {
    avatar.health += 100
}
```

Sometimes:

There are only a few valid uses of strings in Swamp code:

- **Debugging and logging:**

```swamp
info('loading level {level_id}')
debug('new game was selected')
print("starting boss fight")
```

- **Storing player names:** e.g. names coming from external services (Steam,
  PSN, etc.).

- **Assertions:**

```swamp
assert(avatar.velocity < 1000, "Avatar unreasonable velocity")
```

- **Presentation edge:**

when rendering localized, interpolated strings to the player

```swamp
enum LocalizedStringId {
    StartGame,
    AreYouSure,
}

enum Language {
    Swedish,
    English,
}

fn get_localized_string(
    language: Language,
    id: LocalizedStringId,
    game: Game) -> String {
}
```

- **Parsing text formats**

Parsing text formats should usually be done by the engine, not in Swamp --- but
there are times when it's necessary.

Everywhere else: keep your data structured.

## Do not use Defensive Coding

> A hidden bug is a delayed disaster.

Do not accept faulty states or "fix" them silently by clamping, resetting, or
ignoring invalid values. This only hides the real bug and makes it harder to
track down.

Instead, fail fast and loud with asserts. During development, asserts will catch
invalid states immediately. In release builds, they can be stripped
automatically, so they don't cost performance.

**Wrong (defensive coding):**

```swamp
fn update_avatar(mut avatar: Avatar) {
    if avatar.gold < 0 {
        avatar.gold = 0 // silently fix
    }
}
```

**Right:**

```swamp
fn update_avatar(mut avatar: Avatar) {
    assert(avatar.gold >= 0, "Gold amount should never be negative")
}
```

{% note(type="principle") %} Bugs should surface at the moment they happen, not
be hidden under "safe defaults."{% end %}

### When Clamping Is Okay

Clamping or handling wrong states are valid in two situations:

1. At the edges of the game --- when input comes from outside sources you cannot
   fully control (e.g. input, hardware, or network).

2. As part of the simulation rules --- when the game design itself defines a
   hard limit.

```swamp
// Movement speed cannot exceed max velocity by design
avatar.velocity = avatar.velocity.clamp(0, MAX_VELOCITY)
```

## Compile time over Runtime

> Decide as early as possible.

Swamp is designed for determinism and performance. When a value can be known at
compile time, prefer to make it a constant instead of computing or looking it up
at runtime.

This leads to:

**Performance:** fewer runtime instructions.

**Predictability:** no hidden work during simulation.

**Simplicity:** easier to reason about when values never change.

Avoid:

```swamp
fn update_physics() {
    gravity = 9 * 1000 / 60  // recalculated every frame
    velocity += gravity
}
```

Prefer (compile-time constant):

```swamp
const GRAVITY_PER_TICK = 150

fn update_physics() {
    velocity += GRAVITY_PER_TICK
}
```

## Code Is Data

If you know the data ahead of time, bake it directly into the code as constants
instead of loading from a file.

Avoid (runtime loading):

```swamp
fn load_cards() -> [Card] {
    read_json_file("cards.json")
}
```

Prefer (compile time):

```swamp
struct Card {
    id: Int,
    name: LocalizedStringId,
    power: Int,
}

const CARDS = [
    Card { id: 1, name: LocalizedStringId::Fireball, power: 5 },
    Card { id: 2, name: LocalizedStringId::Healing,  power: 3 },
]
```

Here the entire card library is in constant memory. No runtime parsing, no file
I/O, no indirection --- just direct access to data that never changes.

{% note(type="principle") %} If you already know it, compile it in. Runtime is
for the unknown. {% end %}

## Prefer Type Inference

> Infer more, clutter less.

Swamp's type inference makes code shorter, cleaner, and easier to read. Explicit
types are only needed when type cannot be inferred.

Avoid:

```swamp
player: Player = Player::new()
score: Int = 0
```

Prefer (inferred):

```swamp
player = Player::new()
score = 0
```

Sometimes:

Optionals and other cases where a value has multiple meanings may need explicit
annotation.

```swamp
maybe_score: Int? = 0
```

Here, zero is wrapped in `Some`, making the type explicit.

{% note(type="principle") %} Infer the obvious, state the meaningful. {% end %}

## Return Values Instead of Mutating Parameters

> Builders should build --- not patch.

If a function's purpose is to initialize a value, make it return that value.
Don't take an existing variable as a `mut` parameter and fill it in.

This makes such functions easy to use directly inside struct initializers and
expressions, where mutation isn't possible. It also improves performance:
returning writes the result directly into the struct field (via sret), avoiding
a separate variable and the extra store/"copy" into the field.

**Avoid (mutating parameter):**

```swamp
fn create_spawn_from_id(mut pos: Position, id: Int) {
    pos = Position { x: id, y: 0 }
}

a = Avatar {
    position: {
        mut p = Position {..}
        create_spawn_from_id(&p, 42)
        p
    },
}
```

**Prefer (returning the value):**

```swamp
fn create_spawn_from_id(id: Int) -> Position {
    Position { x: id, y: 0 }
}

a = Avatar {
    position: create_spawn_from_id(42),
}
```

## Use Tuples for Small, Self-Explanatory Groups

For small, short-lived groups where order is obvious, prefer a tuple.
Tuples avoid boilerplate and keep code concise.

If the grouping is reused, or if field names add clarity, use a struct instead.

**Avoid:**

```swamp
struct Coords {
    x: Int,
    y: Int,
}

fn move_avatar(delta: Coords) { ... }

move_avatar(Coords { x: 5, y: -3 })
```

**Prefer (tuple):**

```swamp
fn move_avatar(delta: (Int, Int)) { ... }

move_avatar((5, -3))
```

## References

[RamaKak2013]:
  https://engineering.purdue.edu/RVL/Publications/RamaKakAPIQ_SPE.pdf
  'Rama, G., & Kak, A. (2013). Some Structural Measures of API Usability. Software: Practice and Experience, 43(12), 1529–1555.'
[Cowan2010]:
  https://pmc.ncbi.nlm.nih.gov/articles/PMC2864034/
  'Cowan, N. (2010). The Magical Mystery Four: How Is Working Memory Capacity Limited, and Why? Current Directions in Psychological Science, 19(1), 51–57.'
