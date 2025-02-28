# Syntax

Everything defined in Evermore, whether types, functions, or constants, is given simply as the name followed by an equals - 
```
MyFavouriteNumber = 13
```

## Literals
Built-in literals include:

### Numbers
All numeric literals can be written with _ at any point in the literal, intended to help split the number into groups of digits,
similar to how commas work in regular writing, e.g. `10_000_000` is equivalent to `10000000` (or `1e7`).

#### Integers
The regular integer literals have the `IntLit` type, which implicitly coerces/is a subtype of more or less all other numeric datatypes,
with warnings on overflows, or using a negative literal in a uint (maybe?), or significant floating-point precision loss etc.

The bases recognised are:
- Base 10, e.g. `13`, `-1`, `0`, `1e7`
- Base 2, e.g. `0b0110` or `-0b11`
- Base 16, e.g. `0x1ff` or `-0xCAFE` (case insensitive)

#### Floats
Floats have the same notation as integers (incl. bases), but require a `f` suffix (e.g. `3.14f`) to have a _FloatLit_ literal, otherwise it'd be a double.
This is fine, however, since doubles implicitly coerce to floats just fine (with warnings on precision loss etc.).

#### Doubles
Doubles work like floats, but without the `f` suffix (they have an _optional_ `d` suffix, which always causes a warning when coerced into a Float).
Both `1.4` and `1.4d` are of the `DoubleLit` type, but the `d` sets a flag in the type which causes the warning.
If there's a precision loss when coercing, warnings are emitted regardless of if the `d` suffix is present.

#### Strings
Strings are of the `StringLit` type, and look like `"abcde"` normally. We also have raw string literals, which is a `r` N hashes, followed by a `"`,
followed by any unicode except for `"` and N hashes, and terminated with a `"` followed by N hashes (for any N > 0).
Raw strings don't escape any characters within them, and translate exactly as written - e.g. `r#"abde\n\n\t"""#` is `"abcde\\n\\n\\t\"\""` if written normally.
`StringLit` will happily coerce into a list of characters, an array of characters (with the correct length), and the same for `u8` instead of `char`
(raising a warning if there's complex unicode/non-single-byte characters in the string).

#### Dictionaries
Dictionary literals are of the `DictLit` type, and look like `{"abc": 7, "def": "uhh", ghi: r#"\nwhoa"#}`, or key-value pairings for a key that's either a StringLit or 
an identifier, and potentially non-homogenous value types. This is only allowed since a `DictLit` isn't generic over some `K`, `V` key/value types,
but is in fact something more complex (contains a string/type mapping internally).

#### Sets
Sets (of the `SetLit` type) are a list of potentially non-homogenous values, written as `{7, "hi!!", {"abc":8}, StringLit}`. They don't coerce into Lists or their literals,
but will coerce into non-literal `Set` types (provided that the types match).

#### Lists
List literals are square-bracketed homogenous values, like `["abc", "def"]` or `[1.4f, 9.0f, 4f,]`. Again, they coerce into the non-lit types.

#### Codeblocks
Codeblocks (of type `Codeblock`) are blocks of code, delimited by curly braces (`{}`). These are more complex and subtle, and intended for use in Kotlin-style DSLs.
The intention is for this to be handled at compile-time, since at runtime codeblocks have no use, and are flattened to their result value if they exist.
Hence they coerce into their resultant type.
Take, for example, this snippet:
```
const alpha = {
  const beta = 4
  beta + 2
}
```
At compile time, `alpha` has type `Codeblock<IntLit>`, which (if needed) coerces to `IntLit` which coerces to some numeric.
<!--
  TODO: I need to figure out co/contravariance, can I coerce `Codeblock<IntLit>` to `CodeBlock<u8>` (yes, i should be able to, but how to ensure this is nicely done?)
-->

#### Quotes
Like codeblocks, quotes (of type `Quote`) are compile time, but are yet more complex. Quotes are the parsed and typechecked syntax tree of things,
before they are coerced into runtime-level things. Codeblocks, for instance, start life as a `Quote<Codeblock<T>>` which becomes a `Codeblock<T>`
and then a plain `T` at runtime.
