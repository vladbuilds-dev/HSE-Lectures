# Seminar 2: How C++ Stores and Manipulates Numbers

## Introduction: every value must fit

In mathematics, integers go on forever, and real numbers can have infinitely many digits. A computer has limited memory. So every C++ object uses a limited amount of storage, and only a limited set of bit patterns can fit there.

> How does a limited bit pattern represent a value? What happens if an operation needs a value that does not fit?

The answer depends on the type:

- integer types store a limited set of whole numbers;
- floating-point types can cover a very large range, but they are not always exact;
- bitwise operators let us change individual bits.

Python helps show what is new. A Python `int` can grow when the number gets bigger. A C++ integer cannot. A Python `float` usually uses the same format as a C++ `double`. So `0.1 + 0.2 == 0.3` can also be false in Python. This is not a C++-only surprise.

```text
limited storage
    -> limited bit patterns
    -> the type gives those patterns a meaning
    -> this limits range or precision
    -> arithmetic must follow those limits
    -> bitwise operators change selected bits on purpose
```

This workbook is a reference you can read later. We will not cover every subsection in class. After the seminar you should be able to convert small numbers between bases, tell language rules from results that depend on your computer, explain overflow and wraparound, tell floating-point range from precision, and change bits in an unsigned mask.

---

> **Part I — Integers: limited patterns and limited values**  
> Central question: *How do bit patterns become integer values?*

## 1. Retrieval from Seminar 1

Seminar 1 introduced objects, types, literals, and `sizeof`. Recall:

```cpp
int count{42};
```

- `count` identifies an object;
- `int` determines its permitted values and operations;
- `42` is an `int` literal;
- the object occupies storage during its lifetime.

Predict before running:

```cpp
std::cout << sizeof(char) << '\n';
std::cout << sizeof(int) << '\n';
std::cout << sizeof(long) << '\n';
```

`sizeof` tells you how many C++ bytes an object uses. In C++, `char` is the unit of this measurement:

```cpp
sizeof(char) == 1
```

This is a language rule, not a value we measure on the hardware. The compiler does not discover it when the program runs. Every other `sizeof` result is counted in `char` units. If `sizeof(int)` prints `4`, then an `int` uses four `char` objects of storage on this computer. The standard does not say that `int` must be 4 bytes.

The sizes of `int`, `long`, and most other built-in types are chosen by your compiler and platform. The standard only requires some minimum ranges (for example, `int` is at least 16 bits) and this order:

```text
sizeof(char) <= sizeof(short) <= sizeof(int)
    <= sizeof(long) <= sizeof(long long)
```

A C++ byte often has 8 bits, but `sizeof` does not tell you the number of bits. That number is implementation-defined. On almost every machine you will use, it is 8. You can read it as `CHAR_BIT` from `<climits>`.

**Bridge.** `sizeof` counts C++ bytes. Next we need bits, so we can see what those bytes can store.

## 2. Bits, bytes, binary, and hexadecimal

A **bit** has two possible values: `0` or `1`. Each new bit doubles the number of patterns. One bit gives 2 patterns, two bits give 4, three bits give 8. With $N$ bits, there are

$$2^{N}$$

possible patterns.

For example, four bits give sixteen patterns:

```text
0000, 0001, 0010, ..., 1111
```

### Positional notation

The meaning of a digit depends on *where* it sits. We start from the right.

In decimal, each step to the left is 10 times larger: ones, tens, hundreds, thousands. So `205` means:

```text
2 hundreds  +  0 tens  +  5 ones  =  200 + 0 + 5  =  205
```

Binary works the same way, but each step to the left is only 2 times larger: ones, twos, fours, eights.

```text
Place (from the right):   eights   fours   twos   ones
Place value:                  8       4      2      1
Bits in 1101:                 1       1      0      1
```

Read it as: one 8, one 4, no 2, and one 1.

$$1101_{2} = 8 + 4 + 0 + 1 = 13_{10}$$

A `1` turns that place on. A `0` turns it off. The subscript `2` means “this number is written in binary.” The subscript `10` means ordinary decimal.

### Hexadecimal as compact binary

Four bits give 16 patterns. Hexadecimal has 16 digits: `0`–`9` and `A`–`F`. So one hex digit is exactly four bits. If we group binary digits in fours, we write the same number in a shorter form:

| Binary | Hex | Decimal |
|---|---|---:|
| `0000` | `0` | 0 |
| `0001` | `1` | 1 |
| `0010` | `2` | 2 |
| `0011` | `3` | 3 |
| `0100` | `4` | 4 |
| `0101` | `5` | 5 |
| `0110` | `6` | 6 |
| `0111` | `7` | 7 |
| `1000` | `8` | 8 |
| `1001` | `9` | 9 |
| `1010` | `A` | 10 |
| `1011` | `B` | 11 |
| `1100` | `C` | 12 |
| `1101` | `D` | 13 |
| `1110` | `E` | 14 |
| `1111` | `F` | 15 |

