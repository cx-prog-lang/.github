# Function Alias Struct Members

This feature declares a list of function alias members in a structure. Function alias members don't alter the structure's memory layout, as they don't _store_ function pointers like function members do. Instead, function alias members are statically linked to the functions in the default value.

## Syntax

```c
// Inside a struct definition,
switch:
  <func_decl>
  <func_decl>
  ...
```

Equivalent to a list of function declarations (`<func_decl>`) followed by a `switch` label (`switch:`).

## Specification

Similar to [function members](../func_in_struct.md), function alias members are also linked to the functions specified in the default value. Calling a function alias member is the same as calling a function (pointer) member. Function alias members also have _function pointer_ types, not _function_ types.

It's prohibited to change function alias members at runtime.

## Example

