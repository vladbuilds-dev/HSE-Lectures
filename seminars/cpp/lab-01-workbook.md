# Laboratory 1 Preparation Seminar: From Bits to Benchmarks

## Introduction: what connects the five tasks?

The laboratory looks like five separate topics: binary numbers, overflow, floating point, CMake, `chrono`, and speed. They are one investigation:

```text
type
  -> chooses a representation and operations
  -> representation has limits
  -> limits affect correctness
  -> the compiler translates the operations
  -> the processor runs the translated instructions
  -> measurement observes the whole system
```

> How does the chosen data type change the meaning, correctness, bit pattern, and measured speed of an operation?

This is an experiment, not only a coding task. For every task, keep four things separate:

1. **language rule** — what C++ always guarantees;
2. **implementation property** — what this compiler and computer provide;
3. **observation** — what happened in one run;
4. **conclusion** — what that evidence actually supports.

A Python `int` can grow. A C++ integer cannot. A Python `float` is usually the same format as a C++ `double`, so `0.1 + 0.2 == 0.3` can be false in both languages. The new work here is C++ types, undefined behavior, and measuring compiled code.

## Learning outcomes

After the seminar, you should be able to:

1. explain the multi-target CMake project;
2. separate declarations, definitions, and `main`;
3. print and read 32-bit integer bit patterns;
4. use bitwise operators and shifts carefully;
5. tell unsigned wraparound from signed overflow;
6. split a common binary32 `float` into sign, exponent, and fraction;
7. explain rounding, grouping, and `==` problems;
8. measure elapsed time with `<chrono>`;
9. explain what `volatile` changes and what it does not do;
10. compare operations and types more fairly;
11. write a result table and a careful conclusion.

## Laboratory map

| Lab part | Main ideas | Required evidence |
|---|---|---|
| Task 0 | files, headers, linking, CMake targets | correct project structure |
| Task 1 | binary, two's complement, bitwise operators, shifts | code plus bit calculations |
| Task 2 | numeric limits, wraparound, undefined behavior | representation and C++ rules |
| Task 3 | binary32, rounding, associativity, equality | bit layout and experiments |
| Task 4 | clocks, compiler optimization, same meaning | timing table and analysis |
| Task 5 | promotions, registers, floating units, `volatile` | two timing tables and conclusions |

---

# Part I. Task 0 - One project, five independent programs

## 1. Why the laboratory uses separate task directories

Each task answers a different question. A separate program gives every task:

- its own `main()`;
- its own `.cpp` and `.hpp` files;
- its own CMake target (one executable);
- independent compile and run steps;
- its own `README.md` with observations and conclusions.

Required layout:

```text
01lab/
|-- CMakeLists.txt
|-- 01_task/
|   |-- CMakeLists.txt
|   |-- main.cpp
|   |-- task01.hpp
|   |-- task01.cpp
|   `-- README.md
|-- 02_task/
|-- 03_task/
|-- 04_task/
|-- 05_task/
`-- README.md
```

Do not submit generated folders such as `build/` or `cmake-build-debug/` unless the instructor asks for them. They are not source code.

## 2. Declaration, definition, and call

The lab splits each task into three kinds of file. Use this small example.

### Header: `experiment.hpp`

```cpp
#pragma once

int square(int value);
```

This is a **declaration**. It tells other files: “a function with this name, this parameter type, and this return type exists.” It does not contain the body.

### Implementation: `experiment.cpp`

```cpp
#include "experiment.hpp"

int square(int value) {
    return value * value;
}
```

This is the **definition**. It contains the function body. The compiler translates this file. The linker later finds this body when `main` calls `square`.

### Entry point: `main.cpp`

```cpp
#include "experiment.hpp"

#include <iostream>

int main() {
    std::cout << square(5) << '\n';
}
```

`main.cpp` starts the program and prints results. Put reusable work in the implementation file, not all of it inside `main`.

If the definition is missing, the code can still *compile* (the declaration is enough). It will *fail to link*, because no object file contains the function body.

## 3. Why `#pragma once` is required

A header can be included more than once, including through other headers. `#pragma once` asks the compiler to read that header only once per `.cpp` file:

```cpp
#pragma once
```

Put it on the first line of every project header. The laboratory requires this.

Without a guard, repeating a declaration is sometimes OK. Repeating a *definition* can cause a compile error.

