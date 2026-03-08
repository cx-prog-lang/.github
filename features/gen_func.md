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
