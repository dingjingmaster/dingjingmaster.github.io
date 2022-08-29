# C++11智能指针

## 智能指针的作用

在使用C/C++时候，使用指针理由之一就是希望变量在需要的时候突破作用域边界的限制。然而，实际使用中，确保“指针的寿命”和“其所指对象的寿命”一致是件很棘手的事情，特别是多个指针指向同一个对象时候。

<br/>
避免上述问题的一个通常做法就是使用智能指针(smart pointer)。

<br/>
自C++11起，C++标准库提供了两大类型的 smart pointer:

1. [`class shared_ptr`](#class-shared_ptr) 实现共享式拥有概念。多个 smart pointer 可以指向相同对象，该对象和其相关资源会在最后一个引用被销毁时候释放。此类指针有：`weak_ptr`、`bad_weak_ptr`
2. [`class unique_ptr`](#class-unique_ptr) 实现独占式拥有或严格拥有概念。保证同一时间内只有一个 smart pointer 可以指向该对象。你只可以移交拥有权。它对于避免内存泄漏(以new创建对象后忘记调用delete)特别有用。

> c++98标准库中提供给了一个smart pointer: `auto_ptr<>`，其设计是为了执行现今`unique_ptr`的功能。但是，它缺乏现代语言特性支持，比如：“针对构造和赋值”的`move语义`，以及其它瑕疵，在C++11之后被正式反对使用。
<br/>

> 所有 smart pointer 类都被定义于头文件 `<memory>` 内。

## 两类智能指针的使用

### `class shared_ptr`

`shared_ptr`主要应用场景是 —— 同一时间在多处地点使用同一个对象实例，当最后一个对象实例不再被需要时候才释放内存资源。
<br/>

顾名思义，多个 `shared_ptr` 共享同一个对象(多个`shared_ptr`指向同一个被共享的对象)，对象最后一个拥有者销毁对象时候，调用此对象的析构函数，在此之前，其它`shared_ptr`的销毁并不会触发对象的销毁操作(引用计数，每多一个共享对象，引用计数`+1`，每销毁一个对象，引用计数`-1`，当引用计数是`0`则销毁对象资源)。
<br/>

如果对象以 `new` 产生，默认清理工作由 `delete` 完成，若对象以 `new[]` 分配，则必须由 `delete[]` 加以清理，当然还有其它情况，因此`shared_ptr`智能指针在创建时候可以定义其对象销毁的方式。

#### 创建 `shared_ptr` 指针

```c++
// 1.
shared_ptr<string> ptr1(new string("shared ptr"));              // OK
shared_ptr<string> ptr2 = new string("shared ptr");             // 错误，单一参数构造函数是 explicit
shared_ptr<string> ptr3{new string("shared ptr")};              // OK
// 2.
shared_ptr<string> ptr4 = make_shared<string>("shared ptr");    // OK
// 3.
shared_ptr<string> ptr5;
ptr5.reset(new("shared ptr"));                                  // OK
ptr5 = new string("shared ptr");                                // 错误，
```
上述三种创建 `shared_ptr` 指针的方式，推荐使用 第二种：`make_shared<>()`。这种方式比较快、安全。

#### 使用 `shared_ptr` 指针

1. 一般使用

    使用智能指针与使用普通指针是一样的，因为它重载了 `operator*` 和 `operator->`，**但是 shared_ptr 不提供 `operator[]` 和 指针运算**，因此要想访问内存，必须使用`get()`获得被 `shared_ptr` 包裹的内部指针。
    ```c++
    smp.get()[i] = i * 42;
    // 等价
    (&*smp)[i] = i * 42;
    ```
<br/>

2. 放入容器

    但凡容器，总为传入的元素创建属于容器自己的copy，直接插入string，插入的实际上是 string 的拷贝，然而放进去的是指向 string 的指针，被复制的就是指针，于是容器内涵多个“指向同一对象”的引用。

#### `shared_ptr` 对应的删除操作

1. 程序终点处，当 `string` 最后一个拥有者被销毁时候，shared pointer 对其所指向的对象调用 delete。
2. 当最后一个 `shared pointer` 被赋值 `nullptr` 时候也会使 shared pointer 对其所指向的对象调用 delete。

#### `shared_ptr` 定义自己的delete操作

```c++
auto del = [] (string* p) {
    count << "delete " << *p << std::endl;
    delete p;
}
shared_ptr<string> ptr (new string("shared ptr"), del);
ptr = nullptr;      // 这里会导致 shared ptr 调用销毁操作
```

> 注意：`shared_ptr` 提供的默认 delete 操作调用的是 delete，不是 delete[]，因此，当你使用 new[] 创建的数组对象，则必须自定义`shared_ptr`的销毁操作，否则会造成内存泄漏。
> <br/>
> <br/>也可以使用 `unique_ptr` 提供的辅助函数作为 deleter，其内调用 delete[]：
> <br/>`std::shared_ptr<int> p(new int[10], std::default_delete<int[]>());`
<br/>

`shared_ptr`和`unique_ptr`以稍稍不同的方式处理deleter。例如：`unique_ptr`允许你只传递对应的元素类型作为`模板`实参，但对 `shared_ptr` 就不可行：
```c++
std::unique_ptr<int[]> p(new int[10]);      // OK
std::shared_ptr<int[]> p(new int[10]);      // 错误 <我这边亲测可以了...>
```

#### 错误用法
```c++
int* p = new int;
shared_ptr<int> sp1(p);
shared_ptr<int> sp2(p);

// 会出错，因为两个shared_ptr 都会在作用域之外释放对象`p`，导致对象被两次释放。
```

### `class weak_ptr`

`weak_ptr` 是解决`shared_ptr`以下两种情况下无法正常使用的问题而引入的。
<br/>

1. 环式指向：

    如果两个对象使用 `shared_ptr` 互相指向对方，一旦不存在其它引用指向它们时候，shared_ptr 并不会自动释放资源，因为每个`shared_ptr`对象的`use_count()`仍然是1.

2. 明确想共享对象，但并不想拥有对象的情况：

    引用(引用指的是`shared_ptr`智能指针)的寿命比其所指对象的寿命更长，因此通过使用智能指针可以获取到其指向对象是否还有效```class weak_ptr` 允许你"共享但不拥有"某对象，一旦最末一个拥有该对象的 shared_ptr 失去了拥有权(对象释放)，任何 `weak_ptr` 都会自动成空(empty)。

> 在默认的default和copy构造函数外，`class weak_ptr`只提供一个`shared_ptr`构造函数。

> 注意：你不能直接使用操作符`*`和`->`访问 `weak_ptr`指向的对象，而是必须另外建立一个`shared_pointer`，原因如下：
> <br/>1. 在`weak pointer`之外建立一个`shared pointer`可因此检查是否存在一个相应的对象。如果不，操作会抛出异常或建立一个空的共享指针。
> <br/>2. 当指向的对象正被操作时候，shared pointer 无法被释放。
> <br/>基于以上所述，`class weak_ptr` 只提供小量操作，只够用来创建、赋值、赋值weak pointer，以及转换为一个shared pointer，或检查自己是否指向某对象。


### `class unique_ptr`

`unique_ptr` 是C++标准库自 C++11 起开始提供的类型。它避免在异常发生时候造成资源泄漏。一般而言，这个 smart pointer 实现了独占式拥有概念，意味着它可能确保一个对象和其相应资源同一时间只被一个 pointer 拥有。一旦拥有者被销毁或编程empty，或开始拥有另一个对象，先前拥有的那个对象就会被销毁，其任何相应资源亦会被释放。

#### `class unique_ptr`的目的

通常往往以下列方式运作：
1. 获得某些资源
2. 执行某些操作
3. 将取得的资源是放掉

```c++
#include <memory>

void f () 
{
    std::unique<ClassA> ptr(new ClassA);

    // 其它操作
}
```

#### `unique_ptr` 使用

`unique_ptr`和寻常指针接口很相似，`*`用来提取指向对象；`->`用来访问成员；不提供`++`操作。

```c++
std::unique_ptr<int> up = new int;  // 错误，不允许
std::unique_ptr<int> up (new int);  // OK
std::unique_ptr<int> up;            // unique_ptr 不必一定拥有对象，也可以是 empty
```

#### 转移`unique_ptr`所有权

1. `std::move()` 可以将拥有权转移给另一个 `unique_ptr`

> 不可对`unique_ptr`执行`copy`和`assign`，`unique_ptr`唯一的独占一个对象...
> <br/>`auto_ptr`使用 `copy` 来移交拥有权，后续不再使用 `auto_ptr`，这块需要了解。

#### 源头和去处

函数可利用它们将拥有权转移给其它函数。这会发生在两种情况下：
1. 函数是接收端。如果将一个由`std::move()`建立起来的`unique_ptr`以右值引用身份当作函数参数，那么被调用函数的参数将会取得`unique_ptr`的拥有权。因此，如果该函数不再转移拥有权，对象会在函数结束时候被deleted。
    ```c++
    void sink(std::unique_ptr<ClassA> up)
    {
        // ....
    }

    sink (std::move(up)); // up 将失去所有权
    ```

2. 函数是供应端。当函数返回一个`unique_ptr`，其拥有权会转移至调用端场景内。下面的例子展示了这项技术：
    ```c++
    std::unique_ptr<ClassA> source()
    {
        std::unique_ptr<ClassA> ptr(new ClassA);    // ptr 拥有新对象
        return ptr; // 把所有权转移给函数调用者
    }

    void g ()
    {
        std::unique_ptr<ClassA> p;
        for (int i = 0; i < 10; ++i) {
            p = source();   // p 将得到返回对象的所有权
        }
    } // 此后，p对象指向的指针将会被删除
    ```

#### `unique_ptr`被当成成员

在 class 内使用`unique_ptr`可避免资源泄漏，因为只有已经完全构造好的对象，其析构函数才会被调用，所以对于拥有多个指针的类，构造期间，第一个new成功而第二个new失败就可能导致资源泄漏。

```c++
class B
{
    private:
        ClassA* ptr1;       // 使用 std::unique_ptr<ClassA> ptr1;
        ClassA* ptr2;       // 使用 std::unique_ptr<ClassA> ptr2;
    public:
        B (int val1, int val2)
            : ptr1(new ClassA(val1)), ptr2(new ClassA(val2))
        {
            // 构造时候，第一个new成功，第二个new失败，则会导致内存泄漏
        }
        ~B ()
        {
            if (ptr1)   delete ptr1;
            if (ptr2)   delete ptr2;
        }
};
```

#### 针对 array 使用`unique_ptr`

```c++
std::unique_ptr<std::string> up (new std::string[10]);      // 运行时候出错，无法正确被析构
std::unique_ptr<std::string[]> up(new std::string[10]);     // OK
std::cout << *up << std::endl;                              // 错误, * 没有针对数组的定义
```

