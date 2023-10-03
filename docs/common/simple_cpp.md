# CPP 简介

C++ 是一门面向对象的语言，在 C 语言之上添加了许多新特性。我们的实验并不需要你们使用所有高级的 C++ 特性，在此简单介绍一下部分特性以便更好地完成实验。

## STL

STL (Standard Template Library)，意为标准模板库，包含了许多常用的数据结构，在我们的实验中你可能会用到：

- `std::vector`: 可变长数组
- `std::map`: 由红黑树实现的有序关联容器
- `std::set`: 由红黑树实现的有序集合
- `std::unordered_map`: 由哈希表实现的无序关联容器
- `std::list`: 链表

这里的 `std` 是 C++ 中的命名空间，可以防止标识符的重复，详见 [维基百科](https://en.wikipedia.org/wiki/Namespace)

同时，这些容器都是模板 [Template](<https://en.wikipedia.org/wiki/Template_(C%2B%2B)>)，需要实例化，例如一个可变长的整形数组应该实例化为 `std::vector<int>`。

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

在面向对象的术语中，我们通常称成员函数为“方法”（[method](<https://en.wikipedia.org/wiki/Method_(computer_programming)>)）。

```cpp
// 注：C++ 中 struct 也会定义结构体，只是访问控制的默认选项有所区别
struct Animal {
  void eat()
}
```

类的访问控制指的是在定义 class 时，可以用`public`与`private`标签，指定接下来的成员是私有或是公开成员。公开成员可以在外部函数使用该类的实例时访问，而内部成员只有该类的成员函数能访问。访问控制的作用是对使用者隐藏实现的细节，而关注于设计者想要公开的接口，从而让使用者能更容易理解如何使用该类。详细介绍在 [access specifiers](https://en.cppreference.com/w/cpp/language/access)。

我们通常将类的声明放到头文件中，同时为了区别 C 的头文件，用后缀 `.hpp` 表示 C++ 的头文件。

## 类的继承

类的继承是一种面向对象语言常用的代码复用方法，也是一种非常直观的抽象方式。我们可以定义`struct Cat : Animal`来声明`Cat`类是`Animal`类的子类，也就是`Cat`继承了`Animal`类。此时，新的`Cat`类从`Animal`类中继承了`void eat();`成员函数，并且可以在此之上定义额外的成员函数`void nyan()`。同理，我们也可以定义`struct Dog : Animal`来定义`Dog`类。

```cpp
struct Cat : Animal {
  // 从 Animal 中继承了 void eat();
  void nyan()
};

struct Dog : Animal {
  // 从 Animal 中继承了 void eat();
  void wang()
};
```

我们可以通过合理的继承结构来将函数定义在合适的位置，使得大部分通用函数可以共享。

同学们可能会想到同是`Animal`，`Cat`和`Dog`可能会有相同名称与参数的函数，但是却有着不同的实现，这时我们就要用到虚函数了。子类中可以定义虚函数的实现，从而使得不同子类对于同一个名字的成员函数有不同实现。

我们可以用子类的指针给派生类指针赋值，在基类指针上调用虚函数时，会通过虚函数表查找到对应的函数实现，而不是和普通类一样查找对应类型的函数实现。

```cpp
struct Animal {
  // = 0 表示该虚函数在 Animal 类中没有实现
  virtual void say() = 0;
};

struct Cat : Animal {
  // override 表示覆盖父函数中的实现，下同
  void say() override {
    std::cout << "I'm a cat" << std::endl;
  }
};

struct Dog : Animal {
  void say() override{
    std::cout << "I'm a dog" << std::endl;
  }
};

// 试一试
int main() {
  Cat c;
  Dog d;
  Animal* a;
  c.say();
  d.say();
  a = &c;
  // 这里基类指针指向 Cat 类实例，调用 say 时实际会调用 Cat 类的实现，下同
  a->say();
  a = &d;
  a->say();
  return 0;
}
```

同时，既然子类的指针可以赋值给基类，例如上面我们让 `Animal *a = &c`，这时 `a` 指向的是 `Cat` 类的实例，为了得到 `Cat` 类的对应指针，我们可以使用 `dynamic_cast`：

```cpp
auto cc = dynamic_cast<Cat *>(a);
```

这样我们就可以把基类指针又转回对应的子类指针。当然，如果指针不是 `Cat *` 类型的，`dynamic_cast` 将会返回 `nullptr`。（与 C 中统一使用 `NULL` 不同，在 C++ 中，我们用 `nullptr` 表示空指针）

在确信 cast 一定成功时，也可以使用 `static_cast` 来完成转换，可以省去运行时的类型检查。此外，`static_cast` 还用于基本类型的转换，例如：

```cpp
float f = 3.14;
int n = static_cast<int>(f);
```

!!! warning

    你应该避免使用 C style cast，例如 `int n = (int) f`，因为 C++ style cast 会被编译器检查，而 C style 的不会。

## 函数重载

C++ 中的函数可以重载，即可以有同名函数，但是要求它们的形参必须不同。如果想进一步了解，可以阅读[详细规则](https://en.cppreference.com/w/cpp/language/overload_resolution)。下面是函数重载的示例：

```cpp
struct Point {
  int x;
  int y;
};

struct Line {
  Point first;
  Point second;
};

void print(Point p) {
  printf("(%d, %d)", p.x, p.y);
}

void print(Line s) {
  print(s.first) // s.first == Point { ... }
  printf("->");
  print(s.second) // s.second == Point { ... }
}
```

我们可以看到上面的示例定义了两个`print`函数，并且它们的参数列表的类型不同。它们实际上是两个不同的函数（并且拥有不同的内部名字），但是 C++ 能够正确的识别函数调用时使用了哪一个定义（前提是你正确使用了这一特性），并且在编译时就会链接上正确的实现。我们可以看到，这种特性非常符合人的直觉，并且没有带来额外开销。

## 内存管理

C++11 引入了许多智能指针类型来帮助自动内存管理，本实验中用到的有两种，分别是：

1. `std::shared_ptr`: 引用计数智能指针，使用一个共享变量来记录指针管理的对象被引用了几次。当对象引用计数为 0 时，说明当前该对象不再有引用，并且进程也无法再通过其它方式来引用它，也就意味着可以回收内存，这相当于低级的垃圾回收策略。可以用 `std::make_shared` 来创建。
2. `std::unique_ptr`: 表示所有权的智能指针，该指针要求它所管理的对象智能有一次引用，主要用于语义上不允许共享的对象（比如`llvm::Module`）。当引用计数为 0 时，它也会回收内存。可以用 `std::make_unique` 来创建。

在涉及到内存管理时，应该尽量使用智能指针。

## 调试

### Segmentation Fault

在之后的实验中，你可能会各种各样的段错误（Segmentation Fault），不要害怕！clang 提供了一个工具来帮助我们解决内存泄漏。

```bash
$ cd build
$ cmake .. -DCMAKE_BUILD_TYPE=Asan
$ make
```

然后再次运行你的错误代码，Asan 会提供更详细的报错信息。

注：要更换到别的 build type（如 Debug 或 Release）时需要显式指定，否则 cmake 会使用 cached 的版本。
