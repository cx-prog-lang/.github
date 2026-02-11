# C+ Language

_C+_ is a language extension that's _perfectly_ compatible with legacy C code. C+ aims to be an unofficial version of C, so that each feature can be readily merged into standard C at any time. 

## Why?

Excerpt from [the conceptual writing](https://gwangmu.medium.com/sketching-c-the-intermediate-language-between-c-and-c-51664ff4a3f8):

> C might have to remain the bare-minimum system language that interfaces between assembly and human-readable code [...], but system developers also need high-level language features sometimes. C++ is so cluttered, and Rust is a different language. Why can’t we have a small language extension without going too far from C?

## Goal

 - **Near-perfect C Compatibility**
   - "Near-perfect": to the point that most legacy C projects can free-pass the C+ build pipeline.
 - **Improved Productivity**
   - High-level coding techniques can be used for low-level applications.
   - The performance impact of productivity features should be predictable and controllable.
 - **Improved Memory Safety**
   - For the cases where security should be prioritized over small performance overheads.
   - A configurable "turn-key" approach, also benefiting legacy C projects.
  
## Build Pipeline

C+ aims to piggyback on the well-established C build pipeline. To do so, C+ just adds two parts in there: the _sugar_ part and the _salt_ part. The _sugar_ part translates C+ code into the C equivalent (the latest version) so as to reuse the C front-end pipeline. The _salt_ part operates at the IR level to enable features that are tricky to implement at the source code level or that require light-weight static analysis (e.g., data type information).

## Design Principle

 1. No name mangling for non-language-inherent symbols.
 2. Minimum hidden or implicit operations. (If inevitable, tight-scoping of such.)
 3. Clear and simple link between the language specification and its implementation.
 4. Only include the features that are collectively expected to cover large development needs.
 5. (For memory safety) Minimum-to-no amount of legacy code adaptation.
