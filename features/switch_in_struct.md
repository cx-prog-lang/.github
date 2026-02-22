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

Similar to [function members](../func_in_struct.md), function alias members are also initialized to the [default value](../auto_default.md). It's prohibited to change function alias members at runtime.

Calling a function alias member is the same as calling a function (pointer) member. Function alias members also have _function pointer_ types, not _function_ types.

Function alias members are associated with a structure _type_, not its instance: an alias's actual function is decided by the _type_ of the structure in the member access or [method call](../method_call.md) operators, not by its memory value.

## Example

The usage of function alias members is similar to that of function members. The following example declares `identify` as a function alias member instead of a function member:

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
  test.identify();    // Output: I'm a Test
  return 0;
}
```

## Caveat

 - Function alias members **cannot** simulate OOP-style virtual functions, while regular function members could. In the following example,

```c
#include <stdio.h>

void foo() { printf("foo\n"); }
void bar() { printf("bar\n"); }


```

 - Function alias members are **not** equivalent to static member functions in other languages (primarily in C++), as in function alias members can also access the structure instance via an argument. In the example below, the structure variable `c` declares a function alias member `incr`, which modifies `c` via an argument pointer to itself. (See also: [method call operator](../method_call.md))

```c
#include <stdio.h>

struct Counter {
  int i;

switch:
  void incr();
}

void Counter_incl(struct Counter *self) { self->i++; }

struct Counter default = { .i = 0; .incr = Counter_incl; }

int main() {
  struct Counter c;
  c.incr(&c);    // Call 'incr' using a regular call operator.
  c.incr{};      // Call 'incr' using a method call operator.
  printf("%d\n", c.i);

  return 0;
}
```
