# Method Call Operator

This feature introduces a new operator called the _method call operator_ that calls a structure's function pointer member with a reference to the structure instance as the 0th argument.

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
 - `<struct_ptr>`: structure pointer (or any expression that has a structure pointer type)\
 - `<func_ptr_member_name>`: function pointer member's name
 - `<arg_list>`: list of arguments for the function pointer

Equivalent to a combination of an indirect member access operator followed by a call operator, whose 0th argument is a structure pointer. (i.e., `<struct_ptr> -> <func_ptr_member_name> ( <struct_ptr> [,<arg_list>] )`)

## Specification

The first parameter to the operator (i.e., `<struct_var>` or `<struct_ptr>`) can be an expression. In this case, the expression is evaluated only once.

Becuase of the property of the method call operator, the called function pointers _should_ specify the containing structure's pointer as the 0th argument.

The method call operator can also call [function members](../func_in_struct.md).
