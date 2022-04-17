---
layout: wiki
title: c++ 八股文
cate1: c++
cate2:
description: c++ 八股文
keywords: c++
type:
link:
---

## 空类大小

在 c 语言中，空结构体大小为0：

```c
struct MyEmpty
{

};

printf("%d\n", sizeof(struct MyEmpty)); // 0
```

在 c++ 中，标准规定一个对象在内存中的地址不应该与其它对象相同（no object shall have the same address in memory as any other variable），因此空结构体和空类的大小不能为0（一般为1，与编译器规则相关），因为如果为0，那么空类数组的每个元素地址将相同，且指针相减的运算将需要特殊处理：

```c++
class MyClass
{

};

MyClass arr[10];
std::cout << &arr[5]-&arr[1] << std::end; // 4 ==》 等价下边：
std::cout << ((char*)&arr[5]-(char*)&arr[1])/sizeof(MyClass) << std::end; // 4  如果MyClass大小为0将需要特殊处理
```

## 强制类型转换运算符

C++类型转换包括隐式类型转换和显式类型转换，隐式类型转换由编译器默认进行转换，显式类型转换除了支持C风格的`(type)expression`表达形式，C++中还有四种类型转换运算符：

- static_cast
- dynamic_cast
- const_cast
- reinterpret_cast

四种类型转换运算符特点：

| 运算符类型        | 运行时检查 |  安全性   |  效率   |        备注         |
| --------------   | ----------| -------  | --------| --------------------|
| static_cast      |  否       | 一般      |   高    | 效果类似C风格显式转换，失败则无法通过编译 |
| dynamic_cast     |  是       |   高      | 一般    | 只能转换指针和引用，指针失败返回空，引用失败抛异常 |
| const_cast       |  -       |   -       |  -    |   用于移除类型的const、volatile和__unaligned属性 |
| reinterpret_cast |  否      |   低      |  -     |  在编译期完成，可以转换任何类型的指针，极不安全，避免使用 |

示例：

```c++

class A { virtual void f() {} };
class B : public A { void f() {} };
class C { virtual void f() {} };

// static_cast 基本类型转换，不安全，需要编写者确定安全性
int a = 0;
double b = static_cast<double>(a);

// static_cast 对象转换，不安全，无法转换时编译会报错，对于指针无法转换时运行时报错
B b;
A a = static_cast<A>(b);  // 向上转换 ok
B b2 = static_cast<B>(a); // 向下转换 error no match conversion for static_cast from 'A' to 'B' 
B* pb;
A* pa = static_cast<A*>(pb);  // 向上转换 ok
B* pb2 = static_cast<B*>(pa); // 向下转换且类型一致，ok
A* pa2 = new A();
B* pb3 = static_cast<B*>(pa2); // 向下转换失败 runtime error ...

// static_cast 转换空指针，不安全
double b = 10.1;
void *p = &b;
double *dp = static_cast<double*>(p);

// dynamic_cast<>只能转换引用和指针类型，运行时检查
B b;
A& a = b;
B& bb = dynamic_cast<B&>(a); // 安全的向下转换，因为a实际指向其派生类b

A a;
A& aa = a;
B& bb = dynamic_cast<B&>(aa); // 不安全向下转换，抛出 std::bad_cast 异常

B* b = new B();
A* a = dynamic_cast<A*>(b);  // 向上转换总是安全的
B* bb = dynamic_cast<B*>(a); // 安全的向下转换
C* c = dynamic_cast<C*>(b);  // 返回空指针

int c = 1;
const int* a = &c;
int* b = const_cast<int*>(a); // b 变为非 const 指针
*b = 2;
std::cout << *a << *b << std::endl; // *a = *b = 2

int c = 1;
const int& a = c;
int& b = const_cast<int&>(a); // b 变为非 const 引用
b = 2;
std::cout << a << b << std::endl; // a = b = 2
```

## 虚基类（virtual base class）作用

用于保证菱形继承情况下基类构造函数只执行一次，菱形继承指类D继承类B和类C，而类B和类C都继承自类A这种情况。虚基类只会创建虚基类的一个对象（只调用一次基类的构造函数）。虚基类的构造函数由继承层次最深的类调用。虚基类只需在继承时在基类前面加上virtual关键字。

```c++
class A { public: A() { std::cout << "A" << std::endl; } };
class B : virtual public A { public: B() {std::cout << "B" << std::endl;} };
class C : virtual public A { public: C() {std::cout << "C" << std::endl;} };
class D : public A { public: D() {std::cout << "D" << std::endl;} };
class E : public B, public C { public: E() {std::cout << "E" << std::endl;} };
class F : public B, public D { public: F() {std::cout << "F" << std::endl;} };

C c; // 打印：AC    C构造时会调用A
E e; // 打印：ABCE  基类A的构造函数只调用一次，由E调用       （BC都是虚继承A，所以这种情况不会调用A构造）
F f; // 打印：ABADF F构造时会调用一次A，D构造时也会调用一次A
```

## explicit 关键字

explicit 关键字用于修饰类的构造函数，被 explicit 修饰的构造函数必须显式调用，及隐式的转换构造函数调用将被抑制，在编译时即报错。

## c++11 lambda 表达式和函数指针的差异