Thus:

```text
1101 0110₂ = D6₁₆
```

C++ integer literals can make the base explicit:

```cpp
int decimal{214};
int binary{0b1101'0110};
int hexadecimal{0xD6};
```

All three objects store the same number. Apostrophes inside a numeric literal only make the digits easier to read. They do not change the value.

### Displaying bits

`std::bitset` provides a convenient teaching representation:

```cpp
#include <bitset>
#include <iostream>

int main() {
    unsigned int value{13};

    std::cout << value << '\n';
    std::cout << std::bitset<8>{value} << '\n';
}
```

Output:

```text
13
00001101
```

`std::bitset<8>` prints 8 bits because we asked for 8 bits. This does not mean `unsigned int` is 8 bits wide. Extra zeros on the left are only for display.

**Checkpoint.** Convert `0010'1101₂` to decimal and hexadecimal before checking with a program.

## 3. Integer types, sizes, and ranges

The standard integer families include:

```text
signed char, short, int, long, long long
unsigned char, unsigned short, unsigned int,
unsigned long, unsigned long long
```

C++ comes from C. Integer sizes must work on many machines, so they are not fixed to one width. The same program can run on 16-bit, 32-bit, and 64-bit systems. The standard gives minimum sizes (for example, `int` is at least 16 bits) and the `sizeof` order from Section 1. It does not say how many bytes `int` or `long` must use.

Even 64-bit computers do not all agree about `long`. On Windows, `long` is often 4 bytes. On many Linux and macOS systems, `long` is often 8 bytes. Both can follow the standard. So do not copy a maximum value from another computer and treat it as a C++ rule.

Use a program, not a guess, to inspect the current implementation:

```cpp
#include <iostream>
#include <limits>

int main() {
    std::cout << "int bytes: " << sizeof(int) << '\n';
    std::cout << "int min: "
              << std::numeric_limits<int>::min() << '\n';
    std::cout << "int max: "
              << std::numeric_limits<int>::max() << '\n';
    std::cout << "unsigned int max: "
              << std::numeric_limits<unsigned int>::max() << '\n';
}
```

`std::numeric_limits<T>` tells you facts about type `T` on the compiler you just used. This is safer than remembering a maximum from another machine. That remembered number may be wrong here.

For integer types, `min()` and `lowest()` are the same most-negative value. For floating-point types they are different. Section 9 explains that.

### Signed and unsigned ranges

With $N$ bits there are $2^N$ patterns. The range depends on how we use those patterns:

- a signed integer type stores values from $-2^{N-1}$ through $2^{N-1}-1$;
- the matching unsigned type stores values from $0$ through $2^{N}-1$.

Unsigned uses every pattern for 0 and positive numbers. A typical 32-bit `unsigned int` goes up to $2^{32}-1$ (about 4.29 billion). Signed integers also need negative numbers, so about half of the patterns are used for negatives. A typical 32-bit `int` goes from about $-2.15$ billion to about $2.15$ billion. Since C++20, signed integers use two's complement. That is the encoding behind these formulas.

Do not think `unsigned` is always “safer for positive numbers.” Its rules are different. If you subtract below zero, the result wraps to a large number. It does not become negative.

## 4. Fixed-width integer aliases

Section 3 showed that `int` and `long` do not have one size on every computer. That is fine for a classroom counter. It is a problem when the width must be exact.

Imagine a file format, a network packet, or a hardware register that says: “this field is 32 bits.” If you store it in `int`, you might get 16 bits on one machine and 32 bits on another. If you store it in `long`, you might get 32 bits on Windows and 64 bits on Linux. The program would then read or write the wrong number of bits.

The header `<cstdint>` gives extra *names* for this situation. They are not new kinds of integer. They are nicknames for an existing type that has a known width:

```cpp
#include <cstdint>

std::int32_t signed_value{-100};
std::uint32_t flags{};
```

- `std::int32_t` means “a signed integer with exactly 32 value bits.”
- `std::uint32_t` means “an unsigned integer with exactly 32 value bits and no unused padding bits.”

On a typical desktop, `std::int32_t` is another name for `int`. The useful part is the *promise in the name*: 32 bits, not “whatever `int` is today.”

These exact-width names are optional. The compiler provides `std::int32_t` only if it has a matching type. If there is no exact 32-bit signed integer, the name does not exist. On the machines in this course, it almost always exists.

You do **not** need `std::int32_t` for every integer. Use it when the width is part of the requirement. For an ordinary loop counter, `int` is usually clearer.

The header also has names that always exist:

```cpp
std::int_least32_t
std::uint_least32_t
```

