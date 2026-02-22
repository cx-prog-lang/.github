# Function Alias Struct Members

This feature declares a list of function alias members in a structure. Function alias members don't alter the structure's memory layout, as they don't _store_ function pointers like function members do. Instead, function alias members are statically linked to the functions in the default value.

In technical terms, function alias members are equivalent to so-called _static_ function members of a structure.

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

Similar to [function members](../func_in_struct.md), function alias members are also initialized to the [default value](../auto_default.md). It's prohibited to change function alias members at runtime.

Calling a function alias member is the same as calling a function (pointer) member. Function alias members also have _function pointer_ types, not _function_ types.

Function alias members are associated with a structure _type_, not its instance: an alias's actual function is decided by the _type_ of the structure in the member access or [method call](../method_call.md) operators, not by the memory value of the structure.

## Example

```c
#include <stdio.h>

struct Test {
  int x;

switch:
  void identify();
};

void Test_identify() { printf("I'm a Test\n"); }

struct Test default = { .identify = Test_identify };

int main() {
  struct Test test;
  test1.identify();            // Output: I'm a Test
  test2.identify();            // Output: I'm a Test
  return 0;
}
```
