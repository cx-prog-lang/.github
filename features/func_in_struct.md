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

A function member can be used to denote a strong correlation between a function and a structure instance. In the following example, two structure instances, `john` and `mark`, are initialized with different functions. As a result, two instances print different results even if the structure type is the same.

```c
#include <stdio.h>

struct Dancer {
  void do_flip();
};

void do_back_flip() { printf("boing\n"); }
void do_front_flip() { printf("bwang\n"); }

int main() {
  struct Dancer john = { .do_flip = do_back_flip };
  struct Dancer mark = { .do_flip = do_front_flip };

  john.do_flip();    // Output: boing
  mark.do_flip();    // Output: bwang

  return 0;
}
```

Combined with the [automatic default value](../auto_default.md) feature, a function member can mimic an OOP-style static method of a data type. (Note: this can also be achieved with a regular function pointer member field.)

```c
#include <stdio.h>

struct Test {
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

See [Structure Extension](../struct_ext.md) on how to implement dynamic dispatch (a.k.a., subtype polymorphism) using this.

## Caveat

 - The functions initialized in the default value (as in the second example) are **lost** if a structure instance defines its own explicit initializer. To preserve them, the explicit initializer should specify `default(<data_type>)` at the beginning. For example, the instance `test` will initialize `x` to `300`, while preserving the default value to a function member `identify`.

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

## Discussion