These mean “at least 32 bits,” maybe more. Use them when you need “no smaller than this,” not “exactly this many bits.”

### The `int8_t` output trap

`std::int8_t` and `std::uint8_t` are also nicknames. On most systems they are other names for `signed char` and `unsigned char`. `std::cout` already has a special rule for character types: it prints a symbol, not a decimal. The stored value 65 is still 65, but the output may look like `A` (the character with code 65). Convert to a wider integer if you want to print 65:

```cpp
std::uint8_t value{65};

std::cout << value << '\n';                   // may look like A
std::cout << static_cast<unsigned int>(value) // 65
          << '\n';
```

Choose a type from what you need:

- a field that must be exactly 32 bits: consider `std::uint32_t` or `std::int32_t`;
- a container size or array index: normally use the library size type;
- an ordinary classroom counter: `int` is usually clearer;
- bit flags: use an unsigned type that is wide enough, often `std::uint32_t`.

**Bridge.** The type you declare is not always the type used by an operator. C++ may convert operands first.

## 5. Integral promotions and conversions

Seminar 1 showed:

```cpp
auto result{'a' + 1};
```

The result is usually `int`, not `char`. Why? Operators such as `+` mainly work on `int` and larger types. Before `+` runs, a small type is **promoted**: `char`, `short`, and `bool` become `int` (or `unsigned int` if `int` cannot hold every value of the small type).

This is why `'a' + 1` does not wrap inside an 8-bit `char`. The addition happens in `int`. If `'a'` has the value 97, the result is 98, type `int`.

Conversions also happen when two different types meet. The operator needs one common type. Here, `count` is converted to `double` before the multiply. Otherwise `5 * 0.5` would not know whether to use integer or floating-point rules:

```cpp
int count{5};
double scale{0.5};
auto result{count * scale}; // double, value 2.5
```

If both sides stayed `int`, `5 * 0` would be the integer product after truncating `0.5`, which is the wrong idea for a scale factor.

### Conversion versus narrowing

```cpp
double precise{3.9};
int first = precise; // accepted, fractional part discarded
int second{precise}; // rejected: narrowing in list initialization
```

A **narrowing** conversion can lose information: `3.9` cannot be stored exactly in `int`. With `=`, C++ still allows it and drops `.9`, so `first` becomes `3`. With `{}`, C++ is stricter. If the conversion could change the value, the code does not compile. Braces mean “this value must fit.”

Compiler warnings are useful, but a warning is not a language rule. Choose a type that can hold what you need, and write conversions on purpose with `static_cast` when you really want to drop information.

### Mixed signed and unsigned arithmetic

This comparison is easy to misread:

```cpp
int signed_value{-1};
unsigned int unsigned_value{1};

std::cout << (signed_value < unsigned_value) << '\n';
```

You might expect “`-1` is less than `1`,” so the result should be true. That is the math meaning. C++ first converts both sides to one common type. On most systems that common type is `unsigned int`. Converting `-1` to unsigned wraps to the largest unsigned value (all bits 1). That huge number is not less than `1`, so the printed result is false.

Do not mix signed and unsigned types without a clear reason. You do not need to memorize every conversion rule from this one example. The exact rules depend on the sizes and ranges of the two types.

## 6. When an integer result does not fit

The type also limits *arithmetic*, not only storage. Consider:

```cpp
int a{1'000'000'000};
int b{2'000'000'000};
int sum{a + b};
```

On a common 32-bit `int`, each of `a` and `b` fits, but the true sum 3{,}000{,}000{,}000 does not. The question is then: what does C++ do?

### Signed overflow

If a signed result does not fit, the behavior is undefined. Compilers may assume overflow never happens and then simplify the program. Do not guess that you will get a wrapped negative number. Do not run the next line “to see what happens”:

```cpp
int maximum{std::numeric_limits<int>::max()};
maximum++; // undefined behavior: do not execute
```

Undefined behavior means C++ does not say what must happen. The program may print a wrapped number, may change after a small edit, or may do something else. One result on one computer does not become a rule.

### Unsigned modulo arithmetic

Unsigned arithmetic is defined to wrap. For an unsigned type with $N$ bits, results are computed modulo $2^N$. Think of a car odometer with $N$ digits: after the maximum, it returns to 0. If you add 1 to the maximum, all bits change from 1 to 0:

```cpp
unsigned int maximum{
    std::numeric_limits<unsigned int>::max()
};

std::cout << maximum + 1u << '\n'; // 0
```

This wraparound is defined, but defined is not the same as correct for your task. Wrapping a counter or a price to 0 may still be the wrong answer.

### Widen before calculating

Storing the result in a wider type is not enough. The *operands* of `+` decide the type of the addition. You must convert *before* the overflowing operation:

```cpp
int a{1'000'000'000};
int b{2'000'000'000};

long long wrong{a + b};
long long correct{
    static_cast<long long>(a) + b
};
```

In `wrong`, `a + b` is still `int + int`. The overflow (if it happens) is already done. Copying that result into `long long` cannot repair it. In `correct`, we convert one operand first. Then the other operand is converted too, and the addition uses `long long`, which can hold 3{,}000{,}000{,}000. This is safe for these values. It is not a complete solution for every overflow.

**Preview.** Later we can write a function that checks the limits *before* adding. That needs `if`, which is Seminar 3.

### Part I checkpoint

Explain each statement:

1. The number of bytes used by `long` is implementation-defined.
2. `std::uint32_t` is exact-width but conditional on implementation support.
3. Unsigned wraparound is defined; signed overflow is not.
4. Assigning an already-overflowed result to a wider variable does not repair it.

---

> **Part II — Floating point: a large range, but not exact**  
> Central question: *How can a limited pattern cover a huge range?*

## 7. A scientific-notation mental model

In decimal scientific notation we split a number into a fraction part and a power of 10. Then a few digits can describe a very large or very small number:

$$6.25 \times 10^{3} = 6250$$

Binary floating point does the same thing, but with powers of 2. Some bits store the scale (the exponent). The other bits store the fraction. This gives a large range. The cost is that many real numbers cannot be stored exactly. For large numbers, the stored values are farther apart.

$$1.101_{2} \times 2^{4}$$

A common IEEE 754 binary floating-point representation divides its bits into:

```text
sign | exponent | fraction/significand field
```

Most programmers call this IEEE 754. The C++ standard uses the name IEC 60559. `std::numeric_limits<T>::is_iec559` tells you whether your compiler claims this support. In this workbook, we say IEEE 754 for the format, and `is_iec559` for the C++ check.

For common IEEE 754 formats:

| C++ type on a common implementation | Sign | Exponent | Fraction field |
|---|---:|---:|---:|
| `float` as binary32 | 1 | 8 | 23 |
| `double` as binary64 | 1 | 11 | 52 |

For a normal binary number, the first fraction bit is always 1, so it is not stored. binary32 stores 23 fraction bits, which gives 24 bits of precision. binary64 stores 52 fraction bits, which gives 53 bits of precision.

> C++ does not require every implementation to use these exact layouts. Check `std::numeric_limits<T>::is_iec559` and other properties. Do not assume them silently.

Special patterns can also mean very small values, infinity, and NaN (“not a number”). Those are extra topics. The main idea is limited precision.

## 8. Why `0.1 + 0.2` is surprising

Some decimals have no exact binary form, just as $1/3$ has no exact decimal form. Binary place values are $1/2$, $1/4$, $1/8$, and so on. You cannot add a finite number of these to get exactly $1/10$. So the computer stores the closest value it has. Both `0.1` and `0.2` (and often `0.3`) are already rounded. Adding two rounded values may not give the rounded value of $0.3$.

By default, `cout` prints about 6 significant digits. Then `0.1 + 0.2` often looks like `0.3`. That is only the printed text. It does not prove the values are equal. In Seminar 1, `std::fixed` with `std::setprecision` meant digits after the decimal point. Without `std::fixed`, `std::setprecision(17)` means 17 significant digits. That is usually enough to show that a `double` `0.1 + 0.2` is not exactly `0.3`.

The same check is often false in Python too. A normal Python `float` uses the same binary64 format.

```cpp
#include <iomanip>
#include <iostream>

int main() {
    double a{0.1};
    double b{0.2};
    double expected{0.3};

    std::cout << std::setprecision(17);
    std::cout << a + b << '\n';
    std::cout << expected << '\n';
    std::cout << std::boolalpha
              << (a + b == expected) << '\n';
}
```

On a common binary64 implementation, the values display approximately as:

```text
0.30000000000000004
0.29999999999999999
false
```

This is not a random bug, and it does not mean floating point is broken. The computer can store only some real values, and it must round the others.

### Precision is not the same as range

A `double` can be huge, but it cannot store every integer in that huge range. The fraction has a fixed number of bits. For large numbers, those bits are stretched farther apart. Nearby stored values then differ by 2, 4, or more, not by 1.

```cpp
double large{1e16};
double small{1.0};

std::cout << std::boolalpha
          << (large + small == large) << '\n';
```

On a usual `double`, this prints `true`. Near `1e16`, the next stored number is 2 away. Adding `1.0` is too small to change the value. Adding `2.0` reaches the next stored number, so `1e16 + 2.0 == 1e16` is usually false. Homework B asks you to check both.

### Subtraction and cancellation

If two close values are subtracted, their common leading digits disappear. The error that was hiding in the last bits becomes a large part of the result:

```cpp
double x{1.000000000000001};
double y{1.0};
double difference{x - y};
```

