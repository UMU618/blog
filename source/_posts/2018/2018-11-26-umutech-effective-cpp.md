---
layout: post
title: 《Effective C++》读书笔记
date: 2018-11-26 23:22:15
categories: UMUTech
tags:
- cpp
---
## 第一章 让自己习惯 C++

### 1. 视 C++ 为一个语言联邦

C++ 是个多重范型编程语言：面向过程、面向对象、函数式、泛型、原编程式，所以他的规约很多，记住四个次语言可以帮助了解 C++：C、Object-Oriented C++、Template C++、STL。

### 2. 尽量以 const、enum、inline 替换 \#define

他们的根本差别是：前三者是编译器处理的，最后者是预处理器处理的。enum 比 const 更像 #define，比如说 const 定义通常可以求地址或引用，而 enum 不行。

inline 函数比宏多了类型安全和可预料性，一个例子是将 i++ 或 ++i 当参数传给宏时，可能导致 ++ 了多次，而传给 inline 函数则不会。

### 3. 尽可能使用 const

const 可以帮助编译器侦测错误的用法。例如，令函数返回一个常量值，往往可降低因调用者错误而造成的意外，而又不至于放弃安全性和高效性。比如当比较语句少写了一个 = 时：

```cpp
// 本意是 ==，结果导致在 a * b 的临时变量上调用 operator=
if (a * b = c) ...
```

如果 operator= 返回值不是 const 会导致以上错误代码编译通过！

bitwise constness 认为 const 成员函数不可以更改对象内任何 non-static 成员变量，logical constness 主张在调用者侦测不出的前提下可以修改对象内某些 bits，可以利用 mutable 释放掉 non-static 成员变量的 bitwise constness 约束。

在 const 和 non-const 成员函数中避免重复的做法是：让 non-const 成员函数调用 const 成员函数，而不要反过来。

### 4. 确定对象被使用前已被初始化

为内置型对象进行手工初始化，因为 C++ 不保证初始化它们。

构造函数最好使用成员初值列，而不是赋值操作，排列顺序最好和声明次序想同。

为避免跨编译单元的初始化次序问题，用 local static 对象代替 non-local static 对象，参考 Singleton 模式常见实现。

```cpp
XClass& GetInstance()
{
    static XClass instance;
    return instance;
}
```

## 第二章 构造/析构/赋值运算

### 5. 了解 C++ 默默编写并调用哪些函数

编译器可以隐式为类创建：默认构造函数、复制构造函数、赋值构造函数、析构函数。

### 6. 若不想使用编译器自动产生的函数，就应该明确拒绝

拒绝的普遍方法是：把函数设为 private，只有声明没有实现。但 member 函数和 friend 函数还是可以调用 private 函数，由于没有实现，会在连接期报错，不利排插，将错误移至编译期的方法是：private 继承 Uncopyable 类，Boost 也有个类，名为 noncopyable。

更新式的做法是把函数声明为 = delete。

### 7. 为多态基类声明 virtual 析构函数

任何类只要带有 virtual 函数，都几乎确定应该有一个 virtual 析构函数。但有 virtual 函数会降低调用效率和可优化性，所以能不用则不用，比如说，某个类没有考虑作为基类（base class）被继承，则没有必要有 virtual 析构函数，STL 的容器大多如此。

### 8. 别让异常逃离析构函数

如果一个被析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常，然后吞下他们或者结束程序。

如果客户需要对某个函数运行期间抛出的异常做出反应，那么类应该提供一个普通函数执行该操作，而非在析构函数中。

### 9. 绝不在构造和析构过程中调用 virtual 函数

因为这类调用从不降至派生类（derived class），它将调用本层的函数。

### 10. 令 operator= 返回一个 reference to *this

这样才能支持连锁赋值，a = b = c = d。

### 11. 在 operator= 中处理“自我赋值”

方法有：比较来源和目标对象的地址、精心周到的语句顺序、copy-and-swap。要考虑自我赋值的概率，如果很小，则比较地址的方式可能并不好，因为无视它效率更高。

