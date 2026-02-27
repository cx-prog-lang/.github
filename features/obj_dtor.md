# Canonical Object Value Cleanup Function

 - Feature ID: F005

This feature suggests a canonical function for the object _value_ cleanup per data type (a.k.a. _destructors_). The cleanup functions are optionally defined as a function named `break` with its target data type as the only argument, and are called by `break(<obj>)`. By default, undefined cleanup functions are no-op.

## Syntax

```c
void break(<data_type> [<arg_name>]) {
   ...
}
```

## Example

The canonicalization of object value destructors provides a generic way to semantically clean up objects. As a prime example, a composite type can clean up its members without knowing their existence. This example shows a reference counter structure for `struct Elem` type objects (`RCElem`), which automatically cleans up the allocated `struct Elem` object when it should be deallocated. (For clarity, the reference counting logic was omitted.)

```c
#include <stdlib.h>

struct RCElem {
  struct Elem *o;
  int *_refcount;
};

struct RCElem RCElem_create() {
  return (struct RCElem){ .o = malloc(sizeof(struct Elem)), ._refcount = 1 };
}

// ...

void break(struct RCElem prev) {
  if (!prev.o) return;
  if (*prev._refcount == 0) {
    break(*prev.o);
    free(prev.o);
    // ...
  }
}
```

In the above example, `break(*prev.o)` will be linked to the cleanup function for `struct Elem` (i.e., `void break(struct Elem)`) if it exists, or a no-op function if it doesn't. 

Or, if `RCElem` is to be implemented in an entirely decoupled way with object allocation (i.e., the users of `RCElem` are responsible for the (de)allocation of `struct Elem`), `break(*prev.o); free(prev.o);` could be replaced with `break(prev.o)`. Then, it will be linked to the cleanup function for `struct Elem *` (i.e., `void break(struct Elem *)`).

```c
struct RCElem {
  struct Elem *o;
  int *_refcount;
};

struct RCElem RCElem_create(struct Elem *init) {
  return (struct RCElem){ .o = init, ._refcount = 1 };
}

// ...

void break(struct RCElem prev) {
  if (!prev.o) return;
  if (*prev._refcount == 0) {
    break(prev.o);
    // ...
  }
}
```

Then, the users of `RCElem` can implement the cleanup function for `struct Elem *` to internally deallocate the pointer.

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

 - The cleanup function is a convention and is **not** called automatically in any way (e.g., at the end of the object's storage duration).
 - For a composite type, the member's cleanup function is **not** automatically called recursively.
