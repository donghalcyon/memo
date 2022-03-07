## 2.Template Basics
### 2.1 Template Declaration and Definition
在 C++ 中，我们一共可以声明（declare） 5种不同的模板，分别是：类模板（class template）、函数模板（function template）、变量模板（variable template）、别名模板（alias template）、和概念（concept）。

```cpp
// declarations
template <typename T> struct  class_tmpl;
template <typename T> void    function_tmpl(T);
template <typename T> T       variable_tmpl;          // since c++14
template <typename T> using   alias_tmpl = T;         // since c++11
template <typename T> concept no_constraint = true;   // since c++20
```
其中，前三种模板都可以拥有定义（definition），而后两种模板不需要提供定义，因为它们不产生运行时的实体.

```
// definitions
template <typename T> struct  class_tmpl {};
template <typename T> void    function_tmpl(T) {}
template <typename T> T       variable_tmpl = T(3.14);
```
template 关键字表明这是一个模板，尖括号中声明了模板的参数。

### 2.2 Template Parameters and Arguments
在模板中，我们可以声明三种类型的 形参（Parameters），分别是：非类型模板形参（Non-type Template Parameters）、类型模板形参（Type Template Parameters）和模板模板形参（Template Template Parameters）：
```
// There are 3 kinds of template parameters:
template <int n>                               struct NontypeTemplateParameter {};
template <typename T>                          struct TypeTemplateParameter {};
template <template <typename T> typename Tmpl> struct TemplateTemplateParameter {};
```
要注意的是，非类型模板实参必须是常量，因为模板是在编译期被展开的，在这个阶段只有常量，没有变量。
```
template <float &f>
void foo() { std::cout << f << std::endl; }

template <int i>
void bar() { std::cout << i << std::endl; }

int main() {
  static float f1 = 0.1f;
  float f2 = 0.2f;
  foo<f1>();  // output: 0.1
  foo<f2>();  // error: no matching function for call to 'foo', invalid explicitly-specified argument.

  int i1 = 1;
  int const i2 = 2;
  bar<i1>();  // error: no matching function for call to 'bar',
              // the value of 'i' is not usable in a constant expression.
  bar<i2>();  // output: 2
}
```

对于模板模板形参（Template Template Parameters），和类模板的声明类似，**也是在类型的前面加上 template <...>。** **模板模板形参只接受类模板或类的别名模板作为实参**，并且实参模板的形参列表必须要与形参模板的形参列表匹配。
```cpp
template <template <typename T> typename Tmpl>
struct S {};

template <typename T>             void foo() {}
template <typename T>             struct Bar1 {};
template <typename T, typename U> struct Bar2 {};

S<foo>();   // error: template argument for template template parameter
            // must be a class template or type alias template
S<Bar1>();  // ok
S<Bar2>();  // error: template template argument has different template
            // parameters than its corresponding template template parameter
```

模板的模板参数中的参数也可以有默认实参
```cpp
template<template<typename T,
  typename A = MyAllocator> class Container>
class Adaptation {
  Container<int> storage; // Container<int, MyAllocator>
  ...
};
```

模板的模板参数的参数名称只能被自身其他参数的声明使用
```cpp
template<template<typename T, T*> class Buf>  // OK
class Lexer {
  static T* storage;  // 错误：模板的模板参数不能用在此处
  ...
};
```

通常模板的模板参数的名称不会在后面被用到，所以一般可以省略不写
```cpp
template<template<typename, // 省略T
  typename = MyAllocator> class Container>
class Adaptation {
  Container<int> storage; // Container<int, MyAllocator>
  ...
};
```






一个模板可以声明多个形参，更一般地，可以声明一个变长的形参列表，称为 "template parameter pack"，这个变长形参列表可以接受 0 个或多个非类型常量、类型、或模板作为模板实参。变长形参列表必须出现在所有模板形参的最后。
```
// template with two parameters
template <typename T, typename U> struct TemplateWithTwoParameters {};

// variadic template, "Args" is called template parameter pack
template <int... Args>                            struct VariadicTemplate1 {};
template <int, typename... Args>                  struct VariadicTemplate2 {};
template <template <typename T> typename... Args> struct VariadicTemplate3 {};

VariadicTemplate1<1, 2, 3>();
VariadicTemplate2<1, int>();
VariadicTemplate3<>();
```

模板可以声明默认实参，与函数的默认实参类似。只有 **主模板（Primary Template）** 才可以声明默认实参，模板特化（Template Specialization）不可以。

### 2.3 Template Instantiation

