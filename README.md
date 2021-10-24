# C++ std::function, lambda, functor, function pointer

{% tabs %}
{% tab title="Functors, Lambda, Function Pointer" %}
```cpp
#include <iostream>
#include <string>
#include <vector>

struct MyFunctor {
  explicit MyFunctor(int target) : target(target) {}
  bool operator()(int candidate) const { return candidate == target; }

  int target;
};

struct S {
  int a, b;
};

struct MySFunctor {
  bool operator()(const S& s1, const S& s2) const { return s1.b > s2.b; }
};


int main()
{
    
    std::vector<int> candidates = {1, 2, 7};
    // Functor
    if (std::any_of(candidates.begin(), candidates.end(), MyFunctor(2))) {
        std::cout << "target found\n";
    }
    // Lambda
    if (std::any_of(candidates.begin(), candidates.end(), [](int candidate){ return candidate == 2;})) {
        std::cout << "target found\n";
    }
    
    std::vector<S> v = {{1, 1}, {1, 2}, {2, 1}};
    // Functor
    std::sort(v.begin(), v.end(), MySFunctor());
    for(const auto& s : v) {
        std::cout << s.a << "," << s.b << std::endl;    
    }
    // Lambda
    std::sort(v.begin(), v.end(), [](const S& s1, const S& s2){ return s1.b < s2.b;});
    for(const auto& s : v) {
        std::cout << s.a << "," << s.b << std::endl;    
    }

    // Function pointer is lambda without capture
    int (*square)(int) = [](int x) { return x * x;};
    std::cout << square(10);
}

```


{% endtab %}

{% tab title="std::function demo1" %}
```cpp
// Example of std::function and lambda function.
#include <iostream>
#include <string>
#include <functional>

class Scheduler {
 public:
  void Schedule(std::function<void()> work) const {
    std::cout << "Scheduler::Schedule() Begin\n";
    work();
    std::cout << "Scheduler::Schedule() End\n";
  }
};

class Caller {
 public:
  void Call(const Scheduler& scheduler) {
    std::cout << "Caller::Call() Begin\n";
    bool flag = true;
    scheduler.Schedule([this, flag](){Print(flag);});
    std::cout << "Caller::Call() End\n";
  }
  
  void Print(bool flag) {
    std::cout << "Caller::Print() flag="<<flag<<"\n";
  }
};

int main()
{
    Caller().Call(Scheduler());
}

// Caller::Call() Begin
// Scheduler::Schedule() Begin
// Caller::Print() flag=1
// Scheduler::Schedule() End
// Caller::Call() End
 
```
{% endtab %}

{% tab title="std::function demo2" %}
```cpp
// [=] used in lambda

#include <iostream>
#include <functional>

template <typename T>
std::function<T (T)> makeConverter(T factor, T offset) {
    return [=] (T input) -> T { return (input + offset) * factor; };
}

int main()
{

    auto milesToKm = makeConverter(1.60936, 0.0);
    std::cout << "milesToKm(10)=" << milesToKm(10) << std::endl;
    
    auto feetToMeter = makeConverter(0.3048, 0.0);
    std::cout << "feetToMeter(3)=" << feetToMeter(3) << std::endl;
    
    auto cToF = makeConverter(5.0/9, -32.0);
    std::cout << "cToF(98)=" << cToF(98.0) << std::endl;
}

// milesToKm(10)=16.0936
// feetToMeter(3)=0.9144
// cToF(98)=36.6667

```
{% endtab %}

{% tab title="Lamdba use cases" %}
```cpp
#include <iostream>
#include <string>
#include <vector>
#include <map>
#include <set>

class A {
 public:
    A(int v): value(v) {}
    int GetValue() const { return this->value;}
 private:
    int value;    
};

void Print(const std::vector<A>& elems) {
  for(const auto& elem : elems) {
    std::cout << elem.GetValue() << "";
  }    
  std::cout << std::endl;
}

auto by_score = [](const A& a, const A& b) {
  return a.GetValue() < b.GetValue();
};

void Print2(const std::set<A, decltype(by_score)>& elems) {
  for(const auto& elem : elems) {
    std::cout << elem.GetValue() << "";
  }    
  std::cout << std::endl;
}

int main()
{
  // 1. lambda with empty captures As Template Parameters
  std::vector<A> elems = {A(3), A(1), A(2)};
  Print(elems);
  std::sort(elems.begin(), elems.end(), [](const A& a, const A&b) {
        return a.GetValue() < b.GetValue();
      });
  Print(elems);
  std::cout << std::endl;
  
  // 2. lambda with local variables in captures As Template Parameters
  std::vector<A> elems2 = {A(3), A(1), A(2)};
  std::map<int, int> value2weight= {
    {1, 100}, 
    {2, 300},
    {3, 200}
  };
  Print(elems2);
  std::sort(elems2.begin(), elems2.end(), [&value2weight](const A& a, const A&b) {
        return value2weight[a.GetValue()] < value2weight[b.GetValue()];
      });
  Print(elems2);
  std::cout << std::endl;
  
  // 3. lambda used As Container Functors
  std::set<A, decltype(by_score)> s = {A(3), A(1), A(2)};
  Print2(s);
  
  // 4. Initializing const members which are difficult to do
  class UsersHavingInitial {
   public:
    UsersHavingInitial(char letter, const std::set<std::string>& users)
      : letter_(letter),
        count_([this, &users]() -> int {
          int n = 0;
          for (const auto& u : users)
            if (!u.empty() && u[0] == letter_)
              ++n;
          return n;
        }()) {}
   private:
    const char letter_;
    const int count_;  // can be const despite nontrivial initializer.
 };
 
    // 5. Early Return
    //  void IsRegistered(const K& k) {
    //     #if USING_LAMBDA
    //     const bool registered = [this, &k]() -> bool {
    //         for (const auto& reg : registries_)
    //           for (const auto& elem : reg)
    //             if (elem == k) return true;
    //         return false;
    //       }();
    //     #else  // !USING_LAMBDA
    //       bool registered = false;  // can't be const
    //       for (const auto& reg : registries_) {
    //         if (registered) break;  // error-prone
    //         for (const auto& elem : reg) {
    //           if (elem == k) {
    //             registered = true;
    //             break;  // C++ only has single-level break
    //           }
    //         }
    //       }
    //     #endif  // !USING_LAMBDA
    //       // ...
    //  }

    // 6. Localized Static Initializers
    // std::string Lookup(Id key) {
    //   // The initializer for id_names runs on the first time Lookup is called.
    //   static const auto* const id_names = []() -> std::map<Id, std::string>* {
    //     auto m = new std::map<Id, std::string>();
    //     // complex initialization of '*m'.
    //     return m;
    //   }();
    //   auto iter = id_names->find(key);
    //   if (iter == id_names->end()) {
    //     return "nobody";
    //   }
    //   return iter->second;
    // }
}

```
{% endtab %}
{% endtabs %}