This is called **cancellation**. The subtraction looks simple, but the result may be much less accurate than `x` and `y` looked.

## 9. Inspecting floating-point properties

```cpp
#include <iostream>
#include <limits>

int main() {
    std::cout << std::boolalpha;
    std::cout << "IEC 60559: "
              << std::numeric_limits<double>::is_iec559
              << '\n';
    std::cout << "radix: "
              << std::numeric_limits<double>::radix
              << '\n';
    std::cout << "precision bits: "
              << std::numeric_limits<double>::digits
              << '\n';
    std::cout << "safe decimal digits: "
              << std::numeric_limits<double>::digits10
              << '\n';
    std::cout << "round-trip digits: "
              << std::numeric_limits<double>::max_digits10
              << '\n';
}
```

- `digits` is how many digits in the number base (usually 2) the type can store exactly;
- `digits10` is a safe count of decimal digits you can store without change;
- `max_digits10` is enough digits to print a value and read back the same `double`.

For an integer type, `digits` is the number of value bits. For unsigned types this is the full width. For signed types it is one less, because one bit is the sign. Later homework uses `std::numeric_limits<unsigned int>::digits` as that unsigned width.

`min()` and `lowest()` are not the same for every type:

| Query | Integers | Floating-point types |
|---|---|---|
| `min()` | most negative value | smallest *positive* normal value |
| `lowest()` | most negative value | most negative finite value |
| `max()` | most positive value | most positive finite value |

When you print both integers and floating-point types, use `lowest()` and `max()` for the ends of the range.

### Comparing calculated values

Use `==` when you really need exact equality, for example the same stored value twice. For calculated floating-point results, compare the difference with a small limit that fits the problem:

```cpp
#include <cmath>

double actual{0.1 + 0.2};
double expected{0.3};
double tolerance{1e-12};

bool close_enough{
    std::abs(actual - expected) <= tolerance
};
```

There is no one limit that works for every program. Better checks often use both a small fixed limit and a limit that scales with the size of the numbers.

### Part II checkpoint

Explain why all three statements can be true:

1. `double` has a much larger range than `int`.
2. `double` cannot represent every real number in that range.
3. Adding a small value to a very large `double` may leave the stored value unchanged.

---

> **Part III — Bitwise operators: each bit can be a yes/no flag**  
> Central question: *When is it useful to change individual bits?*

## 10. Bitwise operators

So far, bits encoded one number. Bitwise operators look at each bit on its own. This is useful when one integer stores several yes/no flags, such as file permissions.

Bitwise operators work bit by bit on integer values.

Let:

```text
a = 0010 1100₂  (44)
b = 0000 1010₂  (10)
```

| Operator | Meaning | Result |
|---|---|---|
| `a & b` | 1 where both bits are 1 | `0000 1000` = 8 |
| `a | b` | 1 where either bit is 1 | `0010 1110` = 46 |
| `a ^ b` | 1 where bits differ | `0010 0110` = 38 |
| `~a` | invert every bit | depends on promoted type width |

`~` flips every bit of the promoted value, not only the 8 bits we wrote on paper. If `a` is an 8-bit unsigned value, C++ often converts it to `int` first. Then `~a` has many leading 1 bits, not just `1101 0011`. For class examples, use a wide unsigned type such as `std::uint32_t`.

`value |= mask` means `value = value | mask`. The same idea works for `&=` and `^=`. The permission examples use these short forms.

### Logical operators are different

```cpp
6 && 3 // true: both convert to true
6 & 3  // 2: 0110 & 0011 = 0010
```

| Logical | Bitwise |
|---|---|
| `&&`, `||`, `!` | `&`, `|`, `^`, `~` |
| produce `bool` | produce an integral result |
| `&&` and `||` short-circuit | both operands are evaluated |
| reason about truth | reason about bit positions |

Use unsigned types when you start with bit operations. Signed types and promotions make the examples harder to follow.

## 11. Shift operators

A left shift `<<` moves bits to the left and fills 0 on the right. Bits that leave the type are lost. A right shift `>>` moves bits to the right. For unsigned values, `<< 1` is like multiply by 2 (and wrap if needed). `>> 1` is like divide by 2 and drop the remainder:

```cpp
std::uint32_t value{5}; // 00000101

auto left{value << 1};  // 00001010 = 10
auto right{value >> 1}; // 00000010 = 2
```

The shift count must be 0 or more, and smaller than the width of the (promoted) left value. If not, the behavior is undefined. For an unsigned type, that width is `std::numeric_limits<T>::digits`. A shift by exactly that width, such as `1u << std::numeric_limits<unsigned int>::digits`, is also undefined.

Do not start with negative signed values when you learn shifts. Use an unsigned type. Think of shifts as moving bits, not as a general replacement for `*` and `/`.

