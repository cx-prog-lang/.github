# Canonical Object Value Destructor

This feature introduces a reserved function name, `break`, for the object _value_ destructor functions. The destructor may be defined as a function named `break` with its target data type as the only argument, and can be called by `break(<obj>)`. By default, the destructors are no-op if undefined.

## Syntax

```c
void break(<data_type> [<arg_name>]) {
   ...
}
```

## Example

The canonicalization of object value destructors provides a generic way to semantically clean up objects. As a prime example, a composite type can clean up its members without knowing their specific destructor functions (or even their existence).

## Caveat

 - The object value destructor is **not** called automatically at the end of the object's storage duration.
