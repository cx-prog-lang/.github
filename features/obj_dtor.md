# Canonical Object Value Destructor

This feature introduces a reserved function name, `break`, for the destructor functions for the object _value_. The destructor may be defined as a function named `break` with its target data type as the only argument, and can be called by `break(<obj>)`. By default, the destructors are no-op if undefined.

## Caveat

 - The object value destructor is **not** called automatically at the end of the object's storage duration.
