# Data types

- In Rust, every value is of a certain Data Type which tells the compiler what kind of data is being specified so it knows how to work with that data. 
- Rust is a **statically typed** language, meaning it must know the types of all variables at compile time.
## 1. Scalar Types

A scalar type represents a single value. Rust has four primary scalar types:

### Integers
- Numbers without fractional components. 
- They can be **signed** (starting with `i`) or **unsigned** (starting with `u`) 
- range in size from 8-bit to 128-bit (8, 16, 32, 64, 128)
- along with architecture-dependent `isize` and `usize` types. 
- Rust defaults to **i32** for integer types

| **Type**   | **Bits**         | **Range (Min)**            | **Range (Max)**              |
| ---------- | ---------------- | -------------------------- | ---------------------------- |
| **`i8`**   | 8                | $-2^7 = -128$              | $2^7 - 1 = 127$              |
| **`u8`**   | 8                | $0$                        | $2^8 - 1 = 255$              |
| **`i16`**  | 16               | $-2^{15} = -32,768$        | $2^{15} - 1 = 32,767$        |
| **`u16`**  | 16               | $0$                        | $2^{16} - 1 = 65,535$        |
| **`i32`**  | 32               | $-2^{31} = -2,147,483,648$ | $2^{31} - 1 = 2,147,483,647$ |
| **`u32`**  | 32               | $0$                        | $2^{32} - 1 = 4,294,967,295$ |
| **`i64`**  | 64               | $-2^{63}$                  | $2^{63} - 1$                 |
| **`u64`**  | 64               | $0$                        | $2^{64} - 1$                 |
| **`i128`** | 128              | $-2^{127}$                 | $2^{127} - 1$                |
| **`u128`** | 128              | $0$                        | $2^{128} - 1$                |
| usize      | system dependent |                            |                              |
| isize      | system dependent |                            |                              |
### float

- Numbers with decimal points.
- Rust provides **f32** and **f64** (the default), both of which are signed
- It stores the value using [[Notes/Uncategorized/IEEE 745 standard for floating points|IEEE 745 standards]].

### Boolean

- Represented by the **bool** type, these have two possible values:
	- **true** and **false**.
- They are **one byte** in size.

### Characters
- The **char** type represents a **Unicode scalar value** and is four bytes in size.
- It can represent emojis, accented letters, and non-Latin characters.
- `'` (single quote) is used to denote a char

## 2. Compound Types

Compound types can group multiple values into one type.

- **[[Notes/rust/Tuple|Tuples]]**: A general way of grouping together a number of values with a variety of types into one compound type. They have a fixed length.
           
- **[[Notes/rust/Array|Arrays]]**: A collection of multiple values of the **same type**. Unlike vectors, arrays in Rust have a fixed length that must be known at compile time.
- [[Notes/rust/Vector|Vectors]]: A collection of multiple values of the **same type**. I can grow and shrink in size.
- [[Notes/rust/String|Strings]]: Collection of characters
- [[Notes/rust/Hash maps|Hash Maps]]: A key value
### 3. Custom Data Types

Once you master the basics, you move into defining your own complex structures:

- **[[Notes/rust/structs|structs]]**: Used to create custom types by naming and packaging related values.
    
- **[[Notes/rust/Enums|Enums]]**: Allow you to define a type by enumerating its possible variants