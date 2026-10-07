# Seminar 1: From C++ Source Code to a Running Program

## Introduction

When you run a Python file, the interpreter and editor hide many of the steps between source text and execution. A typical C++ workflow exposes more of those steps: source files are translated, object files are linked, and an executable is produced before the operating system starts the program.

This seminar therefore tells two connected stories:

1. **How a C++ program is built and started.** We separate the language from the compiler, editor, and build system, and follow a `.cpp` file until it becomes a running process.
2. **What the code inside the program means.** We connect identifiers, literals, types, objects, memory, expressions, and streams.

These stories meet in a simple idea:

> The build tools determine how source code becomes an executable; the C++ language rules determine what that executable means.

---

> **Part I — From source text to a running process**  
> Central question: *How does `main.cpp` become something the computer can execute?*

## 1. The C++ ecosystem

C++ is a compiled, statically typed, general-purpose programming language. It is used in systems software, browsers, databases, game engines, embedded devices, high-performance services, and applications where predictable performance or direct resource control matters.

The language itself is a specification: it describes valid programs and their meaning. A specification does not translate files or display an editor window. Other tools implement and support the language, which is why “installing C++” actually means installing several cooperating components.

The following tools have different jobs:

| Item | Role | Examples |
|---|---|---|
| C++ | Programming language defined by a standard | C++17, C++20, C++23 |
| Compiler toolchain | Translates and links code | GCC, Clang, MSVC |
| Standard library implementation | Implements library facilities | libstdc++, libc++, MSVC STL |
| Editor or IDE | Helps write and inspect code | VS Code, Visual Studio, CLion |
| Build-system generator | Configures a portable build | CMake |
| Build tool | Executes the build steps | Ninja, Make, MSBuild |

> VS Code does not contain a C++ compiler. The Run button ultimately invokes tools installed elsewhere.

### GCC, Clang, and MSVC

All three compile standard C++, although command-line syntax and platform integration differ.

| Toolchain | Typical command | Common platforms |
|---|---|---|
| GCC | `g++` | Linux, Windows via MinGW/MSYS2 |
| Clang | `clang++` | macOS, Linux, Windows |
| MSVC | `cl` | Windows |

This course uses CMake so the same project structure can work with all three.

### A useful mental model

Think of the tools as different roles in a workshop:

- the **C++ standard** is the design specification;
- the **compiler** checks and translates individual source files;
- the **linker** joins translated pieces and libraries;
- the **build tool** runs the required commands in the correct order;
- **CMake** prepares a build for the selected platform and toolchain;
- the **editor or IDE** provides a convenient interface to these tools.

The boundaries matter when something fails. A red underline in VS Code, a compiler diagnostic, and a linker error may refer to three different layers.

**Bridge to the next section.** Now that the tools are separated from the language, we can compare C++ itself with its close relative, C.

---

## 2. C and C++: related, but distinct

C++ grew from C and retains much of its syntax and low-level model. Modern C++, however, is not merely “C with classes.” It adds references, function overloading, classes, templates, exceptions, namespaces, RAII, containers, algorithms, and other abstraction mechanisms.

### A small C program

```c
#include <stdio.h>

int main(void) {
    int value = 10;
    printf("%d\n", value);
    return 0;
}
```

### A small C++ program

```cpp
#include <iostream>

int main() {
    int value{10};
    std::cout << value << '\n';
}
```

Shared ideas include braces, semicolons, functions, loops, conditions, arithmetic, and pointers. The C++ example also demonstrates:

- `<iostream>` from the C++ standard library;
- the `std` namespace;
- brace initialization;
- stream output;
- implicit `return 0` at the end of `main`.

It is tempting to describe C++ as “C plus classes,” but that mental model encourages C-style solutions even when C++ provides safer abstractions. Modern C++ retains much of C's low-level vocabulary while adding ways to express ownership, lifetime, generic algorithms, and user-defined types.

### A Python comparison

Python, C, and C++ can all express the same elementary algorithm, but their normal execution models and type systems differ:

| Question | Python | C++ |
|---|---|---|
| How is a typical program started? | Interpreter executes the program | Executable is normally built first |
| When are many type errors found? | While the relevant code runs | During compilation |
| Can one name later refer to a value of another type? | Common | An object's declared type does not change |
| How are blocks written? | Indentation | Braces |