## 4. Why global variables are prohibited

A global variable can be used from many functions without appearing in their parameter lists:

```cpp
int value{42}; // global: not allowed in this laboratory
```

That hides data flow. Experiments become harder to isolate. Prefer parameters and return values:

```cpp
std::uint32_t shifted_left(std::uint32_t value, int count) {
    return value << count;
}
```

Named constants can be function arguments or local objects. The grading rule says no global variables. Do not assume a namespace-level `const` is allowed unless the instructor says so.

## 5. What CMake does

CMake does not compile C++ by itself. It prepares a build. Then a build tool calls the compiler and the linker.

Students run these two commands from the project root:

```bash
cmake -S . -B build
cmake --build build
```

- `-S .` = source folder (this project);
- `-B build` = a separate folder for generated files;
- `cmake --build build` = compile and link.

For Visual Studio (a multi-configuration generator), you may need:

```bash
cmake --build build --config Release
```

### Root `CMakeLists.txt`

```cmake
# Oldest CMake version we accept.
cmake_minimum_required(VERSION 3.10)

# Project name. LANGUAGES CXX means "this is a C++ project."
project(LabDataTypes LANGUAGES CXX)

# The template may request C++23. This workbook only needs C++20.
# Keep the standard your instructor requires.
set(CMAKE_CXX_STANDARD 23)
# Do not silently use an older standard.
set(CMAKE_CXX_STANDARD_REQUIRED ON)
# Do not use extra compiler-specific C++ syntax.
set(CMAKE_CXX_EXTENSIONS OFF)

# Build each task folder as its own program.
add_subdirectory(01_task)
add_subdirectory(02_task)
add_subdirectory(03_task)
add_subdirectory(04_task)
add_subdirectory(05_task)
```

`add_subdirectory` means: “also read the `CMakeLists.txt` inside that folder.”

### Task-level `CMakeLists.txt`

```cmake
# One program named task01, built from these two source files.
# main.cpp starts the program. task01.cpp contains the function bodies.
add_executable(
    task01
    main.cpp
    task01.cpp
)

# Let this program find headers in the same folder (task01.hpp).
target_include_directories(task01 PRIVATE .)
```

`add_executable(name file1.cpp file2.cpp)` is one program. Five tasks means five such lines, each in its own folder. That is why five `main()` functions are allowed: they belong to five different programs, not one.

**Checkpoint 0.** Why are five `main()` functions valid in this project, but invalid if you put them all in one executable?

---

# Part II. Task 1 - Integer representations and bitwise operations

## 6. A value is not the same thing as its representation

The number thirteen can be written in more than one way:

```text
13 in decimal  =  1101 in binary  =  D in hexadecimal
```

These are different spellings of the same value. In memory, the computer stores bits. The *type* says how to read those bits.

Binary place values start from the right: ones, twos, fours, eights.

```text
Place:     eights  fours  twos  ones
Value:         8      4     2     1
Bits:          1      1     0     1
```

Read it as: one 8, one 4, no 2, one 1. So `1101` is `8 + 4 + 0 + 1 = 13`.

With eight bits, the same number has leading zeros:

```text
0000 1101  =  13
```

The laboratory uses 32-bit types. Unsigned 13 then looks like this:

```text
00000000 00000000 00000000 00001101
```

## 7. Why use `std::uint32_t` for bit patterns?

`std::uint32_t`, when the compiler provides it, is an unsigned type with exactly 32 value bits and no unused padding bits:

```cpp
#include <cstdint>

std::uint32_t value{42};
```

Then “print 32 bits” matches the type. An ordinary `unsigned int` is often 32 bits, but C++ does not require that width everywhere.

Print the bits with:

```cpp
#include <bitset>

std::bitset<32>{value}
```

`std::bitset<32>` is a display tool. The `32` is how many bits we asked to print. It does not change the original integer.

## 8. Sign-magnitude, ones' complement, and two's complement

Negative integers need a rule for the sign. Computers have used more than one rule. Using eight bits for `-5`:

```text
+5                         0000 0101
sign-magnitude -5          1000 0101
ones' complement -5        1111 1010
two's complement -5        1111 1011
```

C++20 uses **two's complement**. How to get it:

1. write the positive pattern;
2. flip every bit (`0` becomes `1`, `1` becomes `0`);
3. add one.

