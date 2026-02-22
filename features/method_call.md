# Method Call Operator

This feature introduces a new operator called a _method call operator_ that calls a structure's function pointer member with a reference to the structure instance as the 0th argument.

## Syntax

### Direct method call operator

```c
<struct_var> . <func_ptr_member_name> { [<arg_list>] }
```

 - `<struct_var>`: structure variable (or any expression that has a structure type)
 - `<func_ptr_member_name>`: function pointer member's name
 - `<arg_list>`: list of arguments for the function pointer

Equivalent to a combination of a direct member access operator followed by a call operator, whose 0th argument is a reference to the structure instance. (i.e., `<struct_var> . <func_ptr_member_name> ( &<struct_var> [,<arg_list>] )`)

### Indirect method call operator

```c
<struct_ptr> -> <func_ptr_member_name> { [<arg_list>] }
```
 - `<struct_ptr>`: structure pointer (or any expression that has a structure pointer type)
 - `<func_ptr_member_name>`: function pointer member's name
 - `<arg_list>`: list of arguments for the function pointer

Equivalent to a combination of an indirect member access operator followed by a call operator, whose 0th argument is a structure pointer. (i.e., `<struct_ptr> -> <func_ptr_member_name> ( <struct_ptr> [,<arg_list>] )`)

## Specification

The first parameter to the operator (i.e., `<struct_var>` or `<struct_ptr>`) can be an expression. In this case, the expression is evaluated only once.

Because of the property of the method call operator, the called function pointer members (and their pointee functions) _should_ specify the containing structure's pointer as the 0th argument.

The method call operator can also call [function members](./func_in_struct.md) or [function alias members](./switch_in_struct.md).

## Example

In practice, a method call operator is a shorthand for a member access operator followed by a call operator. The operator acts on any function pointer members, including [function members](./func_in_struct.md), provided that the 0th argument is the structure pointer. In the following example, all three calls invoke the same function `func`. Notice that `func` specifies the 0th argument as a pointer to the structure that's intended to be the member of.

```c
#include <stdio.h>

struct Test {
  void (*func_ptr)(struct Test *self, int p);
  void func_mem(struct Test *self, int p);
};

void func(struct Test *self, int q) {
  printf("[func] %d\n", q);
}

struct Test default = { .func_ptr = func, .func_mem = func };

int main() {
  struct Test test;

  test.func_ptr{1};         // Output: [func] 1
  test.func_mem{2};         // Output: [func] 2
  test.func_ptr(&test, 3);  // Output: [func] 3

  return 0;
}
```

The 0th argument of a method call operator can be used to reference other members of the same structure instance, allowing a structure to mimic an OOP-style object. In this example, both function members `up` and `down` operate on the structure's data member `count`.

```c
#include <stdio.h>

struct Counter {
  int count;
  void up(struct Counter *self);
  void down(struct Counter *self);
};

void Counter_up(struct Counter *self) { self->count++; }
void Counter_down(struct Counter *self) { self->count--; }

struct Counter default = {
  .count = 0,
  .up = Counter_up,
  .down = Counter_down
};

int main() {
  struct Counter counter;
  printf("%d\n", counter.count);   // Output: 0

  counter.up{}
  printf("%d\n", counter.count);   // Output: 1

  counter.down{}
  printf("%d\n", counter.count);   // Output: 0

  return 0;
}
```

# Discussion

## Expected Benefits to Legacy C

 - **B3. Canonicalization**: provides a canonical way to use the functions intended to act on a structure.
 - **B4. Code Clarity**: removes verbosity in the combination of a member access and a call.