### 12. 复制对象时勿忘其每一个成分

复制函数应该保证复制“对象内的所有成员变量”及“所有基类成分”。当你编写一个复制函数，请确保（1）复制所有 local 成员变量，（2）调用所有基类内的适当的复制函数。

不要尝试以某个复制函数实现另一个复制函数。应该将共同机能放进第三个函数中，并由两个复制函数共同调用。

## 第三章 资源管理

### 13. 以对象管理资源

获得资源后立刻放进对象（managing object）内。“以对象管理资源”又称“资源取得时机就是初始化时机”（Resource Acquisition Is Initialization; RAII）

管理对象（managing object）运用析构函数确保资源被释放。

为防止资源泄漏，请使用 RAII 对象，它们在构造函数中获得资源并在析构函数中释放资源。

常被使用的 RAII class 是 std::shared_ptr，它是“引用计数器型智能指针”（Reference-counting smart pointer; RCSP），它无法打破环形引用（cycles of reference）。

不要用智能指针管理动态分配的数组，因为会导致错误形式的释放。参考《[\[C++ 学习笔记 1\] delete 和 delete \[\] 的本质区别](https://my.oschina.net/umu618/blog/818839)》。

### <a name="item14">14. 在资源管理类中小心 coping 行为</a>

复制 RAII 对象必须一并复制它管理的资源，常见的 RAII class copying 行为是：

（1）禁止复制；

（2）对底层资源祭出“引用计数法”（reference-count）；

（3）复制底部资源；

（4）转移底部资源的拥有权。

### 15. 在资源管理类中提供对原始资源的访问

APIs 往往要求访问原始资源（raw resource），所以每一个 RAII class 应该提供一个“取得其所管理之资源”的方法，比如 .get()。

对原始资源的访问可能经由显式转换或隐式转换。一般而言，显式转换比较安全，但隐式转换对客户比较方便。

### 16. 成对使用 new 和 delete 时要采取相同形式

如果你在 new 表达式中使用 \[\]，必须在相应的 delete 表达式中也使用 \[\]。如果你在 new 表达式中不使用 \[\]，一定不要在相应的 delete 表达式中使用 \[\]。参考《[\[C++ 学习笔记 1\] delete 和 delete \[\] 的本质区别](https://my.oschina.net/umu618/blog/818839)》。

### 17. 以独立语句将 newed 对象置入智能指针

以独立语句将 newed 对象存储于（置入）智能指针内。如果不这么做，一旦异常被抛出，有可能导致难以察觉的资源泄漏。

```cpp
// 编译器可能为了产生更高效代码，而弹性地改变三个元语句的执行顺序
// 如果 priority() 抛出异常，可能导致 new Widget 返回的指针遗失
processWidget(std::shared_ptr<Widget>(new Widget), priority());

// 以下独立语句可行，因为编译器对“跨越语句的各项操作”没有重新排序的自由。
std::shared_ptr<Widget> pw(new Widget);
processWidget(pw, priority());
```

## 第四章 设计与声明

### 18. 让接口容易被正确使用，不易被误用

“促进正确使用”的办法包括接口的一致性，以及与内置类型的行为兼容。一致性的例子：STL 容器都有 size 成员函数。不一致性对开发人员造成的心理负担，没有任何一个 IDE 可以完全抹除。

 “阻止误用”的办法包括建立新类型、限制类型上的操作，束缚对象值，以及消除客户的资源管理责任。

std::shared_ptr 使用每个指针专属的删除器，消除“cross-DLL problem”；它还支持定制删除器，可被用来自动解除互斥锁（mutexes，见[条款 14](#item14)）。

### 19. 设计 class 犹如设计 type

Class 设计就是 type 的设计，在定义一个新 type 之前，要考虑以下主题：

（1）新 type 的对象应该如何被创建和销毁？

（2）对象的初始化和赋值该有什么差别？

（3）新 type 的对象如果被以值传递（pass by value），意味着什么？

（4）什么的新 type 的合法值？setter 函数要检查错误。

（5）新 type 需要配合某个继承图系（inheritance graph）吗？这影响函数——尤其是析构函数，是否为 virtual（见[条款 7](#7-为多态基类声明-virtual-析构函数)）。

（6）新 type 需要什么样的转换？如果希望 T1 被隐式转换为 T2，必须在 class T1 内写一个类型转换函数（operator T2）或在 class T2 内写一个可被单一实参调用（non-explicit-one-argument）的构造函数。如果只允许 explicit 构造函数存在，就得写出专门负责转换的函数，且不得为类型转换操作符（type conversion perators）或 non-explicit-one-argument 构造函数。（条款 15 有隐式和显式转换函数的范例，[https://my.oschina.net/umu618/blog/839649](https://my.oschina.net/umu618/blog/839649)）

（7）什么样的操作符和函数对此新 type 而言是合理的？这决定你的 class 有哪些函数，其中哪些是 member 函数，哪些则否。（参考条款 23, 24, 26）

（8）什么样的标准函数应该驳回？声明为 private。（见[条款 6](#6-若不想使用编译器自动产生的函数，就应该明确拒绝)）

（9）谁该取用新 type 的成员？这个问题帮你决定成员的可见性（public、protected、private）。也帮你决定哪个 classes 和/或 functions 应该是 friends，以及将它们嵌套于另一个之内是否合理。

（10）什么是新 type 的未声明接口（undeclared interface）？它对效率、异常安全性（见条款 29）以及资源运用（例如多任务锁定和动态内存）提供何种保证？你在这些方面提供的保证，将为你的 class 实现代码加上相应的约束条件。

（11）新 type 有多么一般化？new class or new class template?

（12）真的需要一个新 type 吗？如果只是定义新的子类（derived class）以便为既有 class 添加机能，那么也许单纯定义一或多个 non-member 函数或

### 20. 宁以 pass-by-reference-to-const 替换 pass-by-value

尽量以 pass-by-reference-to-const 替换 pass-by-value。前者通常比较高效，并可避免切割问题（slicing problem，即派生类被转化成基类时丢失派生类特有的成分）。

以上规则并不适用于内置类型，以及 STL 的迭代器和函数对象。对它们而言，pass-by-value 往往比较适当。

### 21. 必须返回对象时，别妄想返回其 reference

不要返回 pointer 或 reference 指向一个 local stack 对象，因为离开作用域即被销毁。

不要返回 reference 指向一个 heap-allocated 对象，因为无法保证配套 delete。

不要返回 pointer 或 reference 指向一个 local static 对象而有可能同时需要多个这样的对象。

```cpp
#include <stdio.h>

#define _WINSOCK_DEPRECATED_NO_WARNINGS // to use inet_ntoa
#include <winsock2.h>

#pragma comment(lib, "ws2_32.lib")

int main()
{
    in_addr a1 = {1, 2, 3, 4};
    in_addr a2 = {5, 6, 7, 8};
    printf_s("You think it's: %s, ", inet_ntoa(a1));
    printf_s("%s\n", inet_ntoa(a2));
    printf_s("But in fact it's: %s, %s\n", inet_ntoa(a1), inet_ntoa(a2));
    return 0;
}

```

以上代码输出为：

> You think it's: 1.2.3.4, 5.6.7.8  
> But in fact it's: 1.2.3.4, 1.2.3.4

### 22. 将成员变量声明为 private

切记将成员变量声明为 private。这可赋予客户访问数据的一致性、可细微划分访问控制、允诺约束条件获得保证，并提供 class 作者以充分的实现弹性。

protected 并不比 public 更具封装性。

### 23. 宁以 non-member、non-friend 替换 member 函数

宁可拿 non-member non-friend 函数替换 member 函数。这样做可以增加封装型、包裹弹性（packaging flexibility）和技能扩充性。

### 24. 若所有参数皆需类型转换，请为此采用 non-member 函数

member 函数的反面是 non-member 函数，而不是 friend 函数。

设计 operator * 时，要能支持乘法交换律。

如果你需要为某个函数的所有参数（包括 this 指针所指的那个隐喻参数）进行类型转换，那么这个函数必须是个 non-member。从 Object-Oriented C++ 跨进 Template C++ 时，会有新争议和解法，参考条款 46。

### 25. 考虑写出一个不抛异常的 swap 函数

通常我们不能改变 std 命名空间内的任何东西，但可以为 temlates 制造特化版本。

C++ 只允许对 class templates 偏特化（partially specialize），而对 function templates 则不许。

当 std::swap 对你的类型效率不高时，提供一个 swap 成员函数，并确定这个函数不抛出异常。因为成员 swap 的一个最好应用是帮助 classes 和 class templates 提供强烈的异常安全性（exception-safety）保障。条款 29 细说。

如果你提供了一个 member swap，也该提供一个 non-member swap 用来调用前者。对于 classes（而非 templates），也请特化 std::swap。

调用 swap 时应针对 std::swap 使用 using 声明式，然后调用 swap 并不带任何“命名空间资格修饰”。

为“用户定义类型”进行 std templates 全特化是好的，但千万不要尝试在 std 内加入某些对 std 而言全新的东西。

## 第五章 实现

### 26. 尽可能延后变量定义式的出现时间

太早出现，可能因为下面出现异常，导致构造白白浪费。

延后可以增加程序的清晰度、改善效率。

### 27. 尽量少做转型动作

dynamic_casts 有性能代价，应该尽量避免。绝对要避免“连串动态转型”（cascading dynamic casts）。

如果转型是必要的，试着将它隐藏于某个函数。客户可以条用该函数，而不需要讲转型放进他们的代码内。

宁可使用 C++-style 转型，不要使用旧式转型。前者很容易辨识出来，而且有分门别类的职掌。

### 28. 避免返回 handles 指向对象内部成分

避免返回 handles（包括 reference、指针、迭代器）指向对象内部，可以增加封装型，帮助 const 成员函数的行为像个 const，将发生“虚吊号码牌”（dangling handles）的可能性降至最低。

反之，传出去的 handles 可能让你暴露在“handles ”的风险下。

### 29. 为“异常安全”而努力是值得的

当异常被抛出时，异常安全的函数会：（1）不泄漏任何资源；（2）不允许数据败坏。

异常安全函数（Exception-safe functions）提供这三个保证之一：（1）基本承诺，如果异常抛出，程序内的任何事物仍然保持在有效状态下。（2）强烈保证，如果异常抛出，程序状态不改变。（3）不抛掷（nothrow）保证，承诺绝不抛出异常。

“强烈保证”往往能够以 copy-and-swap 实现出来，但“强烈保证”并非对所有函数都可以实现或具备现实意义。

木桶原理：函数提供的“异常安全保证”通常最高只等于其所调用的各个函数的“异常安全保证”中的最弱者。

### 30. 透彻了解 inlining 的里里外外

将大多数 inlining 限制在小型、被频繁调用的函数身上。这可使日后的调试过程和二进制升级（binary upgradability）更容易，也可使潜在的代码膨胀问题最小化，使程序的速度提升机会最大化。

定义于 class 内的函数都隐性成为 inline，包括像 operator* 这样的 friend 函数。

不要只因为 function templates 出现在头文件，就将它们声明为 inline。

所有对 virtual 函数的调用（除非是最平淡无奇的）都会使 inlining 落空。

编译器通常不对“通过函数指针而进行的调用”实施 inlining，这意味着对 inline 函数的调用最终是否 inlined 由编译器决定。

构造函数和析构函数往往是 inlining 的糟糕候选，因为他们隐含一些由编译器产生的代码。

inline 函数的风险：它们无法随着程序库的升级而升级，必须重新编译。

### 31. 将文件间的编译依存关系降至最低

支持“编译依存性最小化”的一般构想是：相依于声明式，不要相依于定义式。基于此构想的两个手段是 Handle classes 和 Interface classes。

程序库头文件应该以“完全且仅有声明式”（full and declaration-only forms）的形式存在。这种做法适用于 templates。

（1）如果使用 object references 或 object pointers 可以完成任务，就不要使用 object。

（2）如果可以，尽量以 class 声明式替换 class 定义式。

（3）为声明式和定义式提供不同的头文件。

Java 和 .NET 都不允许在 Interfaces 内实现成员变量或成员函数，但 C++ 可以。

Handle classes 和 Interface classes 有微小的性能损失，但为了降低 classes 之间的耦合性是值得的。如果性能比耦合性重要，才用具象类（concrete ）替换它们。

## 第六章 继承与面向对象设计

### 32. 确定你的 public 继承塑模出 is-a 关系

“public 继承”意味 is-a。适用于 base classes 身上的每一件事一定也适用于 derived classes 身上，因为每一个 derived classes 对象也都是一个 base class 对象。

classes 之间的关系除了 is-a 之外，还有 has-a（有一个）和 is-implemented-in-terms-of（根据某物实现出）两种常见的关系。

### 33. 避免遮掩继承而来的名称

derived classes 内的名称会掩盖 base classes 内的名称。在 public 继承下从来没有人希望如此。

为了让被掩盖的名称再见天日，可使用 using 声明式或转交函数（forwarding function）。

### 34. 区分接口继承和实现继承

接口继承和实现继承不同。在 public 继承之下，derived classes 总是继承 base class 的接口。

成员函数的接口总是会被继承。

pure virtual 函数有两个最突出的特征：他们必须被任何“继承了它们”的具象 class 重新声明，而且它们在抽象 class 中通常没有定义。

声明一个 pure virtual 函数的目的是为了让 derived classes 只继承函数接口。

声明简朴的（非纯）impure virtual 函数的目的，是让 derived classes 继承该函数的接口和缺省实现。

声明 non-virtual 函数的目的是为了令 derived classes 继承函数的接口及一份强制性实现。

### 35. 考虑 virtual 函数以外的其它选择

#### 藉由 Non-Virtual 手法实现 Template Method 模式

```
class GameCharacter {
public:
    int HealthValue() const           // derived classes 不重新定义它
    {
        ...
        int value = DoHealthValue();
        ...
        return value;
    }
    ...
private:    // 不是必须 private
    virtual int DoHealthValue() const // derived classes 可重新定义它
    {
        ...                           // 缺省算法
    }
};
```

令客户通过 public non-virtual 成员函数间接调用 private virtual 函数，称之为 non-virtual interface（NVI）手法。它是 Template Method 设计模式的一个独特表现形式。non-virtual 函数称为 virtual 函数的外覆器（wrapper）。

#### 藉由 Function Pointers 手法实现 Strategy 模式

缺点：将机能从成员函数移到 class 外部函数，导致非成员函数无法访问 class 的 non-public 成员。

#### 藉由 std::function 手法实现 Strategy 模式

#### 古典的 Strategy 模式

古典的 Strategy 模式会将健康函数做成一个分离的继承体系中的 virtual 成员函数。

### 36. 绝不重新定义继承而来的 non-virtual 函数

- class 内声明一个 non-virtual 函数会为该 class 建立起一份不变性（invariant），凌驾其特异性（specialization）。

### 37. 绝不重新定义继承而来的缺省参数值

- 本条款的讨论局限于“继承一个带有缺省参数值的 virtual 函数”，绝对不要新定义继承而来的缺省参数值，因为缺省参数都是静态绑定，而 virtual 函数——你唯一应该复写的东西——却是动态绑定。

### 38. 通过复合塑模出 has-a 或“根据某物实现出”

- 复合（composition）的意义和 public 继承完全不同。

- 在应用域（application domain），复合意味 has-a（有一个）。在实现域（implementation domain），复合意味 is-implemented-in-terms-of（根据某物实现出）。

### 39. 明智而审慎地使用 private 继承

- Private 继承意味 is-implemented-in-terms-of（根据某物实现出）。它通常比复合（composition）的级别低。但是当 derived class 需要访问 protected base class 的成员，或需要重新定义继承而来的 virtual 函数时，这么设计是合理的。

- 和复合（composition）不同，private 继承可以造成 empty base 最优化。这对致力于“对象尺寸最小化”的程序库开发者而言，可能很重要。

### 40. 明智而审慎地使用多重继承

- 多重继承比单一继承复杂。它可能导致新的歧义性，以及对 virtual 继承的需要。

- virtual 继承会增加大小、速度、初始化（及赋值）复杂度等等成本。如果 virtual base classes 不带任何数据，将是最具实用价值的情况。

- 多重继承的确有正当用途。其中一个情节涉及“public 继承某个 Interface class”和“private 继承某个协助实现的 class”的两相组合。

## 第七章 模板与泛型编程

### 41. 了解隐式接口和编译期多态

- class 和 template 都支持接口（interface）和多态（polymorphism）。

- 对 class 而言接口是显示（explicit）的，以函数签名为中心。多态则是通过 virtual 函数发生于运行期。

- 对 template 参数而言，接口是隐式的（implicit），奠基于有效表达式。多态则是通过 template 具现化和函数重载解析（function overloading resolution）发生于编译期。

### 42. 了解 typename 的双重意义

- 声明 template 参数时，前缀关键字 class 和 typename 可互换。

- 请使用关键字 typename 标识嵌套从属类型名称；但不得在 base class lists（基类列）或 member initialization list（成员初值列）内以它作为 base class 修饰符。

### 43. 学习处理模板化基类内的名称

- 当我们从 Object Oriented C++ 跨进 Template C++，继承就不像以前那么畅行无阻了。面对一些编译不通过的情况，可在 derived class templates 内通过 “this->” 指涉 base class templates 内的成员名称，或藉由一个明白写出的 “base class 资格修饰符” 完成。

### 44. 将与参数无关的代码抽离 templates

- Templates 生成多个 classes 和多个函数，所以任何 template 代码都不该与某个造成膨胀的 template 参数产生相依关系。

- 因非类型模板参数（non-type template parameters）而造成的代码膨胀，往往可消除，做法是以函数参数或 class 成员变量替换 template 参数。

- 因类型参数（type parameters）而造成的代码膨胀，往往可降低，做法是让带有完全相同二进制表述（binary representations）的具现类型（instantiation types）共享实现码。

### 45. 运用成员函数模板接受所有兼容类型

- 请使用成员函数模板（member function templates）生成“可接受所有兼容类型”的函数。

- 如果你声明 member templates 用于“泛化 copy 构造”或“泛化 assignment 操作”，你还是需要声明正常的 copy 构造函数和 copy assignment 操作符。

### 46. 需要类型转换时请为模板定义非成员函数

函数的参数可以隐式转换，函数模板不行。

friend 的传统用途是“访问 class 的 non-public 成分”。

在 class 内部声明 non-member 函数的唯一办法就是：令它成为一个 friend。

- 当我们编写一个 class template，而它所提供之“于此 template 相关的”函数支持“所有参数之隐式类型转换”时，请将那些函数定义为“class templdate 内部的 friend 函数”。

### 47. 请使用 traits classes 表现类型信息

STL 的迭代器有 5 种：input、output、forward、bidirectional、random access，为了给它们实现统一的 advance 函数，需要再编译期间判断迭代器的类型，这可以使用 Traits 技术来实现。

- 建立一组重载函数（身份像劳工）或函数模板（例如 doAdvance，最后一个参数是 typename std::iterator_traits<IterT>::iterator_category()），彼此间的差异只在于各自的 traits 参数（可以不命名，只是一个类型）。令每个函数实现码与其接受之 traits 信息相应和。

- 简历一个控制函数（身份像工头）或函数模板（例如 advance），它调用上述那些“劳工函数”并传递 traits class 所提供的信息。

- Traits classes 使得“类型相关信息”在编译期可用。它们以 templates 和“templates 特化”完成实现。

- 整合重载技术（overloading）后，traits classes 有可能在编译期对类型执行 if...else 测试。

### 48. 认识 template 元编程

Traits 解法属于 TMP（Template metaprogramming，模板元编程），比 typeid-based 解法高效，高效的原因：

- 编译期测试类型。

- 类型测试代码不会被链接到可执行程序中。

TMP 是图灵完备（Turling-complete）的。

- TMP 可将工作由运行期移往编译期，因而得以实现早期错误侦测和更高的执行效率。

- TMP 可被用来生成“基于政策选择组合”（based on combinations of policy choices）的客户定制代码，也可用来避免生成对某些特殊类型并不适合的代码。

## 第八章 定制 new 和 delete

### 49. 了解 new-handler 的行为

当 operator new 无法满足某一内存分配需求时，它会先调用一个客户指定的错误处理函数 new-handler。（这并非全部事实，参考条款 51）为了指定“用以处理内存不足”的函数，客户必须调用声明于 \<new\> 的标准库函数 set_new_handler：

```cpp
namespace std {
  typedef void (*new_handler)();
  new_handler set_new_handler(new_handler p) noexcept;
}
```

用法示例：

```cpp
void OutOfMemory() {
  std::cerr << "Unable to satisfy request for memory!\n";
  std::abort();
}

int main() {
  std::set_new_handler(OutOfMemory);
  char* big_array = new char[32 * 1024 * 1024 * 1024];
}
```

设计良好的 new-handler 函数必须做一下事情：

- **让更多内存可被使用**。实现此策略的一个做法是：程序一开始就分配一大块内存，而后当 new-handler 第一次被调用，将它们释还给程序使用。

- **安装另一个 new-handler**。如果当前 new-handler 无法取得更多可用内存，或许它知道另外哪个 new-handler 由此能力。

- **卸除 new-handler**，也就是将 nullptr 传给 set_new_handler，这样 operator new 会在内存分配失败时抛出异常。

- **抛出 bad_alloc 或其派生的异常**。这样的异常不会被 operator new 捕捉，而会被传播到内存索求处。

- **不返回**，通常调用 abort 或 exit。

C++ 不支持 class 专属之 new-handler，如果您需要，可以自己实现，只需每个 class 提供自己的 set_new_handler 和 operator new。

nothrow/noexcept new 是一个颇为局限的工具，因为它只适用于内存分配；后续的构造函数调用还是可能抛出异常。

### 50. 了解 new 和 delete 的合理替换时机

替换编译器提供的 operator new/delete 的最常见理由：

- **用来检测运用上的错误。**

- **为了强化效能。**

  - 增加分配和归还速度。
  
  - 降低缺省内存管理器带来的空间额外开销。

  - 弥补缺省分配器种的非最佳齐位（suboptimal alignment）。

  - 将相关对象成簇集中。

- **为了收集使用上的统计数据。**

- **为了获得非传统的行为。**

### 51. 编写 new 和 delete 时需固守常规

- operator new 应该内含一个无穷循环，并在其中尝试分配内存，如果它无法满足内存需求，就该调用 new-handler。它也应该有能力处理 0 bytes 申请。Class 专属版本则还应该处理“比正确大小更大的（错误）申请”。

- operator delelte 应该在收到 null 指针时不做任何事。Class 专属版本处理“比正确大小更大的（错误）申请”。

### 52. 写了 placement new 也要写 placement delete

- 当您写一个 *placement* operator new，请确定也写出了对应的 *placement* operator delete。否则，您的程序可能会发生隐微而时断时续的内存泄露。

- 当您声明 *placement* new 和 *placement* delete，请确定不要无意识（非故意）地遮掩了它们的正常版本。

## 第九章 杂项讨论

### 53. 不要轻忽编译器的警告

- 严肃对待编译器发出的警告信息。努力在您的编译器的最高（最严苛）警告级别下争取“无任何警告”的荣誉。

- 不要过度依赖编译器的报警能力，因为不同的编译器对待事情的态度并不相同。一旦移植到另一编译器上，您原本依赖的警告信息有可能消失。

### 54. 让自己熟悉 STL

- C++ 标准程序库的主要机能由 STL、iostream、locales 组成。并包含 C99 标准程序库。

- 需要熟悉智能指针（例如 std::shared_ptr）、一般化函数指针（std::function）、hash-based 容器、正则表达式（regular expressions）等。

### 55. 让自己熟悉 Boost

- Boost 是一个社群，也是一个网站。致力于免费、开源、同僚复审的 C++ 程序库开发。Boost 在 C++ 标准化过程中扮演深具影响力的角色。

- Boost 提供数十个类目实现，例如：字符串与文本处理、容器、函数对象和高级编程、泛型编程、模板元编程、数学和数值、正确性与测试、数据结构、语言间的支持、内存、杂项。

（完，最后更新：2020-07-26）

京东联盟购买链接：

推荐购买新版的 [Effective Modern C++](https://union-click.jd.com/jdc?e=&p=JF8BAMUJK1olXg8CXF1UCEkXCl8LElIQWQcBUFxUCXtTXDdWRGtMGENDFlVDFhNSVzMXQA4KD1heSl1UAU4TAmwMGVIUQl9HCANtfjlFUTttGB93A3hCNztHXUxgRT1vXVcZbQcyVF9fDUkVAGwOK2sVWjZDOltdCUIVM244G1wQXQUFUFdaCkIRBl8PG1IlOXpmU15UOHsnAF84K1slXjZAOg0PWxxEUGoKSVgRXAMHAA4NCR8eBzwPTlITXVFXBAkJOEkWAmsBKw) 中国电力出版社 2018年04月01日 平装

![Effective Modern C++](/images/effective-modern-cpp.png) ![Effective Modern C++ QR Code](/images/effective-modern-cpp-jd.png)

[Effective C++：改善程序与设计的55个具体做法（第3版 中文版）(博文视点出品)](https://union-click.jd.com/jdc?e=&p=AyIGZRhfEgoaBVUbXBYyEgZXE1kXAhs3EUQDS10iXhBeGlcJDBkNXg9JHUlSSkkFSRwSBlcTWRcCGxgMXgdIMnt3UkItCxxVZAdtM3FaVF8jUkESYmILWSteFQMTBFIcXRwKIgdUGV4XABEEUytrFQMiUTsbWhQDEwZQG14WMhAGVBlYHQQUB1UrWxIDGgJcG1sSCxYCVStcFQsiYyl%2FXBULIjdVGlkdABAHXCtYJTIiB2UYa1dsEQUHElIVVxICUBtYEAoQAwVIWhAGRVcAGV5AVUEAUk9rFwMTA1w%3D) 出版时间：2011-01-01 用纸：轻型纸 含试读

[Effective C++中文版：改善程序与设计的55个具体做法（第3版）](https://union-click.jd.com/jdc?e=&p=AyIGZRprFQMTAFIeXxAyVlgNRQQlW1dCFFlQCxxKQgFHRE5XDVULR0UVAxMAUh5fEB1LQglGa0JlbHETQQRVYktPAU8fa2p3WApZEGUOHjdQG1oUARUAUxJTJQITBVAZWRYBFDdlG1olSXwGZRlaFAARD1MdWxUyEgBUE14cAxUPXRNYFDIVB1wrP2lmFQdcK2sVAxMAUh5fEDIRN2UrWyUBIkU7GFlHCxsHABteEAIRAl0ZX0VREwJRTAtAABdSAkhcElYiBVQaXxw%3D) 出版时间：2006-07-01