```text
  0000 0101     +5
  1111 1010     flip all bits
+ 0000 0001     add 1
-----------
  1111 1011     -5 in two's complement
```

Why this is useful:

- zero has only one pattern;
- the same addition hardware works for signed and unsigned bits;
- when you make the number wider, copying the sign bit on the left keeps the same value.

## 9. Viewing a signed object's bits

Do not pass a negative number to `std::bitset<32>` without thinking. First copy the bits into an unsigned type:

```cpp
#include <bitset>
#include <cstdint>
#include <iostream>

std::int32_t signed_value{-5};
std::uint32_t bits{
    static_cast<std::uint32_t>(signed_value)
};

std::cout << std::bitset<32>{bits} << '\n';
```

The conversion wraps modulo $2^{32}$. The bit pattern of that unsigned value is the two's-complement pattern of `-5`.

## 10. Bitwise operators work position by position

Bitwise operators look at each bit on its own. Use a small example before the lab values:

```text
a = 0000 1101
b = 0000 1010
```

| Operator | Rule for each bit | Result |
|---|---|---|
| `a & b` | 1 only if both bits are 1 | `0000 1000` |
| `a \| b` | 1 if at least one bit is 1 | `0000 1111` |
| `a ^ b` | 1 if the bits are different | `0000 0111` |
| `~a` | flip every bit | depends on the full type width |

For `~`, print the full width (32 bits in the lab). Flipping only the 8 bits on paper is not the same as flipping a 32-bit value.

## 11. Shifts and their restrictions

```cpp
std::uint32_t value{5}; // ...00000101

auto left{value << 2};  // ...00010100
auto right{value >> 2}; // ...00000001
```

For unsigned values:

- `<<` moves bits to the left and fills `0` on the right;
- `>>` moves bits to the right and fills `0` on the left;
- `<< 1` is like multiply by 2, with wraparound;
- `>> k` is like divide by $2^{k}$ and drop the remainder.

The shift count must be 0 or more, and smaller than the width of the (promoted) left value. If not, the behavior is undefined. For `std::uint32_t`, a shift by 32 is undefined. Do not run that to “see what happens.”

For a negative signed value in C++20, `>>` copies the sign bit on the left. That matches division toward negative infinity, not toward zero:

```text
-17 / 4   using integer division  -> -4  (toward zero)
-17 >> 2                          -> -5  (toward -infinity)
```

So `x >> 1` is not always the same as `x / 2` when `x` can be negative.

**Checkpoint 1.** Before the laboratory pair, write the 0/1 table for one bit of `&`, `|`, and `^`.

---

# Part III. Task 2 - Integer overflow and the boundary of the language

## 12. Numeric limits

Use the type's own limits. Do not copy a remembered constant from another machine:

```cpp
#include <cstdint>
#include <limits>

auto unsigned_max{
    std::numeric_limits<std::uint32_t>::max()
};

auto signed_max{
    std::numeric_limits<std::int32_t>::max()
};
```

## 13. Unsigned arithmetic wraps around

For a 32-bit unsigned type, results wrap modulo $2^{32}$:

```text
UINT32_MAX     =  2^32 - 1
UINT32_MAX + 1 =  0
UINT32_MAX + 2 =  1
```

This is a C++ rule. The extra carry past the top bit is dropped. Defined wraparound can still be the wrong answer for your experiment, so say what you expected.

## 14. Signed overflow is undefined behavior

If a signed result does not fit, C++ does not define a wrapped result:

```cpp
std::int32_t value{
    std::numeric_limits<std::int32_t>::max()
};

value + 2; // undefined behavior: do not execute
```

Do not write “the answer is a negative number” just because one run printed one. After undefined behavior, C++ does not require any particular result. Do not run the signed overflow to “see the number.”

In the laboratory report, keep these three things separate:

```text
Mathematical result       2^31 + 1
32-bit hardware pattern   10000000 00000000 00000000 00000001
C++ signed expression     undefined behavior
```

If you need to show the wrapped bit pattern, use defined unsigned arithmetic. Then say how those bits would be read as a signed number. That is a representation experiment, not the result of the invalid signed expression.

## 15. Undefined behavior versus one computer's output

| Category | Meaning | Example |
|---|---|---|
| Defined | every correct C++ implementation follows the rule | unsigned wraparound |
| Implementation-defined | the implementation chooses and documents one option | some type sizes |
| Unspecified | several outcomes are allowed | some evaluation choices |
| Undefined | C++ does not say what must happen | signed overflow |

