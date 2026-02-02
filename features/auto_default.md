# Automatic Default Value

This feature enables developers to specify the default value for a data type that the compiler implicitly applies when the type is instantiated. The default value takes lower precedence than explicit variable initializations. The default value is referenceable with `default(<type_name>)` so that explicit initializers can be built based on it. A data type will be left _uninitialized_ as per the legacy C standard if no default value was declared for it.

## Syntax

```c
<type_name> default = <default_value>;
```

Equivalent to declaring a global variable named `default` as `<type_name>`.

## Example

Suppose the struct `Test` was declared as below.

```c
struct Test { int a; float b; char c; };
```

Then, declaring a global variable named `default` in a global scope as below will define the default value of the struct `Test`.

```c
struct Test default = { .a = 1, .c = 'x' };
```

In this example, the default value of the field `b` will be `0` as per the initial value of any global variable in C.

A default value can be defined anywhere after the definition of the data type (should it be defined, such as structs), but in order to mark the data type to have a default value,
its declaration should be visible to the code using the data type. For example, in the following example, the variable `t` will be automatically initialized with the default value, assuming `test_struct.c` and `main.c` are eventually linked together.

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

In the case of record types, having a variable initialization overrides _every field_ of the default value (including the fields not specified in the initializer). If the initializer should be built on top of the default value, an expression `default(<data_type>)` should be specified at the beginning of the initializer. For example, the field `a` of the variable `t` will be `999` instead of `1`, but the field `c` will remain as `x` as in the default value. Notice that if the initializer contains any `default(<data_type>)` (except when it's used inside an expression of a member initializer), every regular member initializer should have a designator.

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

Outside the initialization braces, the expression `default(<data_type>)` can also be used as the identifier of the default value. In this example, the output will be `1` as `default(struct Test).a` references the default value of the field `a`.

```c
#include <stdio.h>

struct Test { int a; float b; char c; };
struct Test default = { .a = 1, .c = 'x' };

int main() {
  printf("%d\n", default(struct Test).a);    // Output: 1
  return 0;
}
```

## Caveat

 - Heap-allocated objects are **not** initialized upon allocation because, in standard C, the type of an object is not determined at allocation time (https://en.cppreference.com/w/c/language/object.html). Instead, a heap-allocated object should be initialized explicitly using `default(<type_name>)` as in the example below.

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

 - The nested type's default value is **not** automatically included in their enclosing type's default value. If the enclosing type's default value doesn't specify the nested type's default value, it's automatically initialized as `0` as per the global variable initialization in C. If the enclosing type doesn't define any default value, the entire enclosing type is left uninitialized. In case the enclosing type should have a default value for the nested type, its default value should specify this as in the example below.

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

The implementation is divided into two parts. The first part is, at a text level, to replace the global variables named `default` with a sufficiently mangled but legitimate C global variable marked with the data type, such as `__<type_name>_default__`, to declare the default value variable. The second part is, at an IR level, to insert `memcpy` that copies the default value variable between every instantiation of the data type and its explicit initialization. The second part is performed only if a default value variable exists for the given data type.

## Discussion

### Considered C Design Principle

 - Variables are left uninitialized by default.
    - If the default value was not declared for a data type, its variable will be left uninitialized.
 - Language semantics should be clearly linked to the low-level implementation.
    - There is a one-to-one correspondence between the default value and a memory copy per variable.
 - Every operation may be as explicit as possible from the developer's point of view.
    - The default value is recognized as existing only if the code using the data type can see its declaration.
    - A variable initialization will be done only _once_ either with the default value or with the explicit initializer (which can explicitly be based on the default value).

### Legacy C Compatibility

This feature is transparent to legacy C code, as declaring a global variable named `default` has been invalid in C.
