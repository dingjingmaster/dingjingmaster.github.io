# C 与 C++ 各个版本


## C 各个版本说明

### C 语言早期
- 最早由丹尼斯·里奇（Dennis Ritchie）为了在PDP-11电脑上运行的Unix系统所设计出来的编程语言
- 第一次发展在1969年到1973年之间。
- 在PDP-11出现后，丹尼斯·里奇与肯·汤普逊着手将Unix移植到PDP-11上
- 1973年，Unix操作系统的核心正式用C语言改写，这是C语言第一次应用在操作系统的核心编写上。
- 1975年C语言开始移植到其他机器上使用。史蒂芬·强生实现了一套“可移植编译器”

### K&R c
- 1978年，丹尼斯·里奇和布莱恩·柯林汉合作出版了《C程序设计语言》的第一版。 “K&R C”（柯里C）。

### C89/C90
- 1989年，C语言被美国国家标准协会（ANSI）标准化，这个版本又称为C89
- 标准化的一个目的是扩展 K&R C，增加了一些新特性。
- 1990年，国际标准化组织（ISO）规定国际标准的C语言
- 通过对 ANSI 标准的少量修改，最终制定了 ISO 9899:1990，又称为C90。
- 随后，ANSI亦接受国际标准C，并不再发展新的C标准。

> 注意: C89和C90是一回事，C89是由ANSI(American National Standards Institute)美国国家标准协会制定，C90 是由国际标准协会根据ANSI C89 制定。

#### C89/C90对K&R C改进点
- 增加了真正的标准库
- 新的预处理命令与特性
- 函数原型允许在函数申明中指定参数类型
- 一些新的关键字，包括 const、volatile 与 signed
- 宽字符、宽字符串与多字节字符
- 对约定规则、声明和类型检查的许多小改动与澄清

### C99
- 1994年为C语言创建了一个新标准，但是只修正了一些C89标准中的细节和增加更多更广的国际字符集支持。
- 不过，这个标准引出了1999年ISO 9899:1999的发表。它通常被称为C99。
- C99被ANSI于2000年3月采用。

#### C99新特性(部分)
- 支持不定长的数组，声明时使用 `int a[var]` 的形式。
- 变量声明不必放在语句块的开头，`for` 语句提倡写成 `for(int i=0;i<100;++i)` 的形式
- 允许采用`（type_name）{xx,xx,xx}` 类似于 C++ 的构造函数的形式*构造匿名的结构体*。
- 除了已有的 `__line__` `__file__` 以外，增加了 `__func__` 得到当前的函数名。
- 取消了函数返回类型默认为 `int` 的规定。
- 增加和修改了一些标准头文件(定义bool的、定义复数的、里增加了 struct tmx，对 struct tm 做了扩展。)

### C11
- 2011年12月8日，ISO正式发布了新的C语言的新标准C11，之前被称为C1X
- 官方名称为ISO/IEC 9899:2011
- 新的标准提高了对C++的兼容性，并增加了一些新的特性。
- 这些新特性包括泛型宏、多线程、带边界检查的函数、匿名结构等。

### C18
- C18没有引入新的语言特性，只对C11进行了补充和修正

## C++ 各个版本说明

- 1998年是C++标准委员会成立的第一年，以后每5年视实际需要更新一次标准。
- 2009年，C++标准有了一次更新，一般称该草案为C++0x。
- C++0x是C++11标准成为正式标准之前的草案临时名字。
- 后来，2011年，C++新标准标准正式通过，更名为ISO/IEC 14882:2011，简称C++11。

### C++11
C++11，先前被称作C++0x，即ISO/IEC 14882:2011，是C++编程语言的一个标准。

它取代第二版标准ISO/IEC 14882:2003 （第一版ISO/IEC 14882:1998公开于1998年， 第二版于2003年更新，分别通称C++98以及C++03，两者差异很小），且已被C++14取代

#### C++11 新特性(部分)
##### auto 关键字及用法 
C++11 之前，auto 具有存储期说明符的语义。auto在C++98中的标识临时变量的语义，由于使用极少且多余，在C++11中已被删除。前后两个标准的auto，完全是两个概念。

##### nullptr 关键字及用法
引入nullptr，是因为重载函数处理 NULL 的时候会出问题，二义性
```cpp
void foo(int);   //(1)
void foo(void*); //(2)

foo(NULL);    // 重载决议选择 (1)，但调用者希望是 (2)
foo(nullptr); // 调用(2)
```
##### for循环语法
for ( 范围声明 : 范围表达式 ) 循环语句

##### STL -- `std::array`
std::array 提供了静态数组，编译时确定大小、更轻量、更效率，当然也比 std::vector 有更多局限性。

##### STL -- `std::forward_list`
单向链表

##### STL -- `unordered_map`
##### STL -- `unordered_set`

##### 多线程 -- `std::thread`
在 C++11 以前，C++ 的多线程编程均需依赖系统或第三方接口实现，一定程度上影响了代码的移植性。C++11 中，引入了 boost 库中多线程的部分内容，形成标准后的接口与 boost 库基本没有变化，这样方便了使用者切换使用 C++ 标准接口。

##### 多线程 -- `std::atomic`
从实现上，可以理解为这些原子类型内部自己加了锁。

##### 多线程 -- `std::condition_variable`

##### 智能指针 -- `std::shared_ptr`
##### 智能指针 -- `std::weak_ptr`

##### 其它 -- `std::function`
##### 其它 -- `std::bind`
##### lambda

### C++14
C++14 旨在作为C++11的一个小扩展，主要提供漏洞修复和小的改进。2014年8月18日，经过C++标准委员投票，C++14标准获得一致通过。ISO/IEC 14882:2014

### C++17
C++17 又称C++1z，是继 C++14 之后，C++ 编程语言 ISO/IEC 标准的下一次修订的非正式名称。官方名称 ISO/IEC 14882:2017

基于 C++ 11，C++ 17 旨在简化该语言的日常使用，使开发者可以更简单地编写和维护代码。

> C++ 17是对 C++ 语言的重大更新




