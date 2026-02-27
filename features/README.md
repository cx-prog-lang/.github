# Cx Features

This directory introduces the features of the Cx languages built on top of the standard C language.

 - [Object Default Value](./auto_default.md): define a custom default value per data type.
 - [Function Structure Members](./func_in_struct.md): declare a function type member field in a structure.
 - [Function Alias Structure Members](./switch_in_struct.md): declare function aliases in a structure that don't alter the structure memory layout.
 - [Method Call Operator](./method_call.md): call a structure member with a reference to the structure instance as the 0th argument.
 - [Internal Structure Members](./internal_member.md): indicate the structure members that function (alias) members are only expected to access directly.
 - [Canonical Object Value Cleanup Function](./obj_dtor.md): define a canonical function to clean up an object value per data type.
 - [Automatic Expression](./auto_expr.md): denote an expression to be executed when the block is exited, instead of now.
 - __Type Generic Macro__: pass data types as the arguments to a data type or a function.

Below are the features under consideration.

 - __Pointer Bound Checking Compiler Option__: enable pointer bound checking for every pointer.
 - __Pointer Sanitization Compiler Option__: sanitize every pointer to `0` after memory deallocation.

## Feature Dependency

Each feature page includes a feature dependency diagram that shows the relationships between features. The relationship falls into three categories:

 - **Required**: a feature requires another feature to be complete. (Solid line)
 - **Affected**: a feature can be affected by another feature in implementation or in design. (Dotted line)
 - **None**: the features are independent in implementation and in design. (Not described)

The diagram only describes direct relationships; indirect relationships are applied transitively, with priority given to stronger dependencies (Required > Affected > None).
