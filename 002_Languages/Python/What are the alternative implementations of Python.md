---
up:
  - "[[002_Languages/Python/01_Introduction|01_Introduction]]"
down:
prev:
topic: false
question: " What are the alternative implementations of Python?"
---
#  What are the alternative implementations of Python?

> [!Summary] Summary
> |**Implementation**|**Written In**|**Primary Goal**|**Target Platform**|**Key Benefit**|
>|---|---|---|---|---|
>|**CPython**|C|Reference Standard|Cross-platform|Maximum library compatibility|
>|**PyPy**|RPython|Execution Speed|Cross-platform|High performance via JIT compiler|
>|**MicroPython**|C|Resource Efficiency|Microcontrollers|Extremely low memory footprint|
>|**Jython**|Java|JVM Integration|Java VM|Seamless use of Java libraries|
>|**IronPython**|C#|.NET Integration|.NET / Mono|Native access to .NET framework|
>|**GraalPy**|Java/C|Polyglot Support|GraalVM|High speed & Java interoperability|
>|**RustPython**|Rust|Safety & Portability|Rust Ecosystem|Memory safety and WASM support|

- **PyPy:**
    - **How it works:** Uses a **Just-In-Time (JIT)** compiler to convert Python code into machine code on the fly.
    - **Pros:** Can be significantly faster (sometimes 10x–100x) for long-running, math-heavy, or repetitive tasks.
    - **Cons:** Higher memory consumption and historically slower compatibility with C-extensions (though this has improved greatly).
        
- **GraalPy:**
    - **How it works:** Developed by Oracle, it runs Python on the **GraalVM** (a high-performance runtime that also supports Java, JS, and Ruby).
    - **Pros:** Excellent performance and deep integration with Java. It aims to support the entire scientific stack (NumPy, SciPy) with near-native speeds.

## 2. Platform-Specific Implementations
These are designed to let Python interoperate with other major software ecosystems.
- **Jython (Java):** * Compiles Python code into **Java Bytecode**, allowing it to run on any Java Virtual Machine (JVM).
    - **Best for:** Using Java libraries directly within Python code.
- **IronPython (.NET):**
    - Designed for integration with Microsoft’s **.NET framework**.
    - **Best for:** Developers working in Windows-centric environments who need to use .NET libraries.
## 3. Embedded & Hardware Implementations
Standard Python is too "heavy" for tiny microchips, leading to these stripped-down versions.
- **MicroPython:** * A lean and efficient implementation optimized to run on microcontrollers (like the ESP32 or STM32). It includes a small subset of the Python standard library.
- **CircuitPython:** * A derivative of MicroPython maintained by Adafruit. It is geared toward education and beginners, featuring simplified hardware drivers and "plug-and-play" USB support.

## 4. Modern & Experimental Implementations
- **RustPython:** An implementation of Python written in **Rust**. While still maturing, it aims to leverage Rust's safety and performance to create a modern, secure interpreter.
- **CPython (No-GIL / JIT builds):** While CPython is the "standard," recent versions (3.13+) have introduced experimental **JIT compilers** and the ability to disable the **Global Interpreter Lock (GIL)**, effectively creating "alternative" ways to run the standard implementation for massive parallel performance.

