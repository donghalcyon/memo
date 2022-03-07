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