模板的实例化（Instantiation）是指由泛型的模板定义生成具体的类型、函数、和变量的过程。模板在实例化时，模板形参被替换（Substitute）为实参，从而生成具体的实例。
模板的实例化分为两种：按需（或隐式）实例化（on-demand (or implicit) instantiation） 和 显示实例化（explicit instantiation），其中隐式的实例化是我们平时最常用的实例化方式。隐式实例化，或者说按需实例化，是当我们要用这个模板生成实体的时候，要创建具体对象的时候，才做的实例化。而显式实例化是告诉编译器，你帮我去实例化一个模板，但我现在还不用它创建对象，将来再用。
**要注意，隐式实例化和显式实例化并不是根据是否隐式传参而区分的。**
*自 C++11 后，新标准还支持了显式的实例化声明（explicit instantiation declaration），我们会在后面的 Advanced Topics - Explicit Instantiation Declarations 中介绍这一特性。*
```
// t.hpp
template <typename T> void foo(T t) { std::cout << t << std::endl; }

// t.cpp
// on-demand (implicit) instantiation
#include "t.hpp"
foo<int>(1);
foo(2);
std::function<void(int)> func = &foo(int);

// explicit instantiation
#include "t.hpp"
template void foo<int>(int);
template void foo<>(int);
template void foo(int);
```

当我们在代码中使用了一个模板，触发了一个实例化过程时，编译器就会用模板的实参（Arguments）去替换（Substitute）模板的形参（Parameters），生成对应的代码。
同时，编译器会根据一定规则选择一个位置，将生成的代码插入到这个位置中，**这个位置被称为 POI（point of instantiation）**。
由于要做替换才能生成具体的代码，因此 C++ 要求模板的定义对它的 POI 一定要是可见的。换句话说，在同一个翻译单元（Translation Unit）中，编译器一定要能看到模板的定义，才能对其进行替换，完成实例化。因此最常见的做法是，我们会将模板定义在头文件中，然后再源文件中 #include 头文件来获取该模板的定义。这就是模板编程中的包含模型（Inclusion Model）。

### 2.4 Template Arguments Deduction
为了实例化一个模板，编译器需要知道所有的模板实参，但不是每个实参都要显式地指定。有时，编译器可以根据函数调用的实参来推断模板的实参，这一过程被称为模板实参推导（Template Arguments Deduction)。对每一个函数实参，编译器都尝试去推导对应的模板实参，如果所有的模板实参都能被推导出来，且推导结果不产生冲突，那么模板实参推导成功。举个例子：为了实例化一个模板，编译器需要知道所有的模板实参，但不是每个实参都要显式地指定。有时，编译器可以根据函数调用的实参来推断模板的实参，这一过程被称为模板实参推导（Template Arguments Deduction)。对每一个函数实参，编译器都尝试去推导对应的模板实参，如果所有的模板实参都能被推导出来，且推导结果不产生冲突，那么模板实参推导成功。举个例子：
(```)
template <typename T>
void foo(T, T) {}

foo(1, 1);      // #1, deduced T = int, equivalent to foo<int>
foo(1, 1.0);    // #2, deduction failed.
                // with 1st arg, deduced T = int
                // with 2nd arg, deduced T = double
(```)
C++17 引入了类模板实参推导（Class Template Arguments Deduction），可以通过类模板的构造函数来推导模板实参：
```
template <typename T>
struct S { S(T, int) {} };

S s(1, 2);     // deduced T = int, equivalent to S<int>
```

### 2.5 Template Specialization
模板的特化（Template Specialization）允许我们替换一部分或全部的形参，并定义一个对应改替换的模板实现。其中，替换全部形参的特化称为全特化（Full Specialization），替换部分形参的特化称为偏特化（Partial Specialization），非特化的原始模板称为主模板（Primary Template）。**只有类模板和变量模板可以进行偏特化，函数模板只能全特化**。
```
// function template
template <typename T, typename U> void foo(T, U)       {}     // primary template
template <>                       void foo(int, float) {}     // full specialization

// class template
template <typename T, typename U> struct S             {};    // #1, primary template
template <typename T>             struct S<int, T>     {};    // #2, partial specialization
template <>                       struct S<int, float> {};    // #3, full specialization

S<int, int>();      // choose #2
S<int, float>();    // choose #3
S<float, int>();    // choose #1
```

我们可以只声明一个特化，然后在其他的地方定义它：
```
template <> void foo<float, int>;
template <typename T> struct S<float, T>;
```
这里你可能已经注意到了，特化声明与显式实例化（explicit instantiation）的语法非常相似，注意不要混淆了。
```
// Don't mix the syntax of "full specialization declaration" up with "explict instantiation"
template    void foo<int, int>;   // this is an explict instantiation
template <> void foo<int, int>;   // this is a full specialization declaration
```
**除了语法外，二者的含义也很容易混淆。理论上来说，模板实例化的结果就是一个该模板的全特化，因为它就是一个用确定实参替换了全部形参的模板实现。所以有的书和文档中也会用特化（Specialization）这个词来指代模板实例化之后生成的那个实体（类型、函数、或变量）。为了区分，我们称这种为隐式特化（Implicit Specialization），称我们在本节中讨论的特化机制为显式特化（Explicit Specialization）。很多的书和文档中是不做这种区分的，所以可能会产生误解，需要读者结合上下文去理解 Specialization 指的是什么。本文中我们避免使用特化一词来指代实例化的结果，而改用“实例”或“实体”，特化一词专指模板的特化机制。**

