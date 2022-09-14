# binding

```cpp
#include <iostream>
#include <set>
#include <vector>
#include <algorithm>
#include <functional>

struct Foo {
  int a;
};

struct Bar {
  int b;
};

class FooFactory {
 public:
  virtual Foo CreateFoo(Bar bar) const = 0;
};

class FooFactoryImpl : public FooFactory {
 public:
  // a class member function that creats Foo using Bar.
  Foo CreateFoo(Bar bar) const override {
  	Foo foo {.a= bar.b};
  	std::cout << "Creating Foo with Bar in a class member function binded with a subclass pointer: foo.a=" << foo.a << std::endl;
  	return foo;
  }
};

// A free function that creats Foo using Bar.
Foo CreateFoo(Bar bar, int multiplier) {
 Foo foo {.a= bar.b * multiplier};
 std::cout << "Creating Foo with Bar in a free function: foo.a=" << foo.a << std::endl;
 return foo;
}

// A template function that executes a free function pointer
template <typename T, typename... FuncArgs, typename... Args>
T MakeFuture(T (*func)(FuncArgs...), Args&&... args) {
  return func(std::forward<Args>(args)...);
}


template <typename ... As1, typename ... As2>
void execute(void(*fun)(As1...), As2 ... args) {
    fun(args...);
}

template <typename T, typename Object, typename... FuncArgs, typename... Args>
T MakeFutureTest(T (Object::*func)(FuncArgs...) const, const Object* obj, Args&&... args) {
  auto f = std::bind_front(func, obj, std::forward<Args>(args)...);
  return f();
}

int main()
{
	
	Bar bar{.b=3};
	Foo foo1 = MakeFuture(&CreateFoo, bar, /*multiplier=*/2);
	
	FooFactoryImpl impl;
	auto f = std::bind_front(&FooFactory::CreateFoo, &impl, bar);
	f();
	
	// not working!
	Foo foo2 = MakeFutureTest(&FooFactory::CreateFoo, dynamic_cast<const FooFactory*>(&impl), bar);
	
	return 0;
}


// Creating Foo with Bar in a free function: foo.a=6
// Creating Foo with Bar in a class member function binded with a subclass pointer: foo.a=3
// Creating Foo with Bar in a class member function binded with a subclass pointer: foo.a=3
```
