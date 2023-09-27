# CPP 简介

C++ 是一门面向对象的语言，起初被称作 "C with classes"，作为 C 语言的增强版出现，随后又不断添加了许多新特性。我们的实验实验并不需要你们使用所有高级的 C++ 特性，在此简单介绍一下部分特性便于理解。如果对 C++ 有更深的兴趣，可以从 Milo Yip 的[游戏程序员的学习之路](https://github.com/miloyip/game-programmer/blob/master/game-programmer-zh-cn.jpg?raw=true)的 C++ 部分开始看。

注：本介绍假设你有基本的 C 语言认知（略高于程设课标准），如果有不懂的 C 语言术语请自行搜索。

## STL
STL (Standard Template Library)，意为标准模板库，包含了许多常用的数据结构，在我们的实验中你可能会用到：

- `std::vector`: 可变长数组
- `std::map`: 由红黑树实现的有序关联容器
- `std::set`: 由红黑树实现的有序集合
- `std::unordered_map`: 由哈希表实现的无序关联容器
- `std::list`: 链表

这里的 `std` 是 C++ 中的命名空间，可以防止标识符的重复，详见 [维基百科](https://en.wikipedia.org/wiki/Namespace)

同时，这些容器都是模板 [Template](https://en.wikipedia.org/wiki/Template_(C%2B%2B))，需要实例化，例如一个可变长的整形数组应该实例化为 `std::vector<int>`。

更多信息可以参考：[https://en.cppreference.com/w/cpp/standard_library](https://en.cppreference.com/w/cpp/standard_library)

## String
C++ 提供了更易用的 `std::string` 以处理字符串，可以支持通过 `+` 拼接，还提供了许多方法：

- `length`: 返回字符串长度。
- `push_back(c)`: 在字符串末尾添加一个字符 `c`。
- `append(str)`: 在字符串末尾添加字符串 `str`。
- `substr`: 取子串。

更多方法请查阅：[https://en.cppreference.com/w/cpp/string](https://en.cppreference.com/w/cpp/string)

## Auto

`auto` 关键字可以用于当类型已知时自动推断类型，例如：

```cpp
std::vector<std::string> v;
v.push_back("compile");
auto s = v.front(); // 这里 s 就是 std::string 类型
```

## class

class 是 C++ 面向对象的基础，它相当于对 C 中的结构体的扩展。除了保留了原来的结构体成员（即成员对象），它增加了成员函数、访问控制、继承和多态等。

假设某类为`Animal`，一个它的实例为`animal`，我们可以在`Animal`的定义中增加函数声明`void eat();`，这样声明的函数即是成员函数。成员函数的作用域中自带一个`Animal*`类型的指针`this`，指向调用该成员函数的实例。我们可以通过`animal.eat()`一样，用类似访问成员的方法访问该函数。

在面向对象的术语中，我们通常称成员函数为“方法”（[method](https://en.wikipedia.org/wiki/Method_(computer_programming))）。

```cpp
// 注：C++ 中 struct 也会定义结构体，只是访问控制的默认选项有所区别
struct Animal {
  void eat()
}
```

类的访问控制指的是在定义 class 时，可以用`public`与`private`标签，指定接下来的成员是私有或是公开成员。公开成员可以在外部函数使用该类的实例时访问，而内部成员只有该类的成员函数能访问。访问控制的作用是对使用者隐藏实现的细节，而关注于设计者想要公开的接口，从而让使用者能更容易理解如何使用该类。详细介绍在[access specifiers](https://en.cppreference.com/w/cpp/language/access)。

我们通常将类的声明放到头文件中，同时为了区别 C 的头文件，用后缀 `.hpp` 表示 C++ 的头文件。

## 类的继承

类的继承是一种面向对象语言常用的代码复用方法，也是一种非常直观的抽象