## 12. Masks and flags

A mask chooses one or more bits. One integer can store several yes/no flags: 1 means on, 0 means off. Each mask below is a power of two, so each has exactly one bit set.

Define four permission positions:

```cpp
#include <cstdint>

constexpr std::uint32_t read_permission{1u << 0};
constexpr std::uint32_t write_permission{1u << 1};
constexpr std::uint32_t execute_permission{1u << 2};
constexpr std::uint32_t share_permission{1u << 3};

std::uint32_t permissions{};
```

Their low eight bits are:

```text
read     0000 0001
write    0000 0010
execute  0000 0100
share    0000 1000
```

### Set a flag with `|`

```cpp
permissions |= read_permission;
permissions |= write_permission;
```

### Test a flag with `&`

```cpp
bool can_read{
    (permissions & read_permission) != 0
};
```

### Clear a flag with `&` and `~`

```cpp
permissions &= ~write_permission;
```

`~write_permission` has 1 in every bit except the write bit. AND then keeps the other flags and turns write off. We use `std::uint32_t`, so this invert step stays in a wide unsigned type.

### Toggle a flag with `^`

```cpp
permissions ^= execute_permission;
```

XOR flips the selected bit: 0 becomes 1, and 1 becomes 0.

### Display the mask

```cpp
#include <bitset>

std::cout << std::bitset<8>{permissions} << '\n';
```

### Part III checkpoint

Complete the mapping:

| Intent | Expression form |
|---|---|
| Set flag | `value ___ mask` |
| Test flag | `(value ___ mask) != 0` |
| Clear flag | `value ___ ___mask` |
| Toggle flag | `value ___ mask` |

## 13. Behavior and portability checklist

Before you trust a numeric or bitwise expression, ask:

1. What are the operand types after conversions?
2. Can the result type store the true mathematical result?
3. Is the behavior defined for signed and unsigned operands?
4. Is the shift count in the valid range?
5. Does the code assume an exact width or IEEE layout without checking?
6. Is a printed result only what this computer happened to show?

---

## 14. In-class exercises

Exercises 5 and 6 are the main practice in class. Use the others if you have time, or as extra work before the homework.

How to work:

- For prediction tasks, write your answer on paper *before* you compile.
- For programming tasks, you may edit `seminar02/src/main.cpp` or add a new `.cpp` file. Include what you use (`<iostream>`, `<limits>`, …).
- Do not run undefined expressions to get an answer.

### Exercise 1 — Base conversion

**Goal.** Convert small numbers between binary, decimal, and hexadecimal by hand, then check with a program.

**On paper, fill every `?` in this table.** Do not use a calculator. Use the hex table in Section 2. Group binary digits in fours.

| Binary | Decimal | Hexadecimal |
|---|---:|---|
| `0000 1101` | ? | ? |
| `0010 1101` | ? | ? |
| `1111 0000` | ? | ? |
| ? | 42 | ? |

For the last row, start from decimal 42 and find both binary and hex.

**Then write a short program** that prints the same values. Use binary literals such as `0b0000'1101` and `std::bitset<8>`. Print decimal, hexadecimal (`std::hex`), and eight bits for at least one value. Confirm that the program matches your table.

**Done when:** the table has no `?`, and the program output agrees with it.

### Exercise 2 — Type explorer

**Goal.** See the sizes and ranges of integer types *on this computer*, and say which facts are C++ rules.

**Write a program** that prints, for each of these eight types, three numbers: size in bytes, minimum value, maximum value.

Types: `short`, `unsigned short`, `int`, `unsigned int`, `long`, `unsigned long`, `long long`, `unsigned long long`.

Use `sizeof(T)` and `std::numeric_limits<T>`. For integers, `min()` and `lowest()` are the same; either is fine here.

Example of one line of output (your numbers may differ):

```text
int: bytes=4, min=-2147483648, max=2147483647
```

**Then write short answers** (a few sentences each):

1. Which printed results are C++ rules (true on every correct compiler), and which describe only this computer?
2. How does your `sizeof(long)` compare with a classmate on another OS, or with a remembered value from Windows/Linux?
3. Why is `std::numeric_limits<T>` safer than remembering one maximum, such as `2147483647`?

**Done when:** all eight types are printed, and you have answers to the three questions.

### Exercise 3 — Classify the behavior

**Goal.** Decide what C++ *promises*, without running dangerous code.

**On paper, classify each snippet** using Seminar 1 words: **defined**, **implementation-defined**, or **undefined**. If the result is defined but still depends on how large `int` is on this computer, say that too.

For a defined case, also say what the result is (for example `0`). For undefined cases, do **not** compile or run the expression.

1. Unsigned wrap:

```cpp
unsigned int u{std::numeric_limits<unsigned int>::max()};
u + 1u;
```

2. Signed overflow:

