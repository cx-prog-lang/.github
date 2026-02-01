# Automatic Default Value

This feature enables developers to specify the _automatic_ default value of data types. When declaring a variable, the default value will be implicitly applied by the compiler. (If it's present, the explicit initialization takes precedence.) In the case of record types (i.e., struct and union), the explicit initialization is overridden on top of the default value on a per-field basis. A data type will be left _uninitialized_ as per the legacy C standard if no default value was declared for it.

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
struct Test default;

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

In the case of record types, a variable initialization overrides the default value on a per-field basis. For example, the field `a` of the variable `t` will be `999` instead of `1`, but the field `c` will remain as `x` as in the default value.

```c
#include <stdio.h>
struct Test { int a; float b; char c; };
struct Test default = { .a = 1, .c = 'x' };

int main() {
  struct Test t = { .a = 999 };   // 'a' will be assigned as '999'.
  printf("%d %c\n", t.a, t.c);    // Output: 999 x
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

### Legacy C Compatibility

This feature is transparent to legacy C code, as declaring a global variable named `default` has been invalid in C.
