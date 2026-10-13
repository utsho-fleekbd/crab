# Crab

Crab is a small statically typed language with a Rust interpreter.

```sh
cargo run -p crabi -- path/to/program.crab
cargo run -p crabi
cargo test --workspace
```

In the REPL, end a submission with a blank line. Use `@exit` (`@e`) to exit,
`@syntax_tree` (`@st`) to toggle the syntax tree, and `@bound_tree` (`@bt`) to
toggle the bound tree. EOF executes any pending submission, then exits.

```text
var factorial = 1
for i = 1 to 5 {
    factorial = factorial * i
}
print factorial
```

## Examples

Each program in [example/](example/) runs independently:

```sh
cargo run -p crabi -- example/hello_world.crab
cargo run -p crabi -- example/two_counter_machine.crab
```

| File | Topic |
| --- | --- |
| [syntax.crab](example/syntax.crab) | Syntax reference (comments only) |
| [hello_world.crab](example/hello_world.crab) | Printing and string concatenation |
| [addition.crab](example/addition.crab) | Arithmetic, assignment values, and grouping |
| [conditions.crab](example/conditions.crab) | Conditions and short-circuit logic |
| [scope.crab](example/scope.crab) | Local variables and shadowing |
| [factorial.crab](example/factorial.crab) | Computing 50! |
| [fibonacci.crab](example/fibonacci.crab) | Computing the twentieth Fibonacci number |
| [big_integers.crab](example/big_integers.crab) | Computing 2^256 |
| [gcd.crab](example/gcd.crab) | Euclid's algorithm |
| [game_of_life.crab](example/game_of_life.crab) | Game of Life on an integer-encoded grid |
| [arrays.crab](example/arrays.crab) | Lists, records, dictionaries, and built-ins |
| [insertion_sort.crab](example/insertion_sort.crab) | Sorting with indexed mutation |
| [prime_sieve.crab](example/prime_sieve.crab) | Sieve of Eratosthenes |
| [frequency_count.crab](example/frequency_count.crab) | Word frequencies with a dictionary |
| [game_of_life_arrays.crab](example/game_of_life_arrays.crab) | Game of Life using boolean grids |
| [pathfinding.crab](example/pathfinding.crab) | Breadth-first search and path reconstruction |
| [two_counter_machine.crab](example/two_counter_machine.crab) | Encoded machine interpreter and Turing-completeness construction |

Run Game of Life with `cargo run -p crabi -- example/game_of_life.crab`.
It prints a glider on a 10 by 10 grid for generations 0 through 4. Edit the
size, generation count, and seed condition in the file to try other patterns.
Cells outside the grid are dead; the edges do not wrap.

Change the machine program and starting counters in `two_counter_machine.crab`
to try other computations. With unbounded integers and these machine operations,
Crab can simulate a universal computation model (assuming unlimited memory and
time). See [Dudenhefner's two-counter machine study](https://doi.org/10.4230/LIPIcs.FSCD.2022.16)
for the underlying model.

## Collections

```text
var xs = [10, 20, 30]
xs[1] = 99
var grid = [[false; 10]; 10]
grid[2][3] = true
var person = [name: 'Utsho', age: 25]
person.age = 26
var counts: Map<String, Number> = [:]
counts['crab'] = get(counts, 'crab', 0) + 1
for i in 1..10 { print i }
```

Lists are homogeneous and zero-indexed. Records have fixed field names, with a
separate type for each field; `.name` and `['name']` are equivalent. Record access
requires a literal field name, while dictionaries allow calculated keys. Dictionary
keys are either all integers or all strings, and values share one type. Dot access
also works on string-keyed dictionaries. Keys are never coerced between types.

Optional annotations use `Number`, `Boolean`, `String`, `Range`, `List<T>`, and
`Map<K, V>`. A standalone empty list `[]` or dictionary `[:]` needs an annotation;
assignment to an existing variable, nested literals, and built-in value arguments
can supply the expected type.
`[value; count]` evaluates both expressions once and repeats the value. Negative
counts and sizes beyond addressable memory fail with runtime diagnostics.

Collections have value semantics with copy-on-write storage. Copies and repeated
nested rows behave independently; nested writes update the nearest enclosing
variable. Index expressions run once, left-to-right, followed by the assigned
value; mutation uses the root variable's value after these evaluations. A failed
REPL submission discards nested writes as well as ordinary variable changes.

`start..end` is an inclusive, ascending, lazy integer range. Reversed ranges are
empty. `collect` allocates a list; `len(range)` can report arbitrary-precision
lengths without allocating. `for item in list`, `for item in map`, and
`for item in range` evaluate the iterable once. Lists and dictionaries iterate a
snapshot of their values; use `keys(map)` to iterate dictionary keys.

| Built-in | Behavior |
| --- | --- |
| `len(value)` | List/map/record size, Unicode character count for strings, or range length |
| `push(list, value)` | Return a list with one appended element |
| `pop(list)` | Return `[items: remainingList, value: lastElement]`; empty lists error |
| `keys(collection)` | List indices, dictionary keys, or record field names |
| `values(collection)` | Values as a list; record fields must share one type |
| `has(collection, key)` | Whether a key/index exists; invalid list indices return false |
| `get(collection, key, default)` | Existing value or a default of the same type |
| `remove(collection, key)` | Return an updated collection; list indices shift after removal |
| `collect(range)` | Materialize a range into `List<Number>` |

All built-in arguments, including `get` defaults, evaluate eagerly left-to-right.
Collection built-ins return new values without mutating their arguments. Retain
updates with `xs = push(xs, value)`. Removing a record field changes its structural
type, so bind the result to a new variable. `get` and `remove` require literal keys
on records; `has` permits dynamic string keys. Empty or heterogeneous records cannot
produce a typed `values` list.

Dictionary and record iteration/printing preserve insertion order. A duplicate
dictionary key replaces the earlier value in its original position; duplicate
record fields are compile-time errors. Removing a missing dictionary/record key
is a no-op. Ordinary missing-key reads, invalid list reads/writes/removals, and
empty `pop` operations produce runtime diagnostics. Equality is structural:
record/dictionary entry order does not affect equality, while list order does.
Allocation requests use fallible reservations; actual execution still depends
on available memory and time.

## Language behavior

- Values are signed arbitrary-precision integers, booleans, and strings. Integers
  have no fixed bit-width limit and do not wrap. Division truncates toward zero;
  division by zero produces a runtime error.
- Strings may be empty, contain Unicode, or span multiple lines. Backslashes are
  literal characters; escape sequences are not currently supported.
- `&&` and `||` short-circuit. Assignment expressions return the assigned value.
- Blocks and every `if`, `else`, and loop body introduce lexical scopes, even
  without braces. Assignments can update enclosing variables. A `var` declaration
  replaces a same-scope binding or shadows an enclosing binding.
- `for i = lower to upper` visits an inclusive ascending integer range. Bounds are
  evaluated once. A reversed range performs no iterations. The iterator is local
  to the loop; assigning to it in the body does not alter the iteration sequence.
- Failed REPL submissions discard changes to bindings and variable values. Output
  already printed remains visible.
- File execution returns status 65 for syntax/type errors, 70 for runtime errors,
  and 74 for I/O errors.

## Layout

- `crab`: lexer, parser, typed bound tree, and diagnostics.
- `crabi`: evaluator and command-line REPL.
- `crab/tests`: frontend regression tests.
- `crabi/tests`: interpreter and CLI regression tests, including termination checks.
- `example/`: syntax reference, standalone basic programs, and machine simulation.
