---
up:
  - "[[002_Languages/Python/01_Introduction|01_Introduction]]"
down:
prev:
topic: false
question: " What changes occurred in different versions"
---
#  What changes occurred in different versions


> [!Summary] Summary
> 
> |**Version**|**Released**|**Headline Change**|**Why it mattered**|
> |---|---|---|---|
> |**2.0**|2000|List comprehensions, Garbage collection| Modernized the language core.|
> |**3.0**|2008|Unicode everything, Clean syntax| Fixed fundamental design flaws.|
> |**3.5**|2015|`async` and `await`| Made Python viable for high-concurrency web apps.|
> |**3.10**|2021|Structural Pattern Matching| Added sophisticated logic branching.|
> |**3.11**|2022|Faster CPython| Addressed Python’s "slow" reputation.|
> |**3.13**|2024|Free-threaded & JIT| Beginning of the end for the GIL.|
> |**3.14**|2025|Safer string| auto-complete, template strings, lazy annotations by default, subinterpreters API,  faster startup.|

## **Version 3.0 (The Great Reset)**
- **Released:** December 2008
- **Headline:** "Py3K" – The first intentionally backward-incompatible release.
- **Key Details:**
    - **The Unicode Fix:** Every string became Unicode by default. This fixed the "sad faces" caused by encoding errors in Python 2, but broke almost every existing library at the time.
    - **Print as a Function:** `print "text"` became `print("text")`.
    - **Integer Division:** `5 / 2` now returned `2.5` instead of `2`.
    - **Cleaning House:** Removed redundant ways of doing things (e.g., `raw_input` was renamed to `input`, and the old `input` was deleted).
## **Version 2.0 (The Modern Prototype)**
- **Released:** October 2000
- **Headline:** The shift to a community-backed development process.
- **Key Details:**
    - **List Comprehensions:** Introduced the elegant syntax for creating lists (e.g., `[x for x in data]`), borrowed from Haskell.
    - **Garbage Collection:** Added a cycle-detecting garbage collector, making memory management much more robust.
    - **Unicode Support:** Preliminary support for Unicode was added (though it was still secondary to ASCII strings).
    - **Augmented Assignment:** Added shorthand operators like `+=` and `-=`.
## **Version 1.0 (Functional Foundations)**
- **Released:** January 1994
- **Headline:** The "Functional Programming" update.
- **Key Details:**
    - **Lisp Influence:** This version introduced `lambda`, `map`, `filter`, and `reduce`. Interestingly, these were submitted by a Lisp hacker who missed them in the early 0.x versions.
    - **Maturation:** By 1.0, the language had moved beyond a personal project and began to see adoption in academic and research circles.
## **Version 0.9.0 (The Birth)**
- **Released:** February 1991
- **Status:** The first public release (posted to `alt.sources`).
- **Key Details:**
    - **Core Logic:** It already featured the core of what we recognize as Python today: **classes with inheritance**, **exception handling**, **functions**, and the basic data types (`list`, `dict`, `str`).
    - **The Goal:** It was designed as a successor to the ABC language, intended to be a "bridge" between shell scripting and C, capable of interfacing with the Amoeba operating system.




    - 
## Modern Era: Key 3.x Releases
In the last few years, Python has focused heavily on **speed**, **developer experience**, and **type safety**.

### **Python 3.11 – 3.14: The Speed Era (Current)**
- **Massive Speedup (3.11):** CPython became 10–60% faster due to the "Specializing Adaptive Interpreter."
- **Better Errors (3.11/3.12):** Tracebacks now point to the _exact_ expression that failed, not just the line.
- **The "No-GIL" & JIT (3.13):** This is the current frontier. 3.13 introduced experimental support for running without the **Global Interpreter Lock (GIL)** for true multi-core processing and a basic **JIT compiler**.
- **Subinterpreters(3.14)** run multiple "mini-Pythons" inside one program
- **Safer Strings (t-strings)** allow Python to look at the text _before_ it gets sent to a database
- **Fewer "Stutters" (**Smoother Memory)** it does tiny gc cleanups in the background

### **Python 3.8 – 3.10: Syntactic Sugar & Logic**
- **Walrus Operator `:=` (3.8):** Allows you to assign values to variables as part of an expression.
- **Dictionary Merging (3.9):** Added the `|` operator to merge dictionaries easily.
- **Structural Pattern Matching (3.10):** Introduced `match` and `case` (similar to "switch" in other languages but much more powerful).
### **Python 3.5 – 3.7: The Async Revolution**
- **Async/Await (3.5):** Introduced native syntax for asynchronous programming.
- **F-Strings (3.6):** Added literal string interpolation (e.g., `f"Hello {name}"`), which became the new standard for readability.
- **Data Classes (3.7):** Automated the "boilerplate" code needed to create classes that primarily store data.


    

    

