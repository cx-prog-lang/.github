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

A function member can mimic an OOP-style static method of a data type. In the following example, the function member `identify` points to the default value `Test_identify` regardless of the explicit initializer.

```c
#include <stdio.h>

struct Test {
  int val;
  void identify();
};

void Test_identify() { printf("I'm a Test\n"); }

struct Test default = { .identify = Test_identify };

int main() {
  struct Test test1;       // Without an explicit initializer.
  struct Test test2 = {};  // With an explicit initializer.
  test1.identify();        // Output: I'm a Test
  test2.identify();        // Output: I'm a Test
  return 0;
}
```

Combined with the [structure extension](../struct_ext.md) feature, this can implement dynamic dispatch (a.k.a., subtype polymorphism).

## Implementation

 - Status: Ongoing

The implementation first needs to find a function declaration within a struct definition and enclose the identifier with `(*` and `)`. Then, at the IR level, it should insert a store instruction for each function member that copies its default value (after an explicit initializer, if applicable).

## Discussion

### Considered C Design Principle

TODO

### Legacy C Compatibility

 - Level: Transparent

This feature is transparent to legacy C code, as declaring a function within a structure definition wasn't allowed in standard C.
