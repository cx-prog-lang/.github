# Automatic Default Value

This feature enables developers to specify the default value for a data type that the compiler implicitly applies when the type is instantiated. The default value takes lower precedence than explicit variable initializations. The default value is referenceable with `default(<type_name>)` so that explicit initializers can be built based on it. A data type will be left _uninitialized_ as per the legacy C standard if no default value was declared for it.

## Syntax

```c
<type_name> default = <default_value>;
```

Equivalent to declaring a variable named `default` to the same scope as the type `<type_name>`.

## Specification

Among all types in C (https://en.cppreference.com/w/c/language/type.html), the following types may have a default value:

 - Basic types (e.g., int, char, float, ...)
 - Enumerated types
 - Structure types
 - Union types
 - Pointer types
 - Atomic types

For array types, each array element will be initialized to its default value if it exists. Array types or type aliases from `typedef`s **cannot** have their own default values.

For pointer types, the default value of `void *` is a special default value that applies to _all_ pointer types by default. This is overridden if a specific pointer type defines its own default value. Nested pointer types don't share the same default value with un-nested counterparts (e.g., `int **` is treated as a different pointer type than `int *` in terms of a default value).

All default values are compile-time constants. Also, it's illegal to take a reference to the default value.

## Example

### Basic Usage

Suppose the global struct `Test` was declared as below.

```c
struct Test { int a; float b; char c; };
```

Then, declaring a global variable named `default` as below will define the default value of the struct `Test`.

```c
struct Test default = { .a = 1, .c = 'x' };
```

In this example, the default value of the field `b` will be `0` as per the initial value of unspecified fields in C ("empty-initialized", https://en.cppreference.com/w/c/language/struct_initialization.html).

A default value can be defined anywhere after the data type definition (should it be defined), but its declaration should be visible to the code using the corresponding data type. For example, in the following example, the variable `t` will be automatically initialized with the default value, assuming `test_struct.c` and `main.c` are eventually linked together.

```c
// test_struct.h
struct Test { int a; float b; char c; };
extern struct Test default;

// test_struct.c
struct Test default = { .a = 1, .c = 'x' };

// main.c
#include <stdio.h>
int main() {
  struct Test t;    // Automatically initialized.
  printf("%d %c\n", t.a, t.c);    // Output: 1 x
  return 0;
}
```

As another example, developers can specify the default value for an enum or a pointer, as shown below.

```c
// enum.h
enum Animal { A_COW, A_DOG, A_CAT };
static enum Animal default = A_COW;

// ptr.h
static int *default = 0;

// main.c
#include <stdio.h>
#include <enum.h>
#include <ptr.h>

int main() {
  enum Animal animal;              // animal == A_COW
  int *ptr;                        // ptr == NULL
  printf("%d %p\n", animal, ptr);  // Output: 0 (null)
  return 0;
}
```

### Deriving or Referencing Default Values

In the case of record types, having a variable initialization overrides _every field_ of the default value (including the fields not specified in the initializer). If the initializer should be built on top of the default value, the initializer expression `default(<data_type>)` can be specified at the beginning of the initializer braces. For example, the field `a` of the variable `t` will be `999` instead of `1`, but the field `c` will remain as `x` as in the default value.

```c
#include <stdio.h>

struct Test { int a; float b; char c; };
struct Test default = { .a = 1, .c = 'x' };

int main() {
  struct Test t = { default(struct Test), .a = 999 };   // 'a' will be assigned as '999'.
  printf("%d %c\n", t.a, t.c);    // Output: 999 x
  return 0;
}
```

The expression `default(<data_type>)` can also reference the default value itself if it's not used as an initializer expression. In this example, the output will be `1` as `default(struct Test).a` references the default value of the field `a`.

```c
#include <stdio.h>

struct Test { int a; float b; char c; };
struct Test default = { .a = 1, .c = 'x' };

int main() {
  printf("%d\n", default(struct Test).a);    // Output: 1
  return 0;
}
```

To initialize all array elements with the default value of the element type's default value, the initializer expression `default(<data_type>)...` can be specified at the beginning of the initializer braces. In this example, the array `arr` will have `10` for each element except for the 1st element with `20`.

```c
#include <stdio.h>

int default = 10;

int main() {
  int arr[3] = { default(int)..., [1] = 20 };   // arr == { 10, 20, 10 }
  printf("%d %d %d\n", arr[0], arr[1], arr[2]); // Output: 10 20 10
  return 0;
};
```

Notice that the initializer expressions after `default(<data_type>)` or `default(<data_type>)...` should have designators.

## Caveat

 - Heap-allocated objects are **not** initialized upon allocation because, in standard C, the type of an object is not determined at allocation time (https://en.cppreference.com/w/c/language/object.html). Instead, a heap-allocated object should be initialized explicitly using `default(<data_type>)` or compound literals as in the example below.

```c
#include <stdio.h>
#include <stdlib.h>

struct Test { int a; float b; char c; };
struct Test default = { .a = 1, .c = 'x' };

int main() {
  struct Test *t = malloc(sizeof(struct Test));
  *t = default(struct Test);   // Explicit initialization.
  printf("%d\n", t->a);        // Output: 1
  return 0;
}
```

 - The nested type's default value is **not** automatically included in their enclosing type's default value because, if the nested type's default value isn't specified in the enclosing type's default value, it will automatically be empty-initialized as per the C standard (https://en.cppreference.com/w/c/language/struct_initialization.html), and if the enclosing type doesn't define any default value at all, the nested type will also be left uninitialized. Instead, should a nested type be initialized as its default value inside the enclosing type, it should be explicitly initialized using `default(<data_type>)`, as in the example below.

```c
#include <stdio.h>

struct Test { int a; float b; char c; };
struct Test default = { .a = 1, .c = 'x' };

struct Test2 { int d; struct Test t; };
struct Test2 default = { .d = 32, .t = default(struct Test) };

int main() {
  struct Test2 t2;
  printf("%d\n", t2.t.a);    // Output: 1
  return 0;
}
```

## Implementation

 - Status: ongoing

The implementation is divided into two parts. The first part is, at a text level, to replace the variables named `default` with a sufficiently mangled but legitimate C variable marked with the data type, such as `__<type_name>_default__`, to declare the default value variable. The second part is, at an IR level, to insert `memcpy` (or any equivalent instructions) that copies the default value variable after every instantiation if there is no explicit initialization. The second part is performed only if a default value variable exists for the given data type.

## Discussion

### Considered C Design Principle

Variables are left uninitialized by default.

### Legacy C Compatibility


This feature is **transparent** to legacy C code, as declaring a variable named `default` has been invalid in C.
