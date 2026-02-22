# Function Structure Members

This feature allows a structure type to declare a function member field. It's equivalent to a function **pointer** member, whose [default value](./auto_default.md) is implicitly included in the struct's explicit initializers and compound literals.

## Syntax

```c
// Inside a structure definition...
<return_type> <func_name>(<arg_list>);
```

Equivalent to declaring a function inside a structure definition.

## Specification

A function member is a function pointer whose default value is implied in the containing structure's explicit initializer and compound literal. For this matter, a structure _should_ define a default value if it contains any function members. The function member's initial value specified in the explicit initializer will be ignored.

From the referencer's point of view, a function member is exactly the same as a function pointer member: function members have _function pointer_ types, not _function_ types.

## Example

A function member can mimic an OOP-style method of a data type. In the following example, the function member `identify` points to the default value `Test_identify` regardless of explicit initializers.

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

## Caveat

 - As heap-allocated objects are left uninitialized after allocation (irrespective of the existence of default values), function members are also left **uninitialized** in them. To initialize function members, heap-allocated objects should be explicitly initialized with `default(<data_type>)` or any compound literals of the corresponding struct type. In the following example, the function member `identify` of the variables `test1`, `test2`, and `test3` is correctly initialized, while that of `test4` is not.

```c
#include <stdio.h>
#include <stdlib.h>

struct Test {
  int x;
  void identify();
};

void Test_identify() { printf("I'm a Test\n"); }

struct Test default = { .identify = Test_identify };

int main() {
  struct Test *test1 = malloc(sizeof(struct Test));
  struct Test *test2 = malloc(sizeof(struct Test));
  struct Test *test3 = malloc(sizeof(struct Test));
  struct Test *test4 = malloc(sizeof(struct Test));

  *test1 = default(struct Test);
  *test2 = (struct Test){};
  *test3 = (struct Test){ .x = 30 };
  // No explicit initialization for 'test4'.

  test1.identify();    // Output: I'm a Test
  test2.identify();    // Output: I'm a Test
  test3.identify();    // Output: I'm a Test
  test4.identify();    // Undefined behavior
  return 0;
}
```

 - As the members of a structure, function members are also taken into account in the explicit positional initialization. The following example initializes `test.x` as `10` and `test.y` as `20`; the `0` in the position of `test.identify` will be ignored. Recommendation: Use designated initializers if there are function members.

```c
#include <stdio.h>

struct Test {
  int x;
  void identify();
  int y;
};

void Test_identify() { printf("I'm a Test\n"); }

struct Test default = { .identify = Test_identify };

int main() {
  struct Test test = { 10, 0, 20 };   // '0' for 'identify' will be ignored.
  printf("%d %d\n", test.x, test.y);  // Output: 10 20
  test.identify();                    // Output: I'm a Test
  return 0;
}
```

## Implementation

 - Status: Ongoing

The implementation first needs to find a function declaration within a struct definition and enclose the identifier with `(*` and `)`. Then, at the IR level, it should insert the store instructions to copy function members' default values after explicit initializers and compound literal assignments.

## Discussion

### Expected Benefits to Legacy C

 - **B1. Code Resiliency/Safety**: prevents function misuse by clearly indicating the function's relevance to a structure.
 - **B2. Development Productivity**: facilitates indexing the functions intended to be used with a structure.
 - **B3. Canonicalization**: provides a language-suggested way to indicate the function's relevance to a structure.
 - **B4. Code Clarity**: eliminates the need for separate documentation of a function's relevance to a structure.

### Legacy C Compatibility

 - Level: Transparent

This feature is transparent to legacy C code, as declaring a function within a structure definition wasn't allowed in standard C.

### Concerns

Ideally, a function member should be translated into a **constant** function pointer to eliminate the possibility of (presumably erratic) runtime manipulation. However, this will prevent initialization of heap-allocated objects (e.g., by assigning `default(<data_type>)`) because it will attempt to reassign a constant member field on the way.
