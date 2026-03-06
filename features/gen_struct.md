# Generic Structures

 - Feature ID: GNST

This feature allows for _generic structures_ that accept _generic types_ that are undetermined at the time of structure definition. The generic structures are _instantiated_ as actual structure types when they're referenced with concrete types in any methods (e.g., variable declaration).

# Syntax

 - Definition

```c
struct <struct_name> < <generic_type_list> > {
  <struct_body>
};
```

`<generic_type_list>` is a comma-separated list of generic types, which may be referenced like ordinary data types in `<struct_body>`.

 - Instantiation

```c
struct <struct_name> < <concrete_type_list> >
```

`<concrete_type_list>` is a comma-separated list of concrete types, each of which replaces the generic type of the given position.

# Specification

Generic structures are not data types on their own. Instead, they become actual data types when they are _instantiated_. In line with this, a generic struct cannot be used as a data type of global variables (**except the [default value](./auto_default.md)**), function return types, or function argument types (**except the [cleanup function](./obj_dtor.md)**).

The concrete type provided to generic structures can be any valid data type at the time of instantiation.

Instantiated generic structures are the same type if i) the initial generic structure was the same, and ii) all concrete types for each generic type are the same types.

# Caveat

 - It's valid to instantiate generic structures with the generic types of the enclosing scope. For example, the following example instantiates the generic structure `struct Test<T>` with the generic type `U` from the generic function `foo<U>` and the generic type `V` from the generic function `bar<V>`.

```c
struct Test<T> { T field };

struct Test<U> foo<U>() { return (struct Test<U>){ .field = 0 }; }
V bar<V>(struct Test<V> in) { return in.v + in.v; }
```
