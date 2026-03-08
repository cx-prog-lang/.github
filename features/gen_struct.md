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

Generic structures are not data types on their own. Instead, they become actual data types when they are _instantiated_. For this reason, a generic struct cannot be used as a data type of global variables (**except the [default value](./auto_default.md)**), function return types, or function argument types (**except the [cleanup function](./obj_dtor.md)**).

The default values and the cleanup functions for generic structures are instantiated when the generic structures themselves are instantiated.

The concrete types provided to generic structures can be any valid data types or any expressions that are resolved to valid data types at the time of instantiation.

Instantiated generic structures are the same type if i) the initial generic structure was the same, and ii) all concrete types for each generic type are the same types.

Generic structs _cannot_ be nested, although they can contain _instantiated_ generic structures inside.

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

A generic structure can also have its [default value](./auto_default.md) or the [cleanup function](./obj_dtor.md), which are instantiated together when the generic structure is instantiated. They can further be defined for specific data types, in which case they override the default value or the cleanup function declared with generic types. In the following example, `Test< <type> >` will use the first default value and the cleanup function if `<type>` is `int`; otherwise, it will use the second default value and the cleanup function.

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

 - It's **valid** to instantiate generic structures with the generic types of the enclosing scope. For example, the following example instantiates the generic structure `struct Test<T>` with the generic type `U` from the generic function `foo<U>`. (See: [generic functions](./gen_func.md))

```c
struct Test<T> { T field };

struct Test<U> foo<U>() { return (struct Test<U>){ .field = 0 }; }
```

 - It's **invalid** to declare generic types again when the enclosing scope already declared generic types. This can be avoided by making the enclosing scope declare _all_ generic types used inside. In the bad example, `TestInner<U>` generates a compile error as the enclosing scope (`TestOuter<T>`) already declared a generic type. On the other hand, the good example declares all generic types in the enclosing scope (`TestOuter<T, U>`).

```c
// Bad example
struct TestOuter<T> {       // 'TestOuter' declares a generic type (T).
  T field1;
  struct TestInner<U> {     // Compile error: the enclosing scope already declared a generic type (T).
    U field2;
  } in;
};

// Good example
struct TestOuter<T, U> {    // 'TestOuter' declares all generic types inside (T and U).
  T field1;
  struct TestInner {        // 'TestInner' uses one of the generic types (U).
    U field2;
  } in;
};
```

 - The default value and the cleanup function **can** reference generic types inside their bodies. In this case, the generic types are declared with the object data type (for the default value) or the argument data type (for the cleanup function). In the following example, the default value and the cleanup function reference the generic type `T` within their bodies.

```c
#include <stdio.h>

struct Test<T> { T field };

struct Test<T> default = {
  .field = default(T)
};

void break(struct Test<T> prev) {
  break(prev.field);
}
```
