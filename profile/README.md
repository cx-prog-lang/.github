# C+ Language

_C+_ is a language extension that's _perfectly_ compatible with legacy C code. C+ aims to be a new unofficial version of C so that each feature can readily be merged into standard C at any moment. Please take a look at [the conceptual writing](https://gwangmu.medium.com/sketching-c-the-intermediate-language-between-c-and-c-51664ff4a3f8).

## Why?

Excerpt from [the conceptual writing](https://gwangmu.medium.com/sketching-c-the-intermediate-language-between-c-and-c-51664ff4a3f8):

> C might have to remain the bare-minimum system language that interfaces between assembly and human-readable code [...], but system developers also need high-level language features sometimes. C++ is so cluttered, and Rust is a different language. Why can’t we have a small language extension without going too far from C?

## Build Pipeline

C+ aims to _piggyback_ on the existing C infrastructure. The C+-specific build pipeline is divided into two parts: _sugar_ and _salt_. The _sugar_ part translates C+ code into the C equivalent. The _salt_ part hardens the C code with C+ specifications. The idea behind it is that the "sugar" part reduces the C+ _syntax_ into C, and the "salt" part enforces the C+ _semantics_ in a non-intrusive way. The figure below describes the evolution of source code in the build pipeline.

![Build Sequence drawio](https://github.com/user-attachments/assets/fe84e817-cf70-4893-b98c-0c9013171c8a)

**Figure**. Evaluation of the build pipeline. (Yellow: C+-specific)

## Interoperability

C+ places the utmost importance on _seamless integration_. The core principles:

 * _Every_ C code should be able to pass through the C+ build pipeline.
 * The C+ code should be able to use _everything_ from the C code without adaptation.
 * The C code should be able to use _everything about C_ from the C+ code without adaptation.

In addition to the principles, the translated C+ code should look as close as possible to the original C+ code (e.g., the name of the functions, the "reasonable" representation of structs, and so on) so that developers could intuitively deduce how to use the C+ code in C.

Below are some key interoperability matrices.

|                     | C source   | C+ source |
|---------------------|------------|-----------|
| **Using C header**  | Direct     | Direct    |
| **Using C+ header** | Translated | Direct    |

**Table**. Using C/C+ headers in C/C+ source code.

|                     | C source | C+ source |
|---------------------|----------|-----------|
| **Using C Syntax**  | Yes      | Yes       |
| **Using C+ Syntax** | _N/A_    | Yes       |

**Table**. Using C/C+ syntax in C/C+ source code.