```cpp
int i{std::numeric_limits<int>::max()};
i + 1;
```

3. Widen too late:

```cpp
int a{1'000'000'000};
int b{2'000'000'000};
long long result{a + b};
```

4. Widen first:

```cpp
int a{1'000'000'000};
int b{2'000'000'000};
long long result{
    static_cast<long long>(a) + b
};
```

**Done when:** each of the four cases has a classification and a one-sentence reason.

### Exercise 4 — Floating-point laboratory

**Goal.** See that `double` rounding is real, and that default printing can hide it.

**Write one program** that does all of the following, in order:

1. Print `0.1 + 0.2` and `0.3` with `std::setprecision(17)` and **without** `std::fixed`.
2. Print `(0.1 + 0.2 == 0.3)` using `std::boolalpha` (you should see `true` or `false`).
3. Print whether `std::abs((0.1 + 0.2) - 0.3) <= 1e-12`.
4. Print whether `1e16 + 1.0 == 1e16`.
5. Optional: also print whether `1e16 + 2.0 == 1e16`.
6. Print `std::numeric_limits<double>::is_iec559`, `digits`, `digits10`, and `max_digits10`.

You need `<iostream>`, `<iomanip>`, `<cmath>`, and `<limits>`.

**Then, next to each printed line, write one sentence** that explains it. Do not only copy the numbers. For example: why `==` can be false, why 17 digits are needed, why adding `1.0` to `1e16` may do nothing.

**Done when:** the program prints every required line, and each line has a short written explanation.

### Exercise 5 — Predict bitwise results

**Goal.** Compute `&`, `|`, `^`, and shifts bit by bit, then check with C++.

**On paper first.** Use these unsigned bit patterns (decimal 44 and 10):

```text
a = 0010 1100
b = 0000 1010
```

Fill this table (eight bits and a decimal value for each result):

| Expression | Binary (8 bits) | Decimal |
|---|---|---:|
| `a & b` | ? | ? |
| `a \| b` | ? | ? |
| `a ^ b` | ? | ? |
| `a << 1` | ? | ? |
| `a >> 2` | ? | ? |

Work column by column for `&`, `|`, and `^`. For shifts, move bits and fill `0` in the empty places.

**Then write a program** with `std::uint32_t a{0b0010'1100};` and `b{0b0000'1010};`. Print each result with `std::bitset<8>` and as a decimal number. Check that it matches your table.

**Done when:** the paper table is complete and the program agrees with it.

### Exercise 6 — Permission-mask walkthrough

**Goal.** Store several yes/no flags in one integer, using `|`, `&`, `~`, and `^`.

**Write one program.** Use a single `std::uint32_t permissions{}` (starts at 0). Do **not** use four separate `bool` variables for the flags.

Define these masks:

```cpp
constexpr std::uint32_t read_permission{1u << 0};     // bit 0
constexpr std::uint32_t write_permission{1u << 1};    // bit 1
constexpr std::uint32_t execute_permission{1u << 2};  // bit 2
```

Then do these steps **in this order**, and after every change that modifies `permissions`, print `std::bitset<8>{permissions}`:

1. grant read: `permissions |= read_permission`;
2. grant write: `permissions |= write_permission`;
3. test read: print `(permissions & read_permission) != 0` with `std::boolalpha` (do not need a new bitset line for this test unless you want one);
4. revoke write: `permissions &= ~write_permission`;
5. toggle execute: `permissions ^= execute_permission`.

**Done when:** the printed bits show the flags turning on and off, and the read test prints `true` after step 2. Expected bit patterns after the modifications:

```text
00000001
00000011
true
00000001
00000101
```

### Exercise 7 — Find and explain the mistakes

**Goal.** Name the kind of mistake: undefined behavior, defined but surprising, or a wrong assumption.

**Do not run the first two snippets.** For each of the three examples, write:

- which label it is: “undefined behavior,” “defined but surprising,” or “wrong assumption”;
- one or two sentences that say *why*.

```cpp
int maximum{std::numeric_limits<int>::max()};
std::cout << maximum + 1 << '\n'; // do not run
```

```cpp
unsigned int left{1u};
int shift{std::numeric_limits<unsigned int>::digits};
std::cout << (left << shift) << '\n'; // do not run
```

```cpp
double value{0.1 + 0.2};
std::cout << (value == 0.3) << '\n';
```

The third snippet is safe to compile if you want, but first write your prediction.

**Done when:** all three examples have a label and a reason.

---

## 15. Exit ticket

Answer without running code. The live seminar may use a subset of these questions.