Running a program can show an observation. It cannot turn undefined behavior into a portable result.

**Checkpoint 2.** Why is “it printed the same value three times” not proof that signed overflow is defined?

---

# Part IV. Task 3 - Floating-point representation and numerical behavior

## 16. Floating point as binary scientific notation

In decimal scientific notation we split a number into a fraction part and a power of 10:

$$6250 = 6.25 \times 10^{3}$$

Binary floating point does the same thing with powers of 2:

$$101.01_{2} = 1.0101_{2} \times 2^{2}$$

On the common IEEE 754 binary32 format used for `float`, 32 bits are split like this:

```text
1 sign bit | 8 exponent bits | 23 fraction bits
```

For a normal number:

$$(-1)^{\mathrm{sign}} \times 1.\mathrm{fraction}_{2} \times 2^{\mathrm{exponent}-127}$$

Most programmers say IEEE 754. C++ uses the name IEC 60559. Check `std::numeric_limits<float>::is_iec559` before you read the bits this way. C++ does not require every `float` to be binary32.

```cpp
#include <limits>

static_assert(sizeof(float) == 4);
static_assert(std::numeric_limits<float>::is_iec559);
```

## 17. Conversion procedure

To encode a positive decimal value as binary32:

1. convert the integer part to binary;
2. multiply the fractional part by 2 again and again to get fraction bits;
3. write the number as `1.xxxxx × 2^e`;
4. store sign `0` (positive);
5. store the biased exponent `e + 127`;
6. store the fraction in 23 bits, and round if needed.

Worked example:

```text
5.25 in decimal  =  101.01 in binary  =  1.0101 × 2^2
```

So:

```text
sign      = 0
exponent  = 2 + 127 = 129 = 10000001 in binary
fraction  = 01010000000000000000000
```

Some decimals stop in binary and are stored exactly (`5.25` is one of them). Others, such as `0.1`, repeat forever and must be rounded. That is the same idea as `1/3` in decimal.

## 18. Safely displaying a `float` representation

C++20 has `std::bit_cast` to copy the bits of a `float` into a 32-bit unsigned integer:

```cpp
#include <bit>
#include <bitset>
#include <cstdint>
#include <iostream>

float value{5.25F};
std::uint32_t bits{
    std::bit_cast<std::uint32_t>(value)
};

std::cout << std::bitset<32>{bits} << '\n';
```

This is safer than pretending a `float*` is a `uint32_t*`. You still must check size and IEEE support before you explain the bits.

## 19. Precision and rounding

A `float` can store only a limited number of fraction bits. If the true binary form is longer, the stored value is the nearest value that fits.

Default `cout` prints about 6 significant digits, so rounding can be hidden. Print more digits:

```cpp
#include <iomanip>
#include <limits>

std::cout << std::setprecision(
    std::numeric_limits<float>::max_digits10
);
```

Without `std::fixed`, `setprecision` means significant digits, not digits after the decimal point. `max_digits10` is enough digits to print a value and read back the same `float`. The assignment asks for 12 digits in one experiment; follow that, and explain what extra digits show.

## 20. Floating-point addition is not always associative

For real numbers:

$$(a+b)+c = a+(b+c)$$

Floating-point addition rounds after each step. Changing the parentheses can change which information is lost first:

```text
(rounded a+b) + c
a + (rounded b+c)
```

This is not a compiler bug. A `float` is a limited set of values, not every real number.

## 21. Equality needs a clear requirement

This C++ is defined:

```cpp
bool equal{a == b};
```

The problem is often the assumption. If `a` and `b` came from different rounded calculations, exact `==` may be too strict.

For an approximate check:

```cpp
#include <algorithm>
#include <cmath>

bool approximately_equal(double a, double b) {
    constexpr double absolute_tolerance{1e-12};
    constexpr double relative_tolerance{1e-9};

    const double difference{std::abs(a - b)};
    const double scale{
        std::max(std::abs(a), std::abs(b))
    };

    return difference <=
           absolute_tolerance + relative_tolerance * scale;
}
```

There is no one tolerance for every problem. Use `==` when you really need the same stored value.

**Checkpoint 3.** Why does printing more decimal digits *show* an existing rounding, rather than *create* the error?

---

