# C++ return pointer or reference of local variable

```cpp
// Example program
#include <iostream>
#include <string>

std::string GetByValue() {
  std::string foo = "bar";
  std::cout << "address of foo: " << &foo << "\n"; 
  return foo;
}

std::string* GetByPointer() {
  std::string foo = "bar";
  std::cout << "address of foo: " << &foo << "\n"; 
  return &foo;
}

int main()
{
  // Read Temporary object lifetime
  // https://en.cppreference.com/w/cpp/language/lifetime
  // * The lifetime of a temporary object may be extended 
  // * by binding to a const lvalue reference or to an rvalue 
  // * reference (since C++11).
  
  // Also read Return value optimization
  // https://en.wikipedia.org/wiki/Copy_elision.
  const std::string& const_ref = GetByValue();
  std::cout << "address of const_ref: " << &const_ref << "\n"; 
  std::cout << "const_ref=" << const_ref << "\n";
  
  std::string* pointer = GetByPointer();
  // std::cout << "address of pointer: " << pointer << "\n";
  std::cout << "pointer=" << *pointer << "\n";
}

// address of foo: 0x7fffd13337a8                                                                                                                                                                                                                  
// address of const_ref: 0x7fffd13337a8                                                                                                                                                                                                            
// const_ref=bar                                                                                                                                                                                                                                   
// address of foo: 0x7fffd1333778                                                                                                                                                                                                                  
// address of pointer: 0                                                                                                                                                                                                                           
// Segmentation fault (core dumped)   
```