This is an orientation, not a claim that one language is universally better. Different designs serve different goals.

**Bridge to the next section.** Both the C and C++ examples are still source text. Neither file contains instructions that the processor can execute directly, so we need to examine the build pipeline.

---

## 3. Source code is not an executable

A processor cannot directly execute C++ source text. A simplified build pipeline is:

```text
source file (.cpp)
    -> preprocessing
    -> compilation
    -> assembly
    -> object file (.o/.obj)
    -> linking with other object files and libraries
    -> executable
    -> operating-system loader
    -> running process
```

A compiler driver such as `g++` or `clang++` can run several of these stages with one command:

```bash
g++ -std=c++20 main.cpp -o app
./app
```

On Windows PowerShell, the final command is normally:

```powershell
.\app.exe
```

Useful GCC/Clang options:

```bash
g++ -E main.cpp              # preprocessed source
g++ -S main.cpp              # assembly source
g++ -c main.cpp              # object file, no linking
g++ main.o -o app            # linking
```

The precise internal pipeline can vary, but this model is sufficient for understanding multi-file builds and common errors.

### Why object files and linking exist

Real programs are divided into source files. Each source file can be translated separately into an object file. If only one source file changes, a build can often recompile that file and then link the project again instead of recompiling everything.

The linker resolves connections between separately translated pieces. For example, one file may call a function whose definition is stored in another object file or library. The compiler can accept the call when it sees a suitable declaration; the linker later verifies that a matching definition exists.

This explains an important distinction:

```text
invalid C++ in one source file        -> compile error
missing definition across the project -> link error
```

**Bridge to the next section.** With the pipeline in mind, we can return to the smallest useful C++ program and ask what information each part provides to the compiler.

---

## 4. First C++ program, dissected

**Core.** A first program is more valuable when it is read as a structured message to the toolchain rather than copied as a ritual.

```cpp
#include <iostream>

int main() {
    std::cout << "Hello, C++!\n";
    return 0;
}
```

| Fragment | Meaning |
|---|---|
| `#include <iostream>` | Makes declarations for standard stream I/O available |
| `int` | Return type of `main` |
| `main` | Program entry function |
| `()` | Empty parameter list |
| `{ ... }` | Function body and scope |
| `std::cout` | Standard output stream object |
| `<<` | Stream insertion operator |
| `"Hello, C++!\n"` | String literal ending with a newline |
| `;` | Terminates the statement |
| `return 0` | Reports successful termination |

Reaching the closing brace of `main` is equivalent to returning zero, but writing `return 0;` can be useful while learning the structure.

### Why `std::`?

Names from the standard library are placed in namespace `std`. Prefer explicit names such as `std::cout`, `std::cin`, and `std::string`. Avoid making `using namespace std;` a default habit because it introduces many names into the current scope and may cause collisions.

### From text to structure

The compiler does not see the program as a picture or paragraph. It recognizes tokens and grammatical structures: a preprocessing directive, a function definition, a statement, an expression, and literals. Later sections examine smaller elements such as identifiers and literal types.

**Bridge to the next section.** A direct compiler command is manageable for one file. A course project needs a repeatable description of its sources, language version, target, and warnings. That is the role of CMake.

---

## 5. The standard CMake project

**Core.** CMake is introduced early because it will be the standard build method for later seminars. The goal today is not to learn the whole CMake language; it is to understand one small target and the difference between configuring, building, and running.

Use this structure for seminar exercises:

```text
seminar01/
|-- CMakeLists.txt
`-- src/
    `-- main.cpp
```

`CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.20)

project(seminar01 VERSION 1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

add_executable(seminar01 src/main.cpp)

if(MSVC)
    target_compile_options(seminar01 PRIVATE /W4)
else()
    target_compile_options(seminar01 PRIVATE -Wall -Wextra -Wpedantic)
endif()
```

What each important command does:

- `cmake_minimum_required` states the oldest supported CMake version;
- `project(... LANGUAGES CXX)` declares a C++ project;
- `CMAKE_CXX_STANDARD 20` requests C++20;
- `CMAKE_CXX_STANDARD_REQUIRED ON` rejects fallback to an older standard;
- `CMAKE_CXX_EXTENSIONS OFF` asks for portable standard C++ mode;
- `add_executable` defines the target and its source file;
- `target_compile_options` enables useful warnings for the selected compiler.

### Configure and build

Run these commands in the project root:

```bash
cmake -S . -B build
cmake --build build
```

- `-S .` selects the source directory;
- `-B build` creates a separate build directory;
- `cmake --build build` invokes the generated build tool.

These are separate actions:

| Action | Question answered |
|---|---|
| Configure/generate | Which compiler, platform, options, sources, and build tool will be used? |
| Build | Which files are out of date, and which compile/link commands must run? |
| Run | What happens when the operating system starts the produced executable? |

Changing `main.cpp` normally requires another build, not a complete reconfiguration. Changing the CMake project description may require CMake to configure or regenerate the build rules.

Do not place generated build files next to source files and do not submit the `build/` directory.

The executable location depends on the generator:

- single-configuration generators often produce `build/seminar01`;
- Visual Studio may produce `build/Debug/seminar01.exe`.

For a predictable Debug build across generators:

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build --config Debug
```

`CMAKE_BUILD_TYPE` is used by single-configuration generators; `--config Debug` is used by multi-configuration generators such as Visual Studio.

### VS Code workflow

Install a compiler toolchain, CMake, VS Code, and the **C/C++** and **CMake Tools** extensions. Then:

1. open the folder containing `CMakeLists.txt`;
2. choose **CMake: Select a Kit** or the offered compiler;
3. run **CMake: Configure**;
4. run **CMake: Build**;
5. select the `seminar01` launch target and run or debug it.

If the compiler is not detected, first verify it in a terminal with `g++ --version`, `clang++ --version`, or an MSVC Developer Command Prompt using `cl`.

### Part I checkpoint

You should now be able to complete this explanation:

> VS Code helps me edit the project. CMake configures a build. The build tool invokes the compiler and linker. Their output is an executable, which the operating system can start as a process.

The tooling story is now complete enough for the first seminar. We can move from the tools around the program to the meaning of the code inside it.

---

> **Part II — From program text to typed objects**  
> Central question: *When the compiler reads `int age{20};`, what does it learn and what does the program create?*

## 6. Variables, types, and memory

Python binds names to runtime objects and determines types dynamically. C++ usually declares an object's type explicitly:

```cpp
int age{20};
double temperature{21.5};
bool is_online{true};
char grade{'A'};
```

Consider one declaration:

```cpp
int age{20};
```

It connects several ideas:

- `int` specifies the object's type;
- `age` is an identifier used to name the object;
- `20` is a literal expression providing the initial value;
- `{20}` performs initialization;
- the object occupies storage for some lifetime.

The topics in this part are therefore not isolated syntax rules. They are different answers to the question “what object are we creating?”

Conceptually, `int age{20};` creates an object with:

- a type (`int`);
- a value (`20`);
- some storage during its lifetime;
- a name (`age`) used to access it.

The object also has an address:

```cpp
std::cout << &age << '\n';
```

The address is printed only as a preview; pointers are covered later in the course.

### Type, value, storage, and lifetime

These terms answer different questions:

| Concept | Question |
|---|---|
| Type | Which values and operations are permitted? |
| Value | What information does the object currently represent? |
| Storage | Where are the object's bytes held while it exists? |
| Lifetime | During which part of execution does the object exist? |

For now, it is enough to know that an ordinary local automatic object such as `age` is created when execution reaches its declaration and is destroyed when execution leaves its block. Later seminars refine this model with pointers, dynamic allocation, classes, and RAII.

Do not reduce every local object to “a box on the stack.” That picture can be useful, but the language concepts are type, object, storage duration, and lifetime; compiler optimizations may not preserve a literal box in memory.

### Fundamental types

A type is not only a storage size. It determines the set of values, the available operations, and how expressions interpret the stored bits. For example, the same `/` token selects integer or floating-point division depending on the operand types.

| Category | Common types | Example |
|---|---|---|
| Boolean | `bool` | `true` |
| Character | `char` | `'A'` |
| Integer | `short`, `int`, `long`, `long long` | `42` |
| Unsigned integer | `unsigned int`, etc. | `42u` |
| Floating point | `float`, `double`, `long double` | `3.14` |

The standard specifies minimum ranges and relationships, not one universal byte size for every type. Inspect the current implementation with `sizeof`:

```cpp
std::cout << sizeof(int) << '\n';
std::cout << sizeof(double) << '\n';
```

`sizeof` reports a size in bytes, and a C++ byte is defined as `sizeof(char) == 1`. A byte is commonly eight bits, but `sizeof` alone does not state that.

**Supporting explanation.** Exact representation, overflow, and IEEE floating point belong to Seminar 2. Here, `sizeof` is used to show that the implementation participates in choosing representations.

### Identifiers

An identifier is a name used for a variable, function, type, or another program entity.

Names make program entities usable by humans. Without identifiers, source code would have no practical way to refer to the same object in later expressions. An identifier identifies an entity; it does not itself determine that entity's type or value.

For this course, use the following practical rule:

> Begin an identifier with a letter. Continue with letters, digits, or underscores. Do not create names beginning with an underscore.

Examples:

| Name | Valid? | Reason |
|---|---:|---|
| `my_var` | Yes | Contains letters and an underscore |
| `_count` | Lexically valid | Avoid it: leading underscores have reservation rules |
| `2fast` | No | An identifier cannot begin with a digit |
| `for` | No | `for` is a C++ keyword |
| `user-name` | No | The hyphen is parsed as subtraction |
| `User_Name` | Yes | C++ identifiers are case-sensitive |

Thus, `user_name`, `User_Name`, and `USER_NAME` are three different identifiers.

Good names also communicate meaning. Prefer `track_count` to `tc`, and use a consistent naming convention throughout a project. Naming style is a team convention; lexical validity is a language rule.

**Bridge.** An identifier answers “which entity?”, but a declaration also needs to establish what kind of object it is and often which initial value it receives. Types and literals provide that information.

### Literal types and suffixes

A literal has a type even when no variable declaration appears next to it:

| Literal | Type |
|---|---|
| `42` | `int` |
| `42U` | `unsigned int` |
| `42L` | `long int` |
| `42LL` | `long long int` |
| `3.14` | `double` |
| `3.14f` | `float` |
| `'A'` | `char` |
| `"A"` | `const char[2]` |

The last two are fundamentally different:

```cpp
'A'  // one character
"A"  // an array containing 'A' followed by the terminating '\0'
```

Use straight ASCII quotation marks in C++ source. Typographic quotation marks such as `’A’` are not valid replacements for `'A'`.

Literal suffixes are instructions to the compiler, not decorations for the reader. They can change which overload or arithmetic operation is selected later. This is why `42`, `42U`, and `42.0` should not be treated as interchangeable spellings of the same thing.

**Bridge.** A typed literal can provide an initial value, but C++ has several initialization forms. The chosen form can determine whether a dangerous conversion is accepted.

### Initialization

Initialization gives an object its first value as its lifetime begins. Assignment changes the value of an object that already exists; the two operations are related but not identical.

```cpp
int a;        // uninitialized local object: do not read it
int b = 10;   // copy initialization
int c(10);    // direct initialization
int d{10};    // list initialization
```

Brace initialization is useful because it rejects many narrowing conversions:

```cpp
int x = 3.9;  // allowed with loss of fractional part; warning may be emitted
int y{3.9};   // compile error
```

Reading an uninitialized local `int` has undefined behavior: the C++ standard gives no valid result to rely on.

The empty-brace form is useful for safe default initialization:

```cpp
int count{};       // zero
double total{};    // 0.0
bool finished{};   // false
```

This does not mean that every `{}` in C++ always means zero; classes can define their own initialization behavior. For the fundamental types shown here, it provides the expected zero value.

### `const` and `auto`

```cpp
const double pi{3.141592653589793};
auto count{10};       // deduced as int
auto price{12.50};    // deduced as double
```

`const` prevents modification through that object name. `auto` asks the compiler to deduce a static type; it does not make C++ dynamically typed.

Use `const` to communicate that a value should not change after initialization. Use `auto` when the initializer makes the type clear or when the exact type is cumbersome—not to hide important domain meaning.

### A Python comparison: rebinding versus a typed object

```python
value = 10
value = "hello"
```

In Python, the name is rebound. In the following C++ code, `value` names an `int` object whose type does not change:

```cpp
int value{10};
value = 20;       // OK: new int value
value = "hello";  // compile error: incompatible type
```

### Part II checkpoint

For `const double price{12.5};`, identify:

- the identifier;
- the type;
- the literal and its type;
- the initialization syntax;
- the effect of `const`.

Objects now have values. The next part explains how expressions read, transform, compare, and present those values.

---

> **Part III — From stored values to computation and communication**  
> Central question: *How does a program transform typed values and exchange information with a user?*

## 7. Input, output, and expressions

An expression is a combination of operands and operators that computes a value, may produce a side effect, or both. In `a + b`, the identifiers select objects, reading them produces values, and `+` computes a result. In `std::cout << value`, stream insertion both evaluates an expression and changes the state of an output stream.

```cpp
#include <iostream>

int main() {
    int a{};
    int b{};

    std::cout << "Enter two integers: ";
    std::cin >> a >> b;

    std::cout << "sum = " << a + b << '\n';
}
```

Main operator groups:

| Group | Operators |
|---|---|
| Arithmetic | `+`, `-`, `*`, `/`, `%` |
| Comparison | `==`, `!=`, `<`, `>`, `<=`, `>=` |
| Logical | `&&`, `||`, `!` |
| Assignment | `=`, `+=`, `-=`, `*=`, `/=` |
| Increment/decrement | `++`, `--` |

This table is a map, not a list to memorize today. The meaning of an operator depends on its operands. The same token may behave differently for other types or user-defined classes.

Operand types affect the result:

```cpp
std::cout << 5 / 2 << '\n';     // 2: integer division
std::cout << 5.0 / 2 << '\n';   // 2.5
```

Use an explicit C++ conversion when needed:

```cpp
int total{5};
int count{2};
double average{static_cast<double>(total) / count};
```

Unlike C-style `(double)total`, `static_cast<double>(total)` states the intended conversion clearly and can be checked more precisely by the compiler.

The important sequence is:

1. determine the operand types;
2. apply required conversions;
3. perform the selected operation;
4. obtain a result type and value.

This explains why converting before division matters:

```cpp
static_cast<double>(7 / 2) // integer division first: 3.0
static_cast<double>(7) / 2 // conversion first: 3.5
```

### Boolean values and output

Comparisons and logical operators produce `bool` values:

```cpp
bool first{5 > 3};   // true
bool second{2 == 0}; // false
```

By default, streams print them as `1` and `0`. Use `std::boolalpha` for words:

```cpp
std::cout << first << '\n';                  // 1
std::cout << std::boolalpha << first << '\n'; // true
std::cout << std::noboolalpha;
```

An integer converts to `false` when it is zero and to `true` when it is nonzero. Converting `false` to an integer produces `0`; converting `true` produces `1`.

`std::boolalpha` changes the formatting state of the stream and remains active until it is changed again. This is the same general idea used by `std::fixed` and `std::setprecision` later.

### Short-circuit evaluation

`&&` and `||` do not necessarily evaluate both operands:

- `left && right` skips `right` when `left` is false;
- `left || right` skips `right` when `left` is true.

This can prevent an invalid operation:

```cpp
int denominator{0};

bool valid{
    denominator != 0 &&
    100 / denominator > 5
};
```

The division is not evaluated because `denominator != 0` is false.

Do not confuse logical and bitwise operators:

| Operators | Purpose | Short-circuit? |
|---|---|---:|
| `&&`, `||` | Logical AND/OR | Yes |
| `&`, `|` | Bitwise AND/OR for integers | No |

Bitwise representation is covered in Seminar 2.

Short-circuiting is more than a speed optimization: it can encode a dependency. In the denominator example, the right expression is only meaningful when the left expression establishes that division is safe.

### Integral promotion

Small integer types, including `char`, are normally promoted before arithmetic:

```cpp
auto result{'a' + 1};
```

Here, the type of `result` is `int`, not `char`.

### Precedence is not evaluation order

In this expression:

```cpp
int x{5 + 3 * 2 - 4 / 2};
```

multiplication and division bind more strongly:

```cpp
int x{5 + (3 * 2) - (4 / 2)}; // 9
```

Precedence determines how an expression is grouped. It does not generally determine which independent operand is evaluated first. Use parentheses to communicate intended grouping, and avoid modifying the same object several times in one expression.

Associativity answers how operators of the same precedence group. Sequencing is a different concept: it states when one evaluation must happen before another. Keeping these questions separate prevents many incorrect explanations of complex expressions.

### Prefix and postfix increment

```cpp
int value{5};

int old_value{value++}; // old_value is 5, value becomes 6
int new_value{++value}; // value becomes 7, new_value is 7
```

Use prefix increment when the old value is not required. Postfix increment is appropriate when the previous value is genuinely needed.

Do not write expressions such as:

```cpp
int a{0};
int result{a++ + ++a}; // undefined behavior
```

The two operands of `+` modify the same object without a required sequencing relationship. No printed value or compiler result may be relied upon.

### Formatted floating-point output

`std::fixed` and `std::setprecision` control presentation without changing the stored value:

```cpp
#include <iomanip>
#include <iostream>

int main() {
    double value{10.2427};

    std::cout << std::fixed
              << std::setprecision(3)
              << value << '\n';
}
```

Output:

```text
10.243
```

Formatting changes how a value is represented as text; it does not change the stored `double`. The displayed result is rounded to the requested number of digits.

### Part III checkpoint

Explain why these expressions differ:

```cpp
5 / 2
5.0 / 2
static_cast<double>(5 / 2)
static_cast<double>(5) / 2
```

Your explanation should mention operand types, when conversion occurs, and whether division is integral or floating point.

---

## 8. Four useful error categories

The earlier build pipeline now helps classify failures. The layer reporting a problem often tells you what kind of investigation to perform.

| Category | When discovered | Example |
|---|---|---|
| Compile error | Translation of a source file | missing `;`, incompatible types |
| Link error | Object files are combined | called function has no definition |
| Runtime error | Program is executing | input failure or environment/resource failure |
| Logic error | Program runs but answer is wrong | using `+` instead of `*` |

Undefined behavior is a separate language concept: the standard imposes no requirements on the result. It is not guaranteed to produce an immediate crash or diagnostic.

Related terms should not be confused:

| Term | Beginner-level meaning |
|---|---|
| Defined behavior | The language rules specify what the program means |
| Implementation-defined behavior | The implementation chooses and documents one permitted behavior |
| Unspecified behavior | Several behaviors are permitted; the implementation need not document which one occurs |
| Undefined behavior | The language imposes no requirements on the result |

Undefined behavior does not mean “a random but valid value.” Observing one result on one compiler does not make that result reliable.

Warnings are not errors, but should be investigated. This course enables `/W4` on MSVC and `-Wall -Wextra -Wpedantic` on GCC/Clang.

---

## 9. In-class exercises

The exercises follow the same path as the seminar: identify the tools, build a program, diagnose failures, and then reason about typed expressions. For prediction questions, write an answer before compiling; observation is useful only after a model has been tested.

The live demo project is `seminar01/`. Complete programs and written answers are in `seminar01/solutions/` after you try the work yourself.

### Exercise 1 — Identify the toolchain

1. Determine which compiler CMake selected.
2. Record the compiler family and version.
3. Explain whether VS Code, CMake, or the compiler translates C++ into machine code.

Useful commands include:

```bash
cmake -S . -B build
g++ --version
clang++ --version
```

On Windows with MSVC, use a Developer Command Prompt and run:

```bat
cl
```

### Exercise 2 — Build and modify Hello World

Create the standard project and print:

```text
Hello, C++!
Compiler-backed, CMake-built.
```

Configure, build, and run it. Change one line, rebuild, and run again.

### Exercise 3 — Error laboratory

Create and then fix each failure separately:

1. remove a semicolon after output;
2. write `cout` instead of `std::cout`;
3. call the declared function below without defining it:

```cpp
void print_course_name();

int main() {
    print_course_name();
}
```

For each case, classify the failure and find the most useful part of the diagnostic.

### Exercise 4 — Typed calculator

Write a program that reads two integers and prints:

- their sum;
- their difference;
- their product;
- integer quotient and remainder;
- floating-point quotient.

Assume the second integer is nonzero for this first seminar. Example:

```text
Enter two integers: 7 2
sum = 9
difference = 5
product = 14
integer quotient = 3
remainder = 1
floating quotient = 3.5
```

### Exercise 5 — Type and memory explorer

Print the sizes of `char`, `int`, `long`, `long long`, `float`, and `double`. Then create two initialized integers and print their values and addresses.

Answer:

1. Are the type sizes necessarily identical on every platform?
2. Are the two addresses necessarily consecutive?
3. Does copying one integer into another make them the same object?

### Exercise 6 — Concept checks

Predict each answer before compiling anything.

1. Which names are valid identifiers: `my_var`, `_count`, `2fast`, `for`, `user-name`, `User_Name`?
2. Determine the type of each literal: `42`, `42U`, `42L`, `42LL`, `3.14`, `3.14f`, `'A'`, `"A"`.
3. Determine the value and type of `(5 > 3) && (2 == 0)`.
4. Determine the type of `'a' + 1`.
5. Determine the value of `5 + 3 * 2 - 4 / 2`.
6. Determine the output of:

```cpp
std::cout << 20 / 50 << '\n';
```

7. Print `10.2427` with exactly three digits after the decimal point.

### Optional challenge — sequencing and side effects

Classify each expression as well-defined or undefined. Do not guess a numeric result for undefined behavior.

```cpp
int a{0};
std::cout << a << ' ' << a++ << ' ' << a << '\n';
```

```cpp
int a{0};
std::cout << (a++ + ++a) << '\n';
```

```cpp
int x{5};
int y{(x = 10) + (x = 20)};
```

---

## 10. Exit ticket

Answer without running code:

1. What is the difference between CMake and a compiler?
2. What stage reports a missing function definition?
3. What does `std` mean in `std::cout`?
4. Why does `5 / 2` produce `2`?
5. Is `auto value = 10;` dynamically typed?
6. Why is `int x{3.9};` rejected?
7. What is wrong with reading an uninitialized local `int`?
8. What is the difference between `'A'` and `"A"`?
9. Why might the right operand of `&&` not be evaluated?
10. Does precedence specify the evaluation order of independent operands?

---

## 11. Seminar synthesis

The complete mental model is now:

```text
C++ source files
    -> CMake configures a build
    -> build tool runs required commands
    -> compiler produces object files
    -> linker produces an executable
    -> operating system starts a process
    -> typed objects exist during execution
    -> expressions transform their values
    -> streams exchange formatted data with the user
```

A C++ program is therefore both:

- a **build artifact** produced from source files and libraries; and
- a **system of typed objects and expressions** whose meaning is governed by the language.

When debugging, move through the same model. Ask first whether the project configured, then whether it compiled and linked, and finally whether the running program manipulated the intended types and values.

---

## 12. Homework

### Part A — Unit converter

Create a CMake project called `unit_converter`. Read a temperature in degrees Celsius and print Fahrenheit and Kelvin:

\[
F = C \times \frac{9}{5} + 32, \qquad K = C + 273.15
\]

Requirements:

- use C++20 and the standard course CMake structure;
- read a `double`;
- store `273.15` in a named `const` object;
- print both results;
- build with compiler warnings enabled;
- add a short comment explaining why `9.0 / 5.0` is safer here than `9 / 5`.

Example:

```text
Temperature in Celsius: 25
Fahrenheit: 77
Kelvin: 298.15
```

### Part B — Predict before running

For each expression, predict both its value and type, then verify using a program:

```cpp
7 / 2
7.0 / 2
7 % 2
2 + 3 * 4
static_cast<double>(7 / 2)
static_cast<double>(7) / 2
```

Explain why the last two results differ.

### Part C — Reflection

In 4–6 sentences, explain the route from `main.cpp` to a running process and identify the jobs of CMake, the build tool, compiler, and linker.

### Part D — Output formatting

Write a program that prints `10.2427` in all three forms:

```text
10.243
10.24
10.242700
```

Use `std::fixed` and `std::setprecision`; do not change the stored value between output operations.

---

## Quick reference

```bash
# Configure
cmake -S . -B build -DCMAKE_BUILD_TYPE=Debug

# Build
cmake --build build --config Debug

# Reconfigure from a clean build directory if necessary
# Delete only the project's own build directory, then configure again.
```

```cpp
#include <iostream>

int main() {
    int value{};
    std::cin >> value;
    std::cout << value << '\n';
}
```
