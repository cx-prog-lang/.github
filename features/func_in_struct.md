# Function Type Structure Members

This feature allows a structure type to declare a function member field. It's equivalent to a function **pointer** member, whose [default value](../auto_default.md) is implicitly included in the struct's initializers and compound literals.

## Syntax

```c
// Inside a structure definition...
<return_type> <func_name>(<arg_list>);
```

Equivalent to declaring a function inside a structure definition.

## Specification

A function member is a function pointer whose default value is implied in the containing structure's initializer and compound literal. For this matter, a structure _should_ define a default value if it contains any function members.

From the referencer's point of view, a function member is exactly the same as a function pointer member.

## Example

A function member can mimic an OOP-style static method of a data type. In the following example, the function member `identify` points to the default value `Test_identify` regardless of the existence of explicit initializers.

```c
#include <stdio.h>

struct Test {
  int x;
  void identify();
};

void Test_identify() { printf("I'm a Test\n"); }

struct Test default = { .identify = Test_identify };

int main() {
  struct Test test1;           // Without an explicit initializer.
  struct Test test2 = { 10 };  // With an explicit initializer.
  test1.identify();            // Output: I'm a Test
  test2.identify();            // Output: I'm a Test
  return 0;
}
```

Combined with the [structure extension](../struct_ext.md) feature, this can implement dynamic dispatch (a.k.a., subtype polymorphism).

## Caveat

 - As heap-allocated variables are left uninitialized at allocation time, 

## Implementation

 - Status: Ongoing

The implementation first needs to find a function declaration within a struct definition and enclose the identifier with `(*` and `)`. Then, at the IR level, it should insert a store instruction for each function member that copies its default value (after an explicit initializer, if applicable).

## Discussion

### Considered C Design Principle

TODO

### Legacy C Compatibility

 - Level: Transparent

This feature is transparent to legacy C code, as declaring a function within a structure definition wasn't allowed in standard C.

### Concerns

Ideally, a function member should be translated into a **constant** function pointer to eliminate the possibility of (presumably erratic) runtime manipulation. However, this will preclude a `default(<data_type>)` assignment (e.g., for heap-allocated variables) because it attempts to re-assign a constant member field.