1. Why are there $2^N$ patterns for $N$ bits?
2. Is `sizeof(int)` always 4 on every computer?
3. What does `std::uint32_t` mean when the compiler provides it?
4. Why can `std::uint8_t` print like a character?
5. What is the difference between signed overflow and unsigned wraparound?
6. Why is `long long result{a + b};` sometimes too late to prevent overflow?
7. Why can `0.1 + 0.2 == 0.3` be false?
8. What is the difference between floating-point range and precision?
9. Which operator sets a bit? Which operator tests it?
10. What rules apply to a shift count?

## 16. Seminar synthesis

```text
type
  -> chooses what values and operations exist
  -> limited storage limits range or precision
  -> conversions may change the type used by an operation
  -> integer and floating-point limits affect arithmetic
  -> unsigned bits can store independent flags
```

The important lesson is not that computers “make mistakes.” The program asks a type to do an operation. Start by naming that type and the values it can store.

---

## 17. Homework

Submit four parts. Parts A and B are CMake programs. Parts C and D are written answers (no need to run the undefined lines in C).

Use the CMake skeleton at the end of this workbook. Warnings must be on. Do not submit the `build/` folder.

### Part A — Permission-mask manager

**What to build.** A C++20 CMake project with one `std::uint32_t` that stores five flags:

```text
read, write, execute, share, delete
```

**Steps in `main`:**

1. Define each permission as a `constexpr` mask with exactly one bit: `1u << 0` through `1u << 4`.
2. Start from `std::uint32_t permissions{}` (all bits 0).
3. Grant **read, write, and share** with `|=`. Do not grant execute or delete yet.
4. Print the mask with `std::bitset<8>` and the label `after grant:`.
5. Print whether **read**, **execute**, and **delete** are on, using `(permissions & mask) != 0` and `std::boolalpha`. You must use the `delete` mask here so it is not unused.
6. Revoke write with `&= ~write_permission`. Print the mask.
7. Toggle execute with `^=` **twice**. Print the mask after each toggle.
8. Put a short comment on every bitwise line explaining what it does (`|=` sets, `&` tests, `&= ~` clears, `^=` flips).

After step 3, delete must still be off. Match this output:

```text
after grant:          00001011
read enabled: true
execute enabled: false
delete enabled: false
after revoke write:   00001001
after toggle execute: 00001101
after second toggle:  00001001
```

**Done when:** the program builds with warnings, the printed lines match, and the comments explain the operators.

### Part B — Numeric representation report

**What to build.** One program plus a short written paragraph.

The program must print:

- for `int`, `unsigned int`, `long`, `float`, and `double`: `sizeof`, `lowest()`, and `max()` (do **not** use `min()` for the floating-point range);
- `std::numeric_limits<float>::is_iec559` and the same for `double`;
- `0.1 + 0.2` with `std::setprecision(std::numeric_limits<double>::max_digits10)` and **without** `std::fixed`;
- `(1e16 + 1.0 == 1e16)` and `(1e16 + 2.0 == 1e16)` with `std::boolalpha`.

**Then write 6–10 sentences** that answer:

- which printed facts are C++ rules (same on every correct implementation);
- which facts describe only this computer;
- what is the difference between `min()` and `lowest()` for `float` and `double`.

**Done when:** the program prints all listed items, and the paragraph covers those three points.

### Part C — Predict before running

**On paper** (you may check the first list with a program afterward).

For each defined expression, write **value** and **type**:

```cpp
0b0011u | 0b0101u
0b0110u & 0b0011u
0b0101u ^ 0b0001u
1u << 5
32u >> 3
std::numeric_limits<unsigned int>::max() + 1u
```

**Do not run** these two. Write only **defined** or **undefined**, and a short reason:

```cpp
std::numeric_limits<int>::max() + 1
1u << std::numeric_limits<unsigned int>::digits
```

Recall: for an unsigned type, `digits` is the width in bits. A shift count must be smaller than that width.

**Done when:** every line in the first list has value and type, and both lines in the second list are classified.

### Part D — Reflection

Write **5–8 sentences** that connect this chain in your own words:

> finite memory → bit patterns → type interpretation → range/precision → arithmetic behavior.

Cover integers and floating point. Mention overflow or wraparound, and that a type chooses how bits are used. Do not only list the arrows; explain them.

**Done when:** the paragraph is 5–8 sentences and follows the chain above.

## Standard CMake file

The live project is in `seminar02/`. It has the class demo in `src/main.cpp` and complete programs plus written answers in `seminar02/solutions/`.

Use the same project layout as Seminar 1: `CMakeLists.txt` next to a `src/main.cpp` file.

```cmake
cmake_minimum_required(VERSION 3.20)

project(workbook02 VERSION 1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

add_executable(workbook02 src/main.cpp)

if(MSVC)
    target_compile_options(workbook02 PRIVATE /W4)
else()
    target_compile_options(workbook02 PRIVATE -Wall -Wextra -Wpedantic)
endif()
```

Configure and build from the project root:

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build --config Debug
```
