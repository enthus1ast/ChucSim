# Sim 

![Build Status (github)](https://github.com/ba0f3/sim.nim/workflows/Build/badge.svg) 
[![made-with-nim](https://img.shields.io/badge/Made%20with-Nim-ffc200.svg)](https://nim-lang.org/)

A type-safe configuration file parser for Nim that converts INI files into strongly-typed objects. Define your config structure as Nim objects and let Sim handle the parsing and validation.

## Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Supported Types](#supported-types)
- [Advanced Features](#advanced-features)
  - [Nested Objects](#nested-objects)
  - [Sequences and Wildcards](#sequences-and-wildcards)
  - [Inheritance](#inheritance)
  - [Pragmas](#pragmas)
- [API Reference](#api-reference)
- [Examples](#examples)
- [Requirements](#requirements)
- [About](#about)
- [Support](#support)

## Features

✨ **Type-Safe Parsing**: Automatically convert INI files to strongly-typed Nim objects  
🔄 **Automatic Type Conversion**: Support for numbers, floats, booleans, chars, strings, enums, and sequences  
🌲 **Nested Objects**: Multi-level nested configuration sections  
🧬 **Inheritance**: Share common configuration between sections  
🎯 **Wildcards**: Easy pattern matching for repeated sections  
🏷️ **Pragmas**: Custom parsing behavior with `@defaultValue`, `@ignore`, and `@parseHook`  
📐 **Snake Case**: Automatic conversion from CamelCase to snake_case

## Installation

```shell
nimble install sim
```

## Quick Start

**config.ini:**
```ini
name = "Nim 101"
year = 2019
students = "student*"

[student1]
name = John
age = 18

[student2]
name = Peter
age = 20
```

**example.nim:**
```nim
import sim

type
  Student = object
    name: string
    age: int

  Course = object
    name: string
    year: int
    students: seq[Student]

var cfg = loadObject[Course]("config.ini")
echo &"Welcome to {cfg.name}, we have {cfg.students.len} students registered"
```

**Output:**
```
Welcome to Nim 101, we have 2 students registered
```

## Supported Types

Sim supports the following data types:

| Type | Example Values | Notes |
|------|---------------|-------|
| `int`, `int8`-`int64` | `42`, `-10` | All integer types supported |
| `uint`, `uint8`-`uint64` | `100`, `255` | Unsigned integers |
| `float`, `float32`, `float64` | `3.14`, `-0.5` | Floating point numbers |
| `bool` | `true`, `false`, `on`, `off`, `1`, `0` | Multiple formats supported |
| `char` | `a`, `Z` | Single character (first char of string) |
| `string` | `"Hello World"` | Text values |
| `enum` | By name or numeric value | Custom enumerations |
| `seq[T]` | Lists of any supported type | See [Sequences](#sequences-and-wildcards) |
| `object` | Nested structures | See [Nested Objects](#nested-objects) |

**Note:** Config sections and keys must use `snake_case` naming convention.

## Advanced Features

### Nested Objects

Create multi-level configuration structures:

```ini
[main]
foo = "foo"
baz = "baz"

[main/sub]
foobaz = "foobaz"

[main/sub/level2]
deeply_nested = "value"
```

```nim
type
  Level2 = object
    deeplyNested: string
  
  SubConfig = object
    foobaz: string
    level2: Level2
  
  MainConfig = object
    foo: string
    baz: string
    sub: SubConfig
```

### Sequences and Wildcards

Define lists of values or objects using wildcards:

```ini
# Explicit list
students = "student1,student2"

# Or use wildcards (matches all sections starting with "student")
students = "student*"

[student1]
name = John
age = 18

[student2]
name = Peter
age = 20
```

```nim
type
  Student = object
    name: string
    age: int
  
  Config = object
    students: seq[Student]
```

### Inheritance

Share common configuration values between sections:

```ini
children = "child*"

[dad]
fname = "William"
lname = "Smith"
age = 34
gender = M

[child1.dad]
fname = "Bob"
age = 11

[child2.child1]
fname = "John"
```

In this example:
- `child1` inherits from `dad` (gets `lname = "Smith"`, `gender = M`)
- `child2` inherits from `child1` (gets all values from `child1` and transitively from `dad`)
- Both children override specific fields while inheriting others

### Pragmas

Customize parsing behavior with pragmas:

#### `@defaultValue` - Set Default Values

```nim
type
  Config = object
    port {.defaultValue: 8080.}: int
    host {.defaultValue: "localhost".}: string
```

If `port` or `host` are missing from the config file, the default values will be used.

#### `@ignore` - Skip Fields

```nim
type
  Config = object
    name: string
    computed {.ignore.}: string  # Won't be parsed from config
```

#### `@parseHook` - Custom Parsing

```nim
proc parseUpperCase(s: string): string =
  result = s.toUpperAscii()

type
  Config = object
    name {.parseHook: parseUpperCase.}: string
```

## API Reference

### `loadObject[T](filename: string, useSnakeCase = true): T`

Load and parse a configuration file into an object of type `T`.

**Parameters:**
- `filename`: Path to the INI configuration file
- `useSnakeCase`: Convert CamelCase field names to snake_case (default: `true`)

**Returns:** Parsed object of type `T`

**Raises:**
- `SimParsingException`: Base exception for all parsing errors
- `KeyNotFoundException`: When a required key is not found
- `SectionNotFoundException`: When a required section is not found
- `InvalidValueException`: When a value cannot be converted to the target type

### `loadObject[T](stream: Stream, useSnakeCase = true, filename = "[stream]"): T`

Load and parse configuration from a stream.

**Parameters:**
- `stream`: Input stream containing INI data
- `useSnakeCase`: Convert CamelCase field names to snake_case (default: `true`)
- `filename`: Filename for error messages (default: `"[stream]"`)

## Examples

See the `tests/` directory for comprehensive examples covering:
- Basic type conversion
- Nested objects
- Sequence handling with wildcards
- Inheritance between sections
- Custom parsing with pragmas
- Error handling

## Requirements

- Nim >= 1.6.0

## About

### Why the name "Sim"?

Named after my third daughter ❤️

## Support

If you find this library useful, consider supporting its development:

**Donate:** [PayPal](https://paypal.me/ba0f3)

---

## License

MIT License - see LICENSE file for details

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