### 2.6 Function Template Overloading
**函数模板虽然不能偏特化，但是可以重载（Overloading）**，并且可以与普通的函数一起重载。在 C++ 中，所有的函数和函数模板，只要它们拥有不同的签名（Signature），就可以在程序中共存。一个函数（模板）的签名包含下面的部分：

函数（模板）的非限定名（Unqualified Name）
这个名字的域（Scope）
成员函数（模板）的 CV 限定符(register static extern thread_local mutable)
成员函数（模板）的 引用限定符
函数（模板）的形参列表类型，如果是模板，则取决于实例化前的形参列表类型
函数模板的返回值类型
函数模板的模板形参列表
所以，根据这个规则，下列的所有函数和函数模板foo，都被认为是重载，而非重定义：



## 3. Learn TMP in Use (Part I)
### 3.1 Example 1: Type Manipulation
#### 3.1.1 is_reference
```
template <typename T> struct is_reference      { static constexpr bool value = false; };    // #1 primary template
template <typename T> struct is_reference<T&>  { static constexpr bool value = true; };     // #2 partial specializeation
template <typename T> struct is_reference<T&&> { static constexpr bool value = true; };     // #3 partial specializeation

std::cout << is_reference<int>::value << std::endl;    // 0
std::cout << is_reference<int&>::value << std::endl;   // 1
std::cout << is_reference<int&&>::value << std::endl;  // 1
```
#### 3.1.2 remove_reference
```
template <typename T> struct remove_reference      { using type = T; };     // #1
template <typename T> struct remove_reference<T&>  { using type = T; };     // #2
template <typename T> struct remove_reference<T&&> { using type = T; };     // #3

// case 1:
int&& i = 0;
remove_reference<decltype(i)>::type j = i;    // equivalent to: int j = i;

// case 2:
template <typename T>
void foo(typename remove_reference<T>::type a_copy) { a_copy += 1; }

foo<int>(i);    // passed by value
foo<int&&>(i);  // passed by value
```
remove_reference<T>::type 是一个待决名（Dependent Name），编译器在语法分析的时候还不知道这个名字到底代表什么。对于普通的名字，编译器直接通过名字查找就能知道这个名字的词性。**但对于待决名，因为它是什么取决于模板的实参 T**，所以直到编译器在语义分析阶段对模板进行了实例化之后，它才能对“type”进行名字查找，知道它到底是什么东西，所以名字查找是分两个阶段的，待决名直到第二个阶段才能被查找。**但是在语法分析阶段，编译器就需要判定这个语句是否合法，所以需要我们显式地告诉编译器 “type” 是什么**。在 remove_reference<T>::type 这个语法中，type 有三种可能，一是静态成员变量或函数，二是一个类型，三是一个成员模板。**编译器要求对于类型要用 typename 关键字修饰，对于模板要用 template 关键字修饰，以便其完成语法分析的工作**。


### 3.2 Metafunction Convention
#### 3.2.1 Metafunction always return a "type"
我们称这种在编译期“调用”的特殊“函数”为 Metafunction，它代表了 TMP 中的“逻辑”。Metafunction 接受常量和类型作为参数，返回常量或类型作为结果，我们称这些常量和类型为Metadata，它代表了 TMP 中的“数据”。进一步地，我们称常量为 Non-type Metadata (or Numerical Metadata)，称类型为 Type Metadata。

#### 3.2.2 integral_constant
在真实世界的场景中，一个典型的 Non-type Metadata 是这样定义的：
```
template <typename T, T v>
struct integral_constant {
  static constexpr T value = v;
  using value_type = T;
  using type = integral_constant;   // using injected-class-name
  constexpr operator value_type() const noexcept { return value; } //到value_type的隐式类型转换
  constexpr value_type operator()() const noexcept { return value; } //函数调用运算符重载
};
```
这些成员，特别是 type，都会使 TMP 变得更方便:
```
// alias
template <bool B> using bool_constant = integral_constant<bool, B>;
using true_type  = bool_constant<true>;
using false_type = bool_constant<false>;

template <typename T> struct is_reference      { using type = false_type; };
template <typename T> struct is_reference<T&>  { using type = true_type; };
template <typename T> struct is_reference<T&&> { using type = true_type; };

std::cout << is_reference<int>::type::value;  // 0
std::cout << is_reference<int>::type();       // 0, implicit cast: false_type --> bool
std::cout << is_reference<int>::type()();     // 0
```




C++17中引入了**inline变量**，变量（包括静态数据成员）和变量模板能被内联，这意味着它们的定义能跨编译单元。对于总能定义在多个编译单元中的变量模板来说，这是多余的。但类内定义的静态数据成员不会像成员函数一样内联，因此就要指定inline关键字





