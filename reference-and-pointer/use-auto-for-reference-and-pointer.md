# Use auto for Reference and Pointer

```cpp
int x = 42;
int& y = x;
auto z = y;  // z is 'int', not 'int&', huh?
int* p1 = &x;
auto p2 = p1;   // p2 is 'int*'
auto* p3 = p1;  // p3 is 'int*' (same as bare auto)
```
