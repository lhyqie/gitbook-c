---
layout: landing
---

# absl::optional vs std::unique\_ptr

{% embed url="https://abseil.io/tips/123" %}



Originally posted as totw/123 on 2016-09-06

By Alexey Sokolov [(sokolov@google.com)](mailto:sokolov@google.com) and Etienne Dechamps [(edechamps@google.com)](mailto:edechamps@google.com)

## How to Store Values <a href="#how-to-store-values" id="how-to-store-values"></a>

This tip discusses several ways of storing values. Here we use class member variables as an example, but many of the points below also apply to local variables.

```
#include <memory>
#include "absl/types/optional.h"
#include ".../bar.h"

class Foo {
  ...
 private:
  Bar val_;
  absl::optional<Bar> opt_;
  std::unique_ptr<Bar> ptr_;
};
```

### As a Bare Object <a href="#as-a-bare-object" id="as-a-bare-object"></a>

This is the simplest way. `val_` is constructed and destroyed at the beginning of `Foo`’s constructor and at the end of `Foo`’s destructor, respectively. If `Bar` has a default constructor, it doesn’t even need to be initialized explicitly.

`val_` is very safe to use, because its value can’t be null. This removes a class of potential bugs.

But bare objects are not very flexible:

* The lifetime of `val_` is fundamentally tied to the lifetime of its parent `Foo` object, which is sometimes not desirable. If `Bar` supports move or swap operations, the contents of `val_` can be replaced using these operations, while any existing pointers or references to `val_` continue pointing or referring to the same `val_` object (as a container), not to the value stored in it.
* Any arguments that need to be passed to `Bar`’s constructor need to be computed inside the initializer list of `Foo`’s constructor, which can be difficult if complicated expressions are involved.

### As `absl::optional<Bar>` <a href="#as-absloptionalbar" id="as-absloptionalbar"></a>

This is a good middle ground between the simplicity of bare objects and the flexibility of `std::unique_ptr`. The object is stored inside `Foo` but, unlike bare objects, `absl::optional` can be empty. It can be populated at any time by assignment (`opt_ = ...`) or by constructing the object in place (`opt_.emplace(...)`).

Because the object is stored inline, the usual caveats about allocating large objects on the stack apply, just like for a bare object. Also be aware that an empty `absl::optional` uses as much memory as a populated one.

Compared to a bare object, `absl::optional` has a few downsides:

* It’s less obvious for the reader where object construction and destruction occur.
* There is a risk of accessing an object which does not exist.

### As `std::unique_ptr<Bar>` <a href="#as-stdunique_ptrbar" id="as-stdunique_ptrbar"></a>

This is the most flexible way. The object is stored outside of `Foo`. Just like `absl::optional`, a `std::unique_ptr` can be empty. However, unlike `absl::optional`, it is possible to transfer ownership of the object to something else (through a move operation), to take ownership of the object from something else (at construction or through assignment), or to assume ownership of a raw pointer (at construction or through `ptr_ = absl::WrapUnique(...)`, see [TotW 126](https://abseil.io/tips/126).

When `std::unique_ptr` is null, it doesn’t have the object allocated, and consumes only the size of a pointer[1](https://abseil.io/tips/123#fn:deleter).

Wrapping an object in a `std::unique_ptr` is necessary if the object may need to outlive the scope of the `std::unique_ptr` (ownership transfer).

This flexibility comes with some costs:

* Increased cognitive load on the reader:
  * It’s less obvious what’s stored inside (`Bar`, or something derived from `Bar`). However, it may also decrease the cognitive load, as the reader can focus only on the base interface held by the pointer.
  * It’s even less obvious than with `absl::optional` where object construction and destruction occur, because ownership of the object can be transferred.
* As with `absl::optional`, there is a risk of accessing an object which does not exist - the famous null pointer dereference.
* The pointer introduces an additional level of indirection, which requires a heap allocation, and is [not friendly](https://en.wikipedia.org/wiki/Locality\_of\_reference) to CPU caches; Whether this matters or not depends a lot on particular use cases.
* `std::unique_ptr<Bar>` is not copyable even if `Bar` is. This also prevents `Foo` from being copyable.

## Conclusion <a href="#conclusion" id="conclusion"></a>

As always, strive to avoid unnecessary complexity, and use the simplest thing that works. Prefer bare object, if it works for your case. Otherwise, try `absl::optional`. As a last resort, use `std::unique_ptr`.

|                                 | `Bar`                   | `absl::optional<Bar>`                                                  | `std::unique_ptr<Bar>`                                           |
| ------------------------------- | ----------------------- | ---------------------------------------------------------------------- | ---------------------------------------------------------------- |
| Supports delayed construction   |                         | ✓                                                                      | ✓                                                                |
| Always safe to access           | ✓                       |                                                                        |                                                                  |
| Can transfer ownership of `Bar` |                         |                                                                        | ✓                                                                |
| Can store subclasses of `Bar`   |                         |                                                                        | ✓                                                                |
| Movable                         | If `Bar` is movable     | If `Bar` is movable                                                    | ✓                                                                |
| Copyable                        | If `Bar` is copyable    | If `Bar` is copyable                                                   |                                                                  |
| Friendly to CPU caches          | ✓                       | ✓                                                                      |                                                                  |
| No heap allocation overhead     | ✓                       | ✓                                                                      |                                                                  |
| Memory usage                    | `sizeof(Bar)`           | `sizeof(Bar) + sizeof(bool)`[2](https://abseil.io/tips/123#fn:padding) | `sizeof(Bar*)` when null, `sizeof(Bar*) + sizeof(Bar)` otherwise |
| Object lifetime                 | Same as enclosing scope | Restricted to enclosing scope                                          | Unrestricted                                                     |
| Call `f(Bar*)`                  | `f(&val_)`              | `f(&opt_.value())` or `f(&*opt_)`                                      | `f(ptr_.get())` or `f(&*ptr_)`                                   |
| Remove value                    | N/A                     | `opt_.reset();` or `opt_ = absl::nullopt;`                             | `ptr_.reset();` or `ptr_ = nullptr;`                             |

1. In case of a non-empty custom deleter there is also an additional space for that deleter. [↩](https://abseil.io/tips/123#fnref:deleter)
2. Also padding may be added. [↩](https://abseil.io/tips/123#fnref:padding)
