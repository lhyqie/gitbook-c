# lvalue vs rvalue

```cpp
#include <iostream>
#include <string>
#include <vector>


void f() {}

const int& f2() {
    int x = 0;
    return x;    
}

int f3() {
    int x = 1;
    return x;
}


class A {
 public:
    A(int size) : size_(size) {
        data_ = new int[size];
    }
    A(){}
    A(const A& a) {
        size_ = a.size_;
        data_ = new int[size_];
        std::cout << "calling copy: A(const A& a)" << std::endl;
    }
    A(A&& a) {
        this->data_ = a.data_;
        a.data_ = nullptr;
        std::cout << "calling move: A(A&& a) " << std::endl;
    }
    ~A() {
        if (data_ != nullptr) {
            delete[] data_;
        }
    }
 private:
    int *data_;
    int size_;
};

void PrintV(int &t) {
    std::cout << "lvalue ";
}

void PrintV(int &&t) {
    std::cout << "rvalue ";
}

template<typename T>
void Test(T &&t) {
    PrintV(t);
    PrintV(std::forward<T>(t));
    PrintV(std::move(t));
    std::cout << std::endl;
}

int main()
{

    // 左值一般有：
    // 函数名和变量名
    // 返回左值引用的函数调用
    // 前置自增自减表达式++i、--i
    // 由赋值表达式或赋值运算符连接的表达式(a=b, a += b等)
    // 解引用表达式*p
    // 字符串字面值"abcd"

    int i=0, j=1;
    std::cout << "&i=" << &i << std::endl;                       // ok, i is lvalue
    std::cout << "&f=" << &f << std::endl;                       // ok, f is lvalue
    printf("f address= %p\n", &f);
    std::cout << "&f2()=" << &f2() << std::endl;                 // ok, f2() is lvalue because it returns by reference     
    std::cout << "&i=" << &(++i) << std::endl;                   // ok, ++i is lvalue
    std::cout << "&(i=j)= " << &(i=j) << std::endl;              // ok, i=j is lvalue
    int* pi =&i;
    std::cout << "&(*pi)= " << &(*pi) << std::endl;              // ok, *pi is lvalue
    std::cout << "&(abcd)= " << &("abcd") << std::endl;          // ok, string literal is lvalue
    
    // 纯右值和将亡值都属于右值。

    // 纯右值
    // 运算表达式产生的临时变量、不和对象关联的原始字面量、非引用返回的临时变量、lambda表达式等都是纯右值。
    // 举例：
    // 除字符串字面值外的字面值
    // 返回非引用类型的函数调用
    // 后置自增自减表达式i++、i--
    // 算术表达式(a+b, a*b, a&&b, a==b等)
    // 取地址表达式等(&a)
    
    // std::cout << "&1.0= "<< &(1.0) << std::endl;              // error, 1.0 is rvalue
    // std::cout << "&f3()=" << &f3() << std::endl;              // error, f3() is rvalue because it returns by value
    // std::cout << "i= "<< i << " &i=" << &(i++) << std::endl;  // error, i++ is rvalue
    // std::cout << "&(i+j)= " << &(i+j) << std::endl;           // error, i+j is rvalue
    // std::cout << "&(&i)= " << &(&i) << std::endl;             // error, &(&i) is rvalue
    
    // 将亡值
    // 将亡值是指C++11新增的和右值引用相关的表达式，通常指将要被移动的对象、T&&函数的返回值、std::move函数的返回值、转换为T&&类型转换函数的返回值，
    // 将亡值可以理解为即将要销毁的值，通过“盗取”其它变量内存空间方式获取的值，在确保其它变量不再被使用或者即将被销毁时，
    // 可以避免内存空间的释放和分配，延长变量值的生命周期，常用来完成移动构造或者移动赋值的特殊任务。
    // A a;
    // auto c = std::move(a); // c是将亡值
    // auto d = static_cast<A&&>(a); // d是将亡值
    // std::cout << c.GetValue();    
    // int a = 100;
    // std::cout << "a=" << a << " a address=" << &a << std::endl;
    // int &b = a;  
    // std::cout << "a=" << a << " a address=" << &a <<std::endl;
    // std::cout << "b=" << b << " b address=" << &b <<std::endl;
    // // int&&c = a; // error, a is lvalue.
    // int &&c = std::move(a);
    // std::cout << "a=" << a << " a address=" << &a <<std::endl;
    // std::cout << "c=" << c << " c address=" << &c <<std::endl;
    A a1(10);
    A a2 = a1;  // calling copy: A(const A& a)
    A a3 = std::move(a1); // calling move: A(A&& a) 
    
    
    // 分析 
    // Test(1)：1是右值，模板中T &&t这种为万能引用，右值1传到Test函数中变成了右值引用，但是调用PrintV()时候，
    // t变成了左值，因为它变成了一个拥有名字的变量，所以打印lvalue，而PrintV(std::forward<T>(t))时
    // 会进行完美转发，按照原来的类型转发，所以打印rvalue，PrintV(std::move(t))毫无疑问会打印rvalue。
    // Test(a)：a是左值，模板中T &&这种为万能引用，左值a传到Test函数中变成了左值引用，所以有代码中打印。
    // Test(std::forward<T>(a))：转发为左值还是右值，依赖于T，T是左值那就转发为左值，T是右值那就转发为右值。
    
    Test(1); // lvalue rvalue rvalue
    int a = 1;
    Test(v); // lvalue lvalue rvalue
    Test(std::forward<int>(a)); // lvalue rvalue rvalue
    Test(std::forward<int&>(a)); // lvalue lvalue rvalue
    Test(std::forward<int&&>(a)); // lvalue rvalue rvalue
    
    return 0;
}

```

