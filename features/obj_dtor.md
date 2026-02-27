# Canonical Object Value Cleanup Function

 - Feature ID: F005

This feature suggests a canonical function for the object _value_ cleanup per data type (a.k.a. _destructors_). The cleanup functions can be optionally defined as a function named `break` with its target data type as the only argument, and can be called by `break(<obj>)`. By default, undefined cleanup functions are no-op.

## Syntax

```c
void break(<data_type> [<arg_name>]) {
   ...
}
```

## Example

The canonicalization of object value destructors provides a generic way to semantically clean up objects. As a prime example, a composite type can clean up its members without knowing their existence. This example shows a reference counter structure for `int` type objects (`RCInt`), which automatically cleans up the allocated `int` object when it's deallocated. (For clarity, the object allocation and reference counting logics were omitted.)

```c
#include <stdlib.h>

struct RCInt {
  int *o;
  int *_refcount;
};

// ...

void break(struct RCInt prev) {
  if (!prev.o) return;
  if (*prev._refcount == 0) {
    break(prev.o);
    free(prev.o);
    // ...
  }
}
```

In the above example, `break(prev.o)` will be linked to the cleanup function for `int *` (i.e., `void break(int *)`) if it exists, or a no-op function if it doesn't. 

The canonical object value cleanup function also allows for cleaning up child members of a structure without hard-coding the specific name of the cleanup functions for each member. In this example, the structure `Context` cleans up the members `data`, `record`, and `cache` with the same canonical name `break`.

```c
struct Context {
  struct Data data;
  struct Record record;
  struct Cache cache;
};

void break(struct Context prev) {
  break(prev.data);
  break(prev.record);
  break(prev.cache);
}
```

## Caveat

 - The cleanup function is **not** called automatically at the end of the object's storage duration.
 - For a composite type, the member's cleanup function is **not** automatically called recursively.
