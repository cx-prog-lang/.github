# Method Call Operator

This feature introduces a new operator called the _method call operator_ that calls a structure's function pointer member with a reference to the structure instance as the 0th argument.

## Syntax

 - Direct method call operator
```c
<struct_var> . <func_ptr_member_name> { [<arg_list>] }
```
where `<struct_var>` is a structure variable (or any expression that has a structure type), `<func_ptr_member_name>` is a function pointer member's name, and `<arg_list>` is a list of arguments for the function pointer.

Equivalent to a combination of a direct member access operator followed by a call operator, whose 0th argument is a reference to the structure instance. (i.e., `<struct_var> . <func_ptr_member_name> ( &<struct_var> [,<arg_list>] )`)

 - Indirect method call operator
```c
<struct_ptr> -> <func_ptr_member_name> { [<arg_list>] }
```
where `<struct_ptr>` is a structure pointer (or any expression that has a structure pointer type), `<func_ptr_member_name>` is a function pointer member's name, and `<arg_list>` is a list of arguments for the function pointer.

Equivalent to a combination of an indirect member access operator followed by a call operator, whose 0th argument is a structure pointer. (i.e., `<struct_ptr> -> <func_ptr_member_name> ( <struct_ptr> [,<arg_list>] )`)

## Specification

The method call operator is a ternary operator that accepts three 

Parallel to the member access operator (`.` and `->`), the method call operator also comes in two flavors: the direct one (`. ... { }`) and the indirect one (`-> ... { }`).

Both eventually call the function pointer member with the provided argument list, but the direct one works with a structure variable  inserts a reference to the containing structure instance before the argument list. The indirect one, on the other hand, 
