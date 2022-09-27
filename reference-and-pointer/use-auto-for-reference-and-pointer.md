# Use auto for Reference and Pointer

```cpp
int x = 42;
int& y = x;
auto z = y;  // z is 'int', not 'int&', huh?
int* p1 = &x;
auto p2 = p1;   // p2 is 'int*'
auto* p3 = p1;  // p3 is 'int*' (same as bare auto)
```

The reason is that `auto`s type deduction works the same as the type deduction for T in the following invented function template.

```cpp
template <typename T> void F(T t);
F(x); // T = int
F(y); // T = int
F(p1); // T = int*
```

If we want our `auto`-declared variables to be constant or references we should explicitly declare them as such.

### Rules of Thumb <a href="#rules-of-thumb" id="rules-of-thumb"></a>

* Assume a bare `auto` will make a copy
* Always specify `auto&` or `const auto&` unless a copy is desired
* Prefer `const auto&` over `auto&` unless you need to modify the underlying value
* Prefer `auto*` or `const auto*` to make pointers explicit in the code