先看 lambda 表达式的形式：

```c++
int b = 3, c = 5;

// [] 不捕获外部变量
auto f = [](int a) -> int { return a; };

// [=] 表示引入外部变量的拷贝(值捕获)
auto f = [=](int a) -> int { return a+b; }; 

// [&] 表示引入外部变量的引用(引用捕获)，由于是引用，修改后外部值会变
auto f = [&](int a) -> int { ++b; return a+b; }; 

// [=] 表示引入外部变量的拷贝(值捕获)，mutable 表名捕获的值可以修改，注意修改的是拷贝值，外部值不会变
auto plus = [=] (int x, int y) mutable -> int { b++; return x + y + b + c; }; 

// [=, &b] b引用捕获，其它都是值捕获
auto f = [=, &b](int a) -> int { return a; };
```

c++ lambda 表达式编译器会将其实现为一个仿函数。仿函数（functor）又称为函数对象（function object），仿函数是一个重载了 operator() 运算符的类，可以行使函数的功能：

```c++
class Func {
   public:
       Func(int a);
       void operator() (int b) const {
           cout<<a_+b<<endl;
       }
   private:
       int a_;
};
Func myFunc(1);
myFunc(2); // output 3
```

不捕获外部变量的情况，lambda 表达式和函数指针等价：

```c++
auto f = [](int a, int b) -> int {return a+b;};
typedef int (*func_ptr)(int, int);
func_ptr p = f; // OK lambda函数负值给函数指针p
p(1,2); // 3

// 此时的lambda表达式编译成：
class LambdaClass {
public:
    int operator() (int a, int b) const {
        return a+b;
    }
};
```

捕获外部变量的情况，lambda 表达式无法赋值给函数指针，因为无法在函数指针内处理外部变量：

```c++
int x = 0, y = 2;
auto f = [=, &x](int a, int b) -> int {++x; return a+b+x+y;};

// 此时的lambda表达式编译成：
class LambdaClass {
public:
    LambdaClass(int& x, int y): x_(x), y_(y) { }
    int operator() (int a, int b) /*捕获引用，这里没有 const*/ {
        ++x_;
        return a+b+x_+y_;
    }
private:
    int& x_;
    int y_;
};
```

## 智能指针

### shared_ptr

shared_ptr 内部维护两个指针，分别是被管理对象的指针和被管理对象的控制块对象的指针，shared_ptr 被管理对象可以在多个 shared_ptr 之间通过拷贝或赋值共享，shared_ptr 内部的控制块对象用于维护保持对象的引用计数，shared_ptr 拷贝或赋值时，引用计数将递增，析构时引用计数减少，当引用计数为0时，被管理对象将被析构。

shared_ptr 内部的引用计数使用了 atomic 原子操作，因此多线程读写 shared_ptr 能够保证引用计数的线程安全性，但是 shared_ptr 只有在多线程读（即通过拷贝或赋值共享）时是线程安全的，如果存在写（即改变 shared_ptr 的指向）则需要加锁。原因是 shared_ptr 的读写涉及保持对象的指针操作和引用计数操作两个步骤，这两个步骤并不是原子操作，如下示例程序，一个线程读 shared_ptr ，另一个线程将 shared_ptr 写指向另一个 shared_ptr，此时可能出现读线程完成了持有对象的指向，尚未完成引用计数增加操作时，写线程将原 shared_ptr 指向另一个 shared_ptr ，原来持有的对象将被析构，读线程的 shared_ptr 将指向无效地址：

```c++
#include <iostream>
#include <momery>
#include <thread>
#include <chrono>

void thr_read(std::shared_ptr<int> p) {
    std::shared_ptr<int> pp = p; // read p 两步操作： 1. pp.ptr = p.ptr; 2. pp.use_count = p.use_count;
    std::this_thread::sleep_for(std::chrono::seconds(1));
}

void thr_write(std::shared_ptr<int> p) {
    std::shared_ptr<int> pp = std::make_shared<int>(5);
    p = pp; // write p 如果在 thr_read 第一步操作完成后，第二步操作前完成写 p 那么 thr_read 中的 pp.ptr 将会无效
    std::this_thread::sleep_for(std::chrono::seconds(1));
}

int main() {
    std::shared_ptr<int> p = std::make_shared<int>(1);
    std::thread t1(thr_read, p);
    std::thread t2(thr_write, p);
    t1.join();
    t2.join();
}

```

64位系统，sizeof(shared_ptr)等价于2个指针的大小，即16。

### unique_ptr

unique_ptr 是独占式的，即完全拥有它所管理对象的所有权，不和其它的对象共享。

内部只需要存储一个被管理对象的指针即可，unique_ptr 禁用了拷贝构造和拷贝赋值构造，仅仅实现了移动构造和移动赋值构造，这也就使得它是独占式的。

## 虚析构函数作用

虚析构函数作用：保证基类指针指向派生类对象时，析构时能够正确调用派生类的析构函数。

```c++
class Base { public: virtual ~Base(){}; };
class Derived: public Base {private: int* x; public: Derived(){x = new int(1);}; ~Derived(){delete x;}};

Base* p = new Derived();
delete p; // 如果 ~Base 没有 virtual 则不会调用 Derived 的析构函数，造成内存泄露
```
