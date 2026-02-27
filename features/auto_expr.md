# Automatic Expression

 - Feature ID: F006

This feature defers the evaluation of an expression until the end of the current scope. The evaluation timing is the same as the deallocation of (non-VLA) automatic objects, hence the name "_automatic_".

## Syntax

```c
auto <expr>
```
