# Automatic Expression

 - Feature ID: F006

This feature defers the evaluation of an expression until the end of the current scope. The evaluation timing is the same as the deallocation of (non-VLA) automatic objects, hence the name "_automatic_".

## Syntax

```c
auto <expr>
```

## Example

An automatic expression is similar to [GCC's cleanup attribute](https://gcc.gnu.org/onlinedocs/gcc/Common-Variable-Attributes.html#index-cleanup-variable-attribute) and [Go's `defer`](https://go.dev/blog/defer-panic-and-recover), the prime usage being an automatic cleanup call for an object. In this example, a file pointer is automatically closed at the end of the current scope.

```c
#include <stdio.h>

int main() {
  FILE *f;

  f = fopen("path", "r");
  if (!f) return 0;
  auto fclose(f);

  // ...

  return 0;  // 'fclose(f)' will be evaluated here.
}
```

An automatic expression can be paired with the [canonical object value cleanup function](./obj_dtor.md). In this example, the cleanup function for `FILE *` only closes the file when it's not null, so the automatic expression can be placed immediately after the declaration of `FILE *`.

```c
#include <stdio.h>

void break(FILE *prev) {
  if (prev) fclose(prev);
}

int main() {
  FILE *f;
  auto break(f);

  f = fopen("path", "r");

  if (!f) { /* ... */ }
  else { /* ... */ }

  return 0;  // 'break(f)' will be evaluated here.
}
```

## Caveat

 - The automatic expression is evaluated at the end of _its_ scope, not other scopes related to the expression inside. For example, the following example may cause a premature cleanup of the object `f`.

```c
#include <stdio.h>

int main() {
  FILE *f = fopen("path", "r");
  if (f) {
    auto fclose(f);
  }  // 'fclose(f)' is evaluated here.

  // Warning: 'f' is already closed at this point.

  return 0;
}
```
