# IEEE 745 standard for floating points

- The IEEE Standard for Floating-Point Arithmetic (IEEE 754) is a technical standard for floating-point arithmetic originally established in 1985 by the Institute of Electrical and Electronics Engineers (IEEE).

- The standard defines:

	- **arithmetic formats**: sets of binary and decimal floating-point data, which consist of finite numbers (including signed zeros and subnormal numbers), infinities, and special "not a number" values (NaNs)
	- **interchange formats**: encodings (bit strings) that may be used to exchange floating-point data in an efficient and compact form
	- **rounding rules**: properties to be satisfied when rounding numbers during arithmetic and conversions
	- **operations**: arithmetic and other operations (such as trigonometric functions) on arithmetic formats
	- **exception handling**: indications of exceptional conditions (such as division by zero, overflow, etc.)

## 1. Bit Layout and Components

Floating-point numbers are stored in three distinct fields:

- **Sign Bit (1 bit)**: 0 for positive, 1 for negative.
- **Biased Exponent**: Determines the magnitude. It uses a "bias" (an offset) so it can represent both positive and negative powers of 2 without needing a separate sign bit.
- **Mantissa** (Significand/Fraction): Stores the significant digits. In "normalized" form, there is an implicit leading 1 (e.g., $1.fraction$), which is not stored to save space.

## 2. Common Formats

The standard defines several precisions, with Single and Double being the most common in programming languages like C, Java, and Python.

|Format [1, 2, 11, 12, 13]|Total Bits|Exponent Bits|Mantissa Bits|Exponent Bias|Decimal Precision|
|---|---|---|---|---|---|
|Single (float)|32|8|23 (+1 implicit)|127|~7 digits|
|Double (double)|64|11|52 (+1 implicit)|1023|~16 digits|
|Half (fp16)|16|5|10 (+1 implicit)|15|~3 digits|

## 3. Special Values & Handling

IEEE 754 reserves specific bit patterns for "boundary" cases: [1, 14]

- **Zero**: Represented with an exponent and mantissa of all 0s. The standard distinguishes between +0 and -0.
- **Infinity** ($\infty$): Exponent is all 1s, mantissa is all 0s. Results from operations like $1.0 / 0.0$.
- **NaN** (Not a Number): Exponent is all 1s, but mantissa is non-zero. Indicates invalid operations like $0.0 / 0.0$ or $\sqrt{-1}$.
- **Subnormal** (Denormal) Numbers: Used for extremely small values close to zero where the exponent is all 0s but the mantissa is non-zero. This prevents "flush to zero" and allows for gradual underflow. [1, 2, 15, 16, 17]

## 4. Rounding Rules

Because many decimal numbers (like $0.1$ or $1/3$) cannot be represented exactly in binary, the standard mandates rounding.

- Default: "Round to nearest, ties to even".
- Directed Rounding: Toward zero (truncation), toward $+\infty$, or toward $-\infty$



