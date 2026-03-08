# Generic Functions

 - Feature ID: GNFN

This feature enables _generic functions_ that accept _generic types_ undetermined at the time of function definition. The generic functions are _instantiated_ as functions when they're referenced with concrete types in any means (e.g., function calls).

# Syntax

 - Definition

```c
<return_type> <func_name> < <generic_type_list> >( <arg_list> ) {
  <func_body>
};
```

`<generic_type_list>` is a comma-separated list of generic types, which may be referenced like normal data types in `<func_body>`.

 - Instantiation

```c
<func_name> < <concrete_type_list> >
```

`<concrete_type_list>` is a comma-separated list of concrete types, each of which replaces the generic type of the given position.

# Specification

Generic functions are not functions on their own. Instead, they become actual functions when they are _instantiated_. The instantiation happens in the current translation unit, at the closest previous global scope of the instantiation point. The instantiation stops at declarations if generic functions are declared but not defined in the current translation unit.

The instantiated generic functions _preserve_ the storage-class specifier of the generic functions.

The concrete types provided to generic functions can be any valid _global_ data types (or any expressions that are resolved to valid _global_ data types) at the time of instantiation.

Instantiated generic functions have the same type if i) the initial generic function was the same, and ii) all concrete types for each generic type are the same types.

Generic functions can be _explicitly instantiated_ ("_specialized_") by manually declaring or defining generic functions with actual types. In this case, they can reuse the body of the generic definitions by specifying `...` for their bodies. The explicitly instantiated generic functions are declared or defined on the spot, and the generic function's initial storage-class specifier is ignored.

When a generic function is explicitly instantiated, its name is _encoded_ such that the name is the same if the function type is the same. To be specific, the encoded alphanumeric string of the generic type declarator (i.e., the angle brackets and the types inside) is appended to the generic function's name, demarcated with `_`. All following implicit instantiations (e.g., via function calls) are linked to the explicitly instantiated generic functions if they have the same function type.
