# Function Type Structure Members

This feature allows a structure type to declare a function member field. It's equivalent to a function **pointer** member, whose value is _always_ initialized to the [default value](../auto_default.md).

## Syntax

```c
// Inside a structure definition...
<return_type> <func_name>(<arg_list>);
```

Equivalent to declaring a function inside a structure definition.

## Specification

Since a function member is always initialized to the default value, the containing structure _should_ define a default value if it contains a function member.

From the referencer's point of view, a function member is exactly the same as a function pointer member.

## Example

A function member is the syntactic sugar of a function **pointer** member. In the following example, both `func` and `func_ptr` act like function pointer members that can be assigned and called.

```c
#include <stdio.h>

struct Test {
  void func();
  void (*func_ptr)();
};

void yell() { printf("ha!\n"); }

int main() {
  struct Test test = { .func = yell, .func_ptr = yell };
  test.func();      // Output: ha!
  test.func_ptr();  // Output: ha!
  return 0;
}
```

## Implementation

 - Status: Ongoing

The implementation first needs to find a function declaration within a struct definition and enclose the identifier with `(*` and `)`. Then, at the IR level, it should insert a store instruction for each function member that copies its default value (if applicable, after an explicit initializer).

## Discussion

### Considered C Design Principle

TODO

### Legacy C Compatibility

 - Level: Transparent

This feature is transparent to legacy C code, as declaring a function within a structure definition wasn't allowed in standard C.
