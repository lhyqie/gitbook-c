# binding

<pre class="language-cpp"><code class="lang-cpp">#include &#x3C;iostream>
#include &#x3C;set>
#include &#x3C;vector>
#include &#x3C;algorithm>
#include &#x3C;functional>

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
  	std::cout &#x3C;&#x3C; "Creating Foo with Bar in a class member function binded with a subclass pointer: foo.a=" &#x3C;&#x3C; foo.a &#x3C;&#x3C; std::endl;
  	return foo;
  }
};

// A free function that creats Foo using Bar.
Foo CreateFoo(Bar bar, int multiplier) {
 Foo foo {.a= bar.b * multiplier};
 std::cout &#x3C;&#x3C; "Creating Foo with Bar in a free function: foo.a=" &#x3C;&#x3C; foo.a &#x3C;&#x3C; std::endl;
 return foo;
}

// A template function that executes a free function pointer
template &#x3C;typename T, typename... FuncArgs, typename... Args>
T MakeFuture(T (*func)(FuncArgs...), Args&#x26;&#x26;... args) {
  return func(std::forward&#x3C;Args>(args)...);
}

// A template function that executes a class member function pointer
template &#x3C;typename T, typename Object, typename... FuncArgs, typename... Args>
T MakeFuture(T (Object::*func)(FuncArgs...) const, const Object* obj, Args&#x26;&#x26;... args) {
  auto f = std::bind_front(func, obj, std::forward&#x3C;Args>(args)...);
  return f();
}

int main() {
<strong>  Bar bar{.b=3};
</strong>  Foo foo1 = MakeFuture(&#x26;CreateFoo, bar, /*multiplier=*/2);
  
  FooFactoryImpl impl;
  auto f = std::bind_front(&#x26;FooFactory::CreateFoo, &#x26;impl, bar);
  f();
  
  // Can not make below working without dynamic_cast&#x3C;>. :(
  Foo foo2 = MakeFuture(&#x26;FooFactory::CreateFoo, dynamic_cast&#x3C;const FooFactory*>(&#x26;impl), bar);
  
  return 0;
}


// Creating Foo with Bar in a free function: foo.a=6
// Creating Foo with Bar in a class member function binded with a subclass pointer: foo.a=3
// Creating Foo with Bar in a class member function binded with a subclass pointer: foo.a=3</code></pre>
