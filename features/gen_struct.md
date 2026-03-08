# Generic Structures

 - Feature ID: GNST

This feature allows for _generic structures_ that accept _generic types_ that are undetermined at the time of structure definition. The generic structures are _instantiated_ as structure types when they're referenced with concrete types in any means (e.g., variable declaration).

# Syntax

 - Definition

```c
struct <struct_name> < <generic_type_list> > {
  <struct_body>
};
```

`<generic_type_list>` is a comma-separated list of generic types, which may be referenced like normal data types in `<struct_body>`.

 - Instantiation

```c
struct <struct_name> < <concrete_type_list> >
```

`<concrete_type_list>` is a comma-separated list of concrete types, each of which replaces the generic type of the given position.

# Specification

Generic structures are not data types on their own. Instead, they become actual data types when they are _instantiated_. In line with this, a generic struct cannot be used as a data type of global variables (**except the [default value](./auto_default.md)**), function return types, or function argument types (**except the [cleanup function](./obj_dtor.md)**).

The concrete types provided to generic structures can be any valid data types or any expressions that are resolved to valid data types at the time of instantiation.

Instantiated generic structures are the same type if i) the initial generic structure was the same, and ii) all concrete types for each generic type are the same types.

When generic structures are instantiated, their [default values](./auto_default.md) and [cleanup functions](./obj_dtor.md) are also instantiated with given concrete types.

# Example

Generic structures are very similar to C++ template structures, except that they lack a separate template variable declaration. In the following example, the generic structure `Test1` has one generic type `T`, whereas `Test2` has two generic types `T` and `U`. The generic types are treated as normal data types in the structure body.

```c
struct Test1<T> { T field; };
struct Test2<T, U> { T field1; U field2; };
```

Generic structures are _instantiated_ when they are referenced with concrete types in any way. All instantiated generic structures are of the same type if they are from the same generic structure, and their concrete types are the same as well. For example, all the following references of `Test` instantiate the same structure type `Test<int>`.

```c
struct Test<T> { T field; };

typedef my_int int;

int foo() {
  struct Test<int> test;                             // struct Test<int> = struct Test<int>
  test = (struct Test<my_int>){ .field = 32 };       // struct Test<my_int> = struct Test<int>
  return ((struct Test<typeof((int)0)>){}).field;    // struct Test<typeof((int)0)> = struct Test<int>
}

struct Test<typeof(*(int*)0)> bar() {                // struct Test<typeof(*(int*)0)> = struct Test<int>
  int dummy;
  return (struct Test<typeof(dummy)>){};             // struct Test<typeof(dummy)> = struct Test<int>
}
```

A generic structure can also have its [default value](./auto_default.md) and the [cleanup function](./obj_dtor.md). In this case, they are also instantiated together with the generic structure when the generic structure is instantiated. The default value and the cleanup function can further be defined for specific actual types, as in the example below.

```c
#include <stdio.h>

struct Test<T> { T field; };

// 'default' and 'break' when the actual type is 'int'.
struct Test<int> default = { .field = 32 };
void break(struct Test<int> prev) { printf("int break\n"); }

// 'default' and 'break' for any other actual type.
struct Test<T> default = { .field = 0 };
void break(struct Test<T> prev) { printf("generic break\n"); }
```

# Caveat

 - It's valid to instantiate generic structures with the generic types of the enclosing scope. For example, the following example instantiates the generic structure `struct Test<T>` with the generic type `U` from the generic function `foo<U>` and the generic type `V` from the generic function `bar<V>`.

```c
struct Test<T> { T field };

struct Test<U> foo<U>() { return (struct Test<U>){ .field = 0 }; }
V bar<V>(struct Test<V> in) { return in.v + in.v; }
```