# Part V. Tasks 4 and 5 - From operations to measurements

## 22. What a microbenchmark actually measures

A timing result is not only the source operator. Many layers sit in between:

```text
source expression
  -> compiler optimization
  -> generated instructions
  -> CPU execution
  -> clock resolution and operating-system noise
  -> reported duration
```

So “bitwise syntax is faster” is not a valid conclusion unless you check that both programs did equivalent work and that the compiler produced different instructions.

## 23. Measuring elapsed time with `<chrono>`

For intervals, `std::chrono::steady_clock` is a good choice. It does not jump backward:

```cpp
#include <chrono>

const auto start{std::chrono::steady_clock::now()};

// repeated operation

const auto end{std::chrono::steady_clock::now()};

const auto elapsed{
    std::chrono::duration<double, std::micro>{end - start}
};
```

`high_resolution_clock` has the smallest tick, but it may be another name for `steady_clock` or `system_clock`. A small tick is not the same as “never goes backward.”

## 24. Why repeat an operation?

One `+` or `>>` can be shorter than the clock can measure. Repeat it so the total time is large enough to see:

```cpp
#include <cstddef>

for (std::size_t i{}; i < iterations; ++i) {
    // operation under investigation
}
```

A simple average is:

$$t_{\mathrm{average}} = \frac{t_{\mathrm{total}}}{N}$$

Do not call this “the exact time of one CPU instruction.” The loop itself, memory, and compiler changes are also in the number.

## 25. The compiler may remove unused work

If a result cannot change what the program prints or does, the compiler may delete the calculation:

```cpp
int result{};

for (std::size_t i{}; i < iterations; ++i) {
    result += 1;
}
```

If nobody reads `result`, the optimizer may remove the loop. That is allowed: the visible program behavior is the same.

Always use the result after the timed region (print it). Even then, the compiler may compute the final value without running every step.

## 26. What `volatile` means in this laboratory

`volatile` tells the compiler: “treat reads and writes of this object as visible.” Then it is less free to remove them:

```cpp
volatile std::uint32_t value{123456789U};
```

In this lab, `volatile` helps you see the cost of repeated loads and stores. It also stops some simple loop removal.

It does **not**:

- make an operation atomic;
- make code safe for threads;
- block every optimization;
- make signed overflow defined;
- make a benchmark automatically fair.

Comparing “with and without `volatile`” is mainly about what the optimizer can see, not only about the data type.

## 27. Arithmetic and bitwise forms are not always equivalent

| Pair | Same meaning when | Watch out |
|---|---|---|
| `x / 2` and `x >> 1` | unsigned values | negative odd signed values round differently |
| `x * 2` and `x << 1` | a chosen unsigned test | signed range and overflow |
| `x % 2 == 0` and `(x & 1) == 0` | integer even/odd check | keep the types clear |

A modern compiler can turn `/ 2` or `* 2` into a shift when the meaning stays the same. Then two different source lines can become the same machine code.

If the divisor is `volatile`, the compiler cannot treat it as the constant 2. That asks a different question:

> How expensive is division by a value known only at run time, compared with a shift by a known count?

That is not the same as comparing `/ 2` with `>> 1` in normal optimized code.

## 28. Why smaller types may not be faster

`char` and `short` are usually converted to `int` before arithmetic:

```text
char + char
    -> int + int
    -> int
    -> convert back to char if you store it there
```

Many processors also do integer work in their native register size. A smaller stored type does not automatically mean a faster instruction.

Speed can also depend on:

- generated instructions;
- conversions;
- memory access;
- Debug vs Release;
- how instructions wait for each other;
- the CPU;
- the rest of the program.

## 29. A hidden issue in repeated addition

Adding `1` again and again is not the same story for every type.

- small integer types may wrap or convert when the value no longer fits;
- `int` can hold $10^{8}$, but not every larger loop count;
- a `float` cannot store every integer above $2^{24}$, so repeated `+ 1.0F` eventually stops changing the value;
- a `double` can store every integer up to $2^{53}$.

So a timing difference may be conversion, lost precision, or a removed loop — not simply “this type is smaller.” Print the final value as a check.

## 30. A reproducible measurement procedure

For each comparison:

1. record compiler, version, OS, CPU, and Debug or Release;
2. use the same loop count and surrounding code;
3. build the intended configuration; compare Debug and Release;
4. run a warm-up outside the timed region;
5. run each test several times;
6. report several results or a median, not only the best one;
7. print the final value after the timed region;
8. check that both versions compute the same result for the tested inputs;
9. avoid undefined behavior;
10. call the numbers observations from this environment.

---

# Part VI. Guided seminar exercises

Exercises 1–4 are the core live practice. Use 5–8 if time remains, or as extra preparation for the laboratory.

## Exercise 1 - Project reasoning

For a task containing `main.cpp`, `representation.cpp`, and `representation.hpp`:

1. identify which file should declare `print_bits`;
2. identify which file should define it;
3. identify which file should call it;
4. write the `add_executable` declaration;
5. explain the linker's role.

## Exercise 2 - Representation practice

Using eight bits for illustration:

1. write `13` in binary;
2. derive the two's-complement representation of `-13`;
3. calculate `13 & 10`, `13 | 10`, and `13 ^ 10`;
4. explain why a 32-bit display needs leading bits.

## Exercise 3 - Behavior classification

Classify each case:

```cpp
std::uint32_t u{UINT32_MAX};
auto a{u + 2U};
```

```cpp
std::int32_t s{INT32_MAX};
auto b{s + 2};
```

```cpp
std::uint32_t x{5};
auto c{x << 32};
```

Use the Seminar 1 words: **defined**, **undefined**, and **implementation-defined**. Do not run the undefined examples.

## Exercise 4 - Binary32 practice

Encode `5.25F` by finding:

1. the binary value;
2. the normalized form;
3. the sign bit;
4. the biased exponent;
5. the 23-bit fraction;
6. the complete 32-bit representation.

Then explain why the same easy procedure may produce an infinite sequence for a decimal fraction such as `0.1`.

## Exercise 5 - Associativity prediction

Choose a very large positive `float`, its negative, and a small value. Predict how different parentheses can lose the small value at different times. Check with `float` and enough printed digits.

## Exercise 6 - Same meaning?

For each input, compare `x / 2` and `x >> 1`:

```text
unsigned x = 9
signed x = 8
signed x = -8
signed x = -9
```

Which result shows that the two forms are not always the same?

## Exercise 7 - Benchmark diagnosis

A program says an empty-looking loop takes zero microseconds in Release mode. Give at least three possible explanations or checks before you conclude that the operation costs nothing.

## Exercise 8 - Result interpretation

Suppose `char`, `short`, and `int` show similar addition times. Write two explanations stronger than “the timer is wrong.” Include promotion to `int` and the CPU’s native integer width.

---

# Part VII. Writing each task's `README.md`

Every task report should contain:
Это пример, можем быть другая структура по ТЗ в PDF

```markdown
# Task N - Title

## Environment
- OS:
- architecture:
- compiler and version:
- C++ standard:
- build configuration:

## Method
What was executed, with which inputs and iteration counts?

## Results
Representations, outputs, or timing tables.

## Explanation
Which language rules and implementation properties explain the results?

## Answers to control questions
Answer every question from the specification.

## Limitations
What might affect or restrict the result?

## Conclusion
What does the experiment support, and what does it not prove?
```

A conclusion is required for grading. Do not only repeat the table.

Weak conclusion:

> Bitwise operations are always faster.

Stronger conclusion:

> In this Release build on the tested compiler and processor, the two source forms had similar median times. The compiler can replace arithmetic with equivalent bitwise instructions when the meaning stays the same. This experiment does not prove that bitwise syntax is always faster.

---

# Part VIII. Submission checklist

Before creating the archive, verify:

- [ ] root `CMakeLists.txt` includes all five task directories;
- [ ] every task has its own executable target;
- [ ] every task contains `main.cpp`, a header, an implementation file, and `README.md`;
- [ ] every header begins with `#pragma once`;
- [ ] no task depends on source files from another task directory;
- [ ] no global variables are used;
- [ ] every control question is answered;
- [ ] every task has a conclusion;
- [ ] signed overflow is described as undefined behavior;
- [ ] floating-point layout assumptions are checked and stated;
- [ ] timing environment and Debug/Release are recorded;
- [ ] final values are checked as well as timings;
- [ ] generated build folders and IDE-local files are not in the archive;
- [ ] the project configures and builds from a clean directory.

Final clean-build test:

```bash
cmake -S . -B build
cmake --build build --config Release
```
