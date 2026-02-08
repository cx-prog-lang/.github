# Function Type Structure Members

This feature allows a structure type to declare a function type member field. It's automatically converted into a function **pointer** type member field of the same name.

## Syntax

```c
// Inside a structure definition...
<return_type> <func_name>(<arg_list>);
```

Equivalent to declaring a function inside a structure definition.

## Specification

A function member is treated in the exact same way as a function pointer member.

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

## Caveat

 - The function members initialized in the [default value](../auto_default.md) are **lost** if a structure instance defines its own explicit initializer. To preserve them, the explicit initializer should specify `default(<data_type>)` at the beginning. For example, the instance `test` will initialize `x` to `300`, while preserving the default value to a function member `identify`.

```c
#include <stdio.h>

struct Test {
  int x;
  void identify();
};

void Test_identify() { printf("I'm a Test\n"); }

struct Test default = { .identify = Test_identify };

int main() {
  struct Test test = { default(struct Test), .x = 300 };
  test.identify();          // Output: I'm a Test
  printf("%d\n", test.x);   // Output: 300
  return 0;
}
```

## Implementation

 - Status: Ongoing

The implementation simply needs to find a function declaration inside a struct definition and enclose the identifier with `(*` and `)`.

## Discussion

### Considered C Design Principle

TODO

### Legacy C Compatibility

 - Level: Transparent

This feature is transparent to legacy C code, as declaring a function within a structure definition wasn't allowed in standard C.

### Concern

 - It can be confusing (and possibly a source of runtime errors) that an explicit initialization of a struct variable can nullify the default value for function members. A potential mitigation would be to make the function members special and be initialized even in the absence of `default(<data_type>)`, but this can go against the language philosophy of C ("implicit initialization").
