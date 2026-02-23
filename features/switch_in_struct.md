# Function Alias Struct Members

This feature declares a list of function alias members in a structure. Function alias members don't alter the structure's memory layout, as they don't _store_ function pointers like function members do. Instead, function alias members are statically linked to the functions specified in the default value.

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

Function alias members are initialized to the [default value](./auto_default.md) of the direct parent structure. It's prohibited to change function alias members at runtime.

Calling a function alias member is the same as calling a function (pointer) member. Function alias members also have _function pointer_ types, not _function_ types.

Function alias members are associated with a structure _type_, not its instance: an alias's actual function is decided by the _type_ of the structure in the member access or [method call](./method_call.md) operators, not by its memory value.

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

A function alias member should be initialized through its direct parent struct's default value; it's impossible to indirectly initialize it via the default value of the enclosing structure. In the following example, the attempt to initialize `func` through `struct BiggerTest` will generate a compile error because `struct BiggerTest` is not a direct parent struct of `func`.

```c
#include <stdio.h>

struct Test {
switch:
  void func();
};

struct BiggerTest {
  struct Test test;
};

void foo() { printf("foo\n"); }
void bar() { printf("bar\n"); }

struct Test default = { .func = foo };
//struct BiggerTest default = { .test.func = bar };    // Compile error: 'func' cannot be indirectly initialized.

int main() {
  struct BiggerTest bigger_test;
  bigger_test.test.func();        // Output: foo, because the 'func' of 'struct Test' was linked to 'foo'.
  return 0;
}
```

## Caveat

 - Unlike regular member functions, function alias members **cannot** simulate polymorphism in OOP because they are associated with the actual function at compile-time by their type. In the following example, the variable `as_alias_2` would yield `foo`, while `not_alias_2` would yield `bar`.

```c
#include <stdio.h>

void foo() { printf("foo\n"); }
void bar() { printf("bar\n"); }

struct AsAlias1 {
switch:
  void func();
};

struct AsAlias2 {
switch:
  void func();
};

struct NotAlias1 {
  void func();
};

struct NotAlias2 {
  void func();
};

struct AsAlias1 default = { .func = foo };
struct NotAlias1 default = { .func = foo };
struct AsAlias2 default = { .func = bar };
struct NotAlias2 default = { .func = bar };

int main() {
  struct AsAlias2 as_alias_2;
  struct NotAlias2 not_alias_2;

  (*(struct AsAlias1 *)&as_alias_2).func();    // Output: foo, because the struct type in the expression is 'struct AsAlias1'.
  (*(struct NotAlias1 *)&not_alias_2).func();  // Output: bar, because 'func' of 'not_alias_2' is 'bar'.

  return 0;
}
```

 - Function alias members are **not** equivalent to static member functions in other languages (primarily in C++), as function alias members can access the structure instance via an argument. In the example below, the structure variable `c` declares a function alias member `incr`, which modifies `c` via an argument pointer to itself. (See also: [method call operator](./method_call.md))

```c
#include <stdio.h>

struct Counter {
  int i;

switch:
  void incr();
};

void Counter_incl(struct Counter *self) { self->i++; }

struct Counter default = { .i = 0; .incr = Counter_incl; }

int main() {
  struct Counter c;
  c.incr(&c);    // Call 'incr' using a regular call operator.
  c.incr{};      // Call 'incr' using a method call operator.
  printf("%d\n", c.i);    // Output: 2

  return 0;
}
```

 - Unlike function members, function alias members don't require initialization before access, including indirect initialization through the enclosing structure. For example, the following call to `func` is valid.

```c
#include <stdio.h>
#include <stdlib.h>

struct Test {
switch:
  void func();
};

struct BiggerTest {
  struct Test t;
};

void foo() { printf("foo\n"); }

struct Test default = { .func = foo };

int main() {
  struct BiggerTest *bt =
    malloc(sizeof(struct BiggerTest));    // 'bt->t' is uninitialized, but can still call 'func'.
  bt->t.func();                           // Output: foo
  return 0;
}
```
