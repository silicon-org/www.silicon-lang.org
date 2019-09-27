# Silicon Language Specification

## Syntax

### Comments

- `//` Regular code comments
- `///` Documentation comments

### Literals

- `ddd` signed, unsized, integer literal
- `No0...` sized, unsigned, octal literal
- `Nb0...` sized, unsigned, binary literal
- `Nx0...` sized, unsigned, hexadecimal literal
- `true` | `false` (has type `Bit`)

## Types

- `()`: Unit type
- `Bit`: One bit primitive
- `SInt<N>` and `UInt<N>`: Signed (two's complement) and unsigned integer representations with size N
- `Int`: Unsized signed integer, does not map to hardware; can only be used to generate hardware
- `[T; N]`: Array of type `T` of length `N`
- `struct`s and `enum`s: User defined types
- `Type`: Denotes a type.
- `(T1, ..., Tn)`: Denotes a tuple.

## Expressions

### Operators

The following operators are available, in order of increasing precedence:

- unary `- ! & | ^`
- `**`
- `* / %`
- `+ -`
- `<< >>`
- `< > <= >=`
- `== !=`
- `&`
- `^`
- `|`

#### Negation Operators

- `-`; `T -> T`; `T in {SInt<N>, UInt<N>, Int}` - Negation (two's complement)
- `!`; `T -> T`; `T in {Bit, [Bit; N], SInt<N>, UInt<N>}` - Bitwise inversion (one's complement)

#### Reduction Operators

- `& | ^`; `T -> Bit`; `T in {Bit, [Bit; N], SInt<N>, UInt<N>}` - Reduction AND, OR, XOR

#### Arithmetic Operators

- `** * / % + -`; `(T, T) -> T`; `T in {SInt<N>, UInt<N>, Int}` - Basic arithmetic

#### Relational Operators

- `== !=`; `(T, T) -> Bit` - Equality operators
- `< > <= >=`; `(T, T) -> Bit`; `T in {Bit, [Bit; N], SInt<N>, UInt<N>, Int}` - Relational operators

#### Shift Operators

- `<< >>`; `(T, I) -> T`; `T in {[Bit; N], SInt<N>, UInt<N>, Int}, I in {SInt<N>, UInt<N>, Int}` - Logical left and right shift

### Casts

### Packing / Unpacking

### Case / Match

## Hardware Generation

### Module Instatiation

### Function Calls

```
fn unpack<R>(ty: Type, x: [Bit; N]) -> R where R = ty {
    ...
}

let soup: [Bit; 64] = ...;
unpack(SInt<64>, soup); -> R?
```

`foo(bar)`; `foo(SInt<32>, bar)`

### Indexing and Slicing

`foo[bla]`; `foo[base; length]`

### Member Accesses

`foo.bar`

## Registers

## Modules

Modules represent pieces of hardware with a clearly defined list of input and output signals.
They can be parametric over types and constant values.
Every statement and expression will eventually be unrolled and captured inside a module.
Modules are defined as follows:
```
module Foo {
    port clk: input Bit;
    port master: input Axi, AxiAtop,...;
    port slave: output Axi;
    param Cfg: FooCfg;
    port data_in: input SInt; // -> implied parametricity over data_in.N
    master.AW
}
```

## Structs

Structs are declared as follows:
```
struct ReadyValid {
    ready: Bit,
    valid: Bit,
}
```
Structs can be parameterized as follows:
```
struct Decoupled {
    param T: Type,
    data: T,
    ready: Bit,
    valid: Bit,
}

let foo: Decoupled;
foo.T = [Bit; 2];
```
Optionally a selection can be hoisted up next to the struct name for easier specializations.
```
struct Decoupled<T> {
    param T: Type,
    data: T,
    ready: Bit,
    valid: Bit,
}

let foo: Decoupled<[Bit; 2]>;
```
The type specialization is optional and can be omitted and the type parameter can be late bound.

- If the struct definition defines a parameter in its diamond operator it can be assigned by position.
- If there is no diamond involved, the parameter can be assigned by name when the struct is used.

## Traits

Not in the first revision of the language.


## Packages

Packages allow for functions, types, and modules to be grouped and organized. There are two ways to define a package:
```
package foo { /*...*/ }  // inline
package bar; // external
```
The first declares an inline package `foo` with the content provided in curly braces.
The second searches for a file `bar.si` or `bar/pkg.si`, compiles it, and exposes its content as package `bar`.
The content of a package can be accessed by prefixing a name with the package's name and `::` (namespace operator).
Packages may also be nested.
For example:
```
package a {
    struct A;
    package b {
        struct B;
    }
}

// Exposes the following names:
// a::A
// a::b::B
```
Packages are used to structure larger designs into separate files.
For example:
```
// foo.si
struct A;

// bar.si
struct B;
package quux;

// quux.si
struct C;

// pkg.si
package foo;
package bar;

// Exposes the following names in pkg.si:
// foo::A
// bar::B
// bar::quux::C
```

## Features

- Stable name generation
- Seamless ASIC toolflow integration
- Constraints in RTL
- Late param/port binding
- SystemVerilog black boxes

## Convetion

- Type and module names are CamelCased.