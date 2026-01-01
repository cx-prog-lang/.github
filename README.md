# C+ Language

_C+_ is an extended language that's _perfectly_ compatible with C code. Please take a look at [this](https://gwangmu.medium.com/sketching-c-the-intermediate-language-between-c-and-c-51664ff4a3f8) conceptual writing.

## Build Pipeline

C+ aims to _piggyback_ on the existing C infrastructure. The C+-specific build pipeline is divided into two parts: _sugar_ and _salt_. The _sugar_ part translates C+ code into the C equivalent. The _salt_ part hardens the C code with C+ specifications. The figure below describes the evolution of source code in the build pipeline.

![Build Sequence drawio](https://github.com/user-attachments/assets/03fef56c-a3fa-465f-b82c-6eed7c4431c6)

**Figure**. Evaluation of the build pipeline.

## Interoperability

C+ places the utmost importance on _seamless integration_. The core principles:

 * _Every_ C code should be able to pass through the C+ build pipeline.
 * The C+ code should be able to use _everything_ from the C code without adaptation.
 * The C code should be able to use _everything_ from the translated C+ code without adaptation.

In addition to the principles, the translated C+ code should look as close as possible to the original C+ code (e.g., the name of the functions, the "reasonable" representation of structs, and so on) so that developers could intuitively deduce how to use the C+ code in C.

Below are some key interoperability matrices.

|                     | C source   | C+ source |
|---------------------|------------|-----------|
| **Using C header**  | Direct     | Direct    |
| **Using C+ header** | Translated | Direct    |

**Table**. Using C/C+ headers in C/C+ source code.

|                     | C source | C+ source |
|---------------------|----------|-----------|
| **Using C Syntax**  | Yes      | Almost*   |
| **Using C+ Syntax** | _N/A_    | Yes       |

**Table**. Using C/C+ syntax in C/C+ source code. (*: some restrictions depending on the accompanying C+ syntax.)
