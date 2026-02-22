# Cx Features

This directory introduces the features of the Cx languages built on top of the standard C language.

 - [Object Default Value](./auto_default.md): define a custom default value per data type.
 - [Function Structure Members](./func_in_struct.md): declare a function type member field in a structure.
 - [Function Alias Structure Members](./switch_in_struct.md): declare function aliases in a structure that don't alter the structure memory layout.
 - [Internal Structure Members](./internal_member.md): indicate the structure members that function (alias) members are only expected to access directly.
 - [Method Call Operator](./method_call.md): call a structure member with a reference to the structure instance as the 0th argument.
 - [Canonical Object Destructor](./obj_dtor.md): define a canonical function to destroy an object per data type.
 - [Automatic Expression](./auto_expr.md): denote an expression to be executed when the block is exited, instead of now.
 - [Type Generic Macro](./type_gen.md): pass data types as the arguments to a data type or a function.

Below are the features under consideration.

 - [Pointer Bound Checking Compiler Option](./ptr_boundck.md): enable pointer bound checking for every pointer.
 - [Pointer Sanitization Compiler Option](./ptr_sanitize.md): sanitize every pointer to `0` after memory deallocation.
