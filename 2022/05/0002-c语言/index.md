# C语言


## C语言
C是一种通用的、命令式的计算机编程语言，支持结构化编程、词法变量作用域和递归，而静态类型系统可以防止许多意想不到的操作。通过设计，C提供了有效地映射到典型机器指令的构造，因此它在以前用汇编语言编写的应用程序中得到了持久的使用，包括操作系统，以及从超级计算机到嵌入式系统的各种应用程序软件。

尽管具有低级功能，但该语言的设计初衷是鼓励跨平台编程。一个符合标准的、可移植编写的C程序可以用于各种各样的计算机平台和操作系统，只需对其源代码进行很少的更改。从嵌入式微控制器到超级计算机，该语言已经在非常广泛的平台上可用。

C语言最初是由Dennis Ritchie于1969年至1973年在贝尔实验室开发的，用于重新实现Unix操作系统。它已经成为有史以来使用最广泛的编程语言之一，来自不同供应商的C编译器可用于大多数现有的计算机架构和操作系统。

### C编译器
- GCC, GNU 编译器
- clang, 是为LLVM项目提供了C语言家族(C, c++, Objective C/c++, OpenCL, CUDA和RenderScript)的编译器
- MSCV, 微软C/C++编译器

[gcc 编译器](https://www3.ntu.edu.sg/home/ehchua/programming/cpp/gcc_make.html)

### 编译器C版本支持
编译器不一定完全支持 C 所有特性，同时编译器也可能通过参数支持一些 C 标准没有的特性，使用时候具体参看 C编译器和对应版本说明。

### 代码风格
[GNU代码风格](https://www.gnu.org/prep/standards/html_node/index.html)

### C版本

| 版本 | 标准                   | 发行时间   |
| ---- | ---------------------- | ---------- |
| K&E  | 无                     | 1978-02-22 |
| C89  | ANSI X3.159-1989       | 1989-12-14 |
| C90  | ISO/IEC 9899:1990      | 1990-12-20 |
| C95  | ISO/IEC 9899/AMD1:1995 | 1995-03-30 |
| C99  | ISO/IEC 9899:1999      | 1999-12-16 |
| C11  | ISO/IEC 9899:2011      | 2011-12-15 |

### hello world

```c
#include <stdio.h>

int main(void)
{
    puts("Hello, World");
    return 0;
}
```
编译
```shell
gcc hello.c -o hello

// 我们还可以使用警告选项-Wall -Wextra -Werror，这有助于识别可能导致程序失败或产生意外结果的问题
gcc -Wall -Wextra -Werror -o hello hello.c 
```

对比 K&R C 的Hello world
```c
#include <stdio.h>

main()
{
    printf("hello, world\n");
}
```
> 注意，在编写《The C programming language》的第一版(1978年)时，C编程语言还没有标准化，而且这个程序可能无法在大多数现代编译器上编译，除非它们被指示接受C90代码。

K&R书中的第一个示例现在被认为质量很差，部分原因是它缺少main()的显式返回类型，部分原因是它缺少返回语句。

在C89中，main的类型默认为int，但K&R示例不向环境返回定义的值。在C99和以后的标准中，返回类型是必需的，但是省略main的返回语句(而且只有main)是安全的，因为C99 5.1.2.2.3引入了一个特殊情况—它相当于返回0，表示成功。

## 字符类型和转换

从流中读取字符类型
```c
#include <ctype.h>
#include <stdio.h>

typedef struct 
{
    size_t space;
    size_t alnum;
    size_t punct;
} chartypes;

chartypes classify(FILE *f) 
{
    chartypes types = { 0, 0, 0 };
    int ch;

    while ((ch = fgetc(f)) != EOF) {
        types.space += !!isspace(ch);
        types.alnum += !!isalnum(ch);
        types.punct += !!ispunct(ch);
    }

    return types;
}
```
`classify`函数从流中读取字符并统计空格、字母数字和标点符号的数量。它需要注意以下几个问题：
- 当从流中读取一个字符时，结果被保存为int，因为读取EOF(文件结束标记)和具有相同位模式的字符之间会有歧义。
- `classify`函数希望它们的参数可以表示为unsigned char，或者EOF宏的值。因为这正是fgetc返回的，所以这里不需要转换。
- 字符分类函数的返回值只区分零(false)和非零(true)。为了计算字符分类出现的次数，需要将该值转换为1或0，这可以通过双重否定实现 

### C标准库 -- `ctype.h` 

```c
int c = 'A';

isalpha(c);  /* Checks if c is alphabetic (A-Z, a-z), returns non-zero here. */
isalnum(c);  /* Checks if c is alphanumeric (A-Z, a-z, 0-9), returns non-zero here. */
iscntrl(c);  /* Checks is c is a control character (0x00-0x1F, 0x7F), returns zero here. */
isdigit(c);  /* Checks if c is a digit (0-9), returns zero here. */
isgraph(c);  /* Checks if c has a graphical representation (any printing character except space), returns non-zero here. */
islower(c);  /* Checks if c is a lower-case letter (a-z), returns zero here. */
isprint(c);  /* Checks if c is any printable character (including space), returns non-zero here. */
ispunct(c);  /* Checks if c is a punctuation character, returns zero here. */
isspace(c);  /* Checks if c is a white-space character, returns zero here. */
isupper(c);  /* Checks if c is an upper-case letter (A-Z), returns non-zero here. */
isxdigit(c); /* Checks if c is a hexadecimal digit (A-F, a-f, 0-9), returns non-zero here.*/

// C99
isblank(c); /* Checks if c is a blank character (space or tab), returns non-zero here. */
```
## 别名和有效类型

> 这里的`别名`是英文直译，可以理解为我们常见的`类型转换`

- 别名是指向同一个对象的两个指针a和b的属性，即a == b。
- C使用数据对象的有效类型来确定可以对该对象执行哪些操作。具体来说，有效类型用于确定两个指针是否可以相互别名。

C语言的严格别名规则是指编译器可能假定哪些对象会(或不会)别名。对于数据指针，你应该记住两条经验法则:
- 除非另有说明，具有相同基类型的两个指针可以别名。
- 两个具有不同基类型的指针不能别名，除非两个类型中至少有一个是字符类型。

> 这里的基本类型指的是抛开类型限制，比如const，比如：`double* a` 和 `const double* b`，编译器通常必须假设对`*a`的修改可能会改变`*b`。

违反第二条规则会有灾难性后果，所以为了减少问题发生概率，除非源或目标类型为void，否则具有不同基类型的指针之间的所有指针转换都必须是显式的。
> 这里必须做的类型转换不包括添加限定符 `const`

### 例子
#### 不能通过非字符类型访问字符类型
```c
int main( void )
{
    char a[100];
    int* b = ( int* )&a;
    *b = 1;

    static char c[100];
    b = ( int* )&c;
    *b = 2;

    _Thread_local char d[100];
    b = ( int* )&d;
    *b = 3;
}
```
在例子中，一个char数组被重新解释为int类型，并且每次对int指针b解引用时，该行为都是未定义的。因为它违反了“有效类型”规则，具有有效类型的数据对象不能通过非字符类型的其他类型访问。因为这里的另一个类型是int，所以这是不允许的。

即使内存对齐和指针大小是已知的，这也不能免除此规则，行为仍然是未定义的。

这特别意味着，在标准C语言中不可能保留可以通过不同类型的指针使用的字符类型的缓冲区对象，因为您将使用由malloc或类似函数接收的缓冲区。

实现上述目标的正确方法是使用 `union`
```c
typedef union bufType bufType;
union bufType 
{
    char c[sizeof(int[25])];
    int i[25];
};

int main( void )
{
    bufType a = { .c = { 0 } }; // reserve a buffer and initialize
    int* b = a.i; // no cast necessary
    *b = 1;

    static bufType a = { .c = { 0 } };
    int* b = a.i;
    *b = 2;

    _Thread_local bufType a = { .c = { 0 } };
    int* b = a.i;
    *b = 3;
}
```
这里，union确保编译器从一开始就知道缓冲区可以通过不同的方式访问。这样做的另一个好处是，现在缓冲区有一个a.i，它已经是int类型，不需要进行指针转换。

#### 有效类型
数据对象的有效类型是与之关联的最后一个类型信息(如果有的话)。
```c
// 有效类型是 uint32_t
uint32_t a = 0.0;

// *pa的有效类型是 uint32_t
uint32_t* pa = &a;

// 目前 *q 并不是一个有效类型
void* q = malloc(sizeof uint32_t);

// 这里的 q 仍然不是一个有效类型，因为还没有数据写入
uint32_t* qb = q;

// *qb 目前是一个有效类型 uint32_t，因为一个 uint32_t 值被写入了
*qb = 37;

// r 指向的对象尽管已经初始化了，但还不是有效类型
void* r = calloc(1, sizeof uint32_t);

// 不是有效类型
uint32_t* rc = r;

// *rc 已经是一个有效类型 uint32_t 因为一个值从它那读取了
// 因为使用 calloc 初始化了，所以读操作是合理的
// 此时尽管我们没有写它的值，但是 *rc 已经是一个有效类型了
uint32_t c = *rc;

void* s = malloc(sizeof uint32_t);
// 因为有 uint32_t 类型复制到指定对象里，所以 *s 已经是有效类型了
memcpy(s, r, sizeof uint32_t);
```
#### 严重违反类型转换规则

在下面的代码中，让我们假设`float`和`uint32_t`有相同的内存大小
```c
void fun(uint32_t* u, float* f) 
{
    float a = *f, *u = 22;
    float b = *f;
    print("%g should equal %g\n", a, b);
}
```
u 和 f 具有不同的基类型，因此编译器可以假定它们指向不同的对象。

在a和b的两次初始化之间`*f`不可能发生变化，因此编译器可能会优化代码，使其等价于:
```c
void fun(uint32_t* u, float* f) 
{
    float a = *f, *u = 22;

    print("%g should equal %g\n", a, a);
}
```
即`*f`的二次加载操作可以完全优化出来。
如果这样调用:
```c
float fval = 4;
uint32_t uval = 77;
fun(&uval, &fval);

// 输出:
4 should equal 4
```

但是如果这样调用：
```c
float fval = 4;
uint32_t* up = (uint32_t*)&fval;
fun(up, &fval);
```
我们违反了严格的混叠规则。然后这种行为就变得没有定义了。如果编译器优化了第二次访问，输出可能与上面一样，或者完全不同，从而使程序最终处于完全不可靠的状态。

#### 限制条件
如果有两个相同类型的指针实参，编译器就不能做任何假设，必须总是假设对`*e`的修改可能会改变`*f`:
```c
void fun(float* e, float* f) 
{
    float a = *f *e = 22;
    float b = *f;
    print("is %g equal to %g?\n", a, b);
}
float fval = 4;
float eval = 77;
fun(&eval, &fval);
```
正常输出是 `is 4 equal to 4?`

如果我们通过一些外部信息知道e和f永远不会指向相同的数据对象，那么这可能是低效的。我们可以通过给指针形参添加限制限定符来反映这一点:
```c
void fan(float*restrict e, float*restrict f) 
{
    float a = *f *e = 22;
    float b = *f;
    print("is %g equal to %g?\n", a, b);
}
```
那么编译器可能总是假设e和f指向不同的对象。

#### 改变字节
一旦对象具有有效类型，就不应该试图通过其他类型的指针修改它，除非其他类型是字符类型、char、signed char或 unsigned char。
```c
#include <inttypes.h>
#include <stdio.h>

int main(void) 
{
    uint32_t a = 57;
    // conversion from incompatible types needs a cast !
    unsigned char* ap = (unsigned char*)&a;

    for (size_t i = 0; i < sizeof a; ++i) {
        /* set each byte of a to 42 */
        ap[i] = 42;
    }
    printf("a now has value %" PRIu32 "\n", a);
}
```
打印结果: `a now has value 707406378`
正常是因为：
- 使用unsigned char类型访问单个字节，因此每次修改都有很好的定义。
- 对象的两个视图，通过别名a和`*ap`，但由于ap是指向字符类型的指针，因此不适用严格的别名规则。因此，编译器必须假设a的值可能在for循环中被更改了。a的修改值必须从已更改的字节构造。
- a的类型的`uint32_t`没有填充位。其表示的所有位都为值计数，这里是707406378，并且不能有陷阱表示。

## 数组
数组是派生数据类型，表示另一种类型的值(“元素”)的有序集合。C语言中的大多数数组都有固定数量的任意一种类型的元素，其表示形式是连续地将元素存储在内存中，没有间隔或填充。C允许多维数组，它的元素是其他数组，也可以是指针数组。

C支持动态分配数组，其大小在运行时确定。C99及以后版本支持可变长度数组或VLAs。

### 语法
- type name[length];
- int arr[10] = {0}; // 初始化所有元素为0
- int arr[10] = {42}; // 初始化第一个元素为42，其它为0
- int arr[] = {4, 2, 3, 1}; 
- arr[n] = value;
- value = arr[n];

### 为什么需要数组
数组提供了一种将对象组织成具有自身意义的聚合的方法。例如，C字符串是字符数组(chars)，而像“Hello, World!”这样的字符串具有聚合的意义，它不是单个字符固有的。类似地，数组通常用于表示数学向量和矩阵，以及多种类型的列表。此外，如果没有对元素进行分组的方法，就需要单独处理每个元素，比如通过单独的变量。它不仅笨重，而且不能方便地容纳不同长度的集合。

### 大多数上下文数组隐式转为指针
除了 `sizeof` 操作，使用 `_Alignof` 操作、`&`操作、迭代字符串，它都会转换为第一个元素首地址。

这个隐式转换与数组下标操作符`([])`的定义紧密耦合:表达式`arr[idx]`被定义为等价于`*(arr + idx)`。此外，由于指针算法是可交换的，`*(arr + idx)`也等价于`*(idx + arr)`，而`*(idx + arr)`又等价于`idx[arr]`。如果`idx`或`arr`是指针(或数组，可衰变为指针)，另一个是整数，且整数是指针所指向数组的有效索引，则所有这些表达式都是有效的，计算结果相同。

作为一个特例，观察`&(arr[0])`等价于`&*(arr + 0)`，简化为`arr`。所有的这些表达式都是可互换的(`attr`最衰减到指针)。这只是再次表示数组衰变为指向其第一个元素的指针。

相反，如果地址操作符应用于类型为`T[N]`的数组(即`&arr`)，则结果类型为`T (*)[N]`并指向整个数组。这与指向数组第一个元素的指针不同，至少在指针算术方面是不同的，指针是根据指向类型的大小定义的。

### 函数参数不是数组
```c
void foo(int a[], int n);
void foo(int *a, int n);
```
尽管foo的第一个声明对参数a使用了类似数组的语法，但这种语法用于声明一个函数参数，将该参数声明为指向数组元素类型的指针。因此，foo()的第二个签名在语义上与第一个签名相同。这对应于数组值的衰减指针出现作为一个函数调用的参数,这样,如果一个变量和一个函数参数声明相同的数组类型,变量的值是适用于一个函数调用的参数与参数有关。

### 特殊例子
特殊的初始化：
```c
int array[5] = {[2] = 5, [1] = 2, [4] = 9}; /* array is {0, 2, 5, 0, 9} */
int array[] = {[3] = 8, [0] = 9}; /* size is 4 */
```
> 不允许声明零长度的数组

### 变长数组(C99/C11)
可变长度数组(简称VLA)是在C99中添加的，在C11中是可选的。它们与普通数组相同，但有一个重要的区别:在编译时不必知道长度。VLA具有自动存储期限。
```c
size_t m = calc_length(); /* calculate array length at runtime */
int vla[m]; /* create array with calculated length */
```
> 重要: VLA有潜在的危险。如果上面示例中的数组vla需要的堆栈空间大于可用空间，则堆栈将溢出。因此不建议使用VLA

### 清除数组内容
有时需要在初始化完成后将数组设置为零。
```c
#include <stdlib.h> /* for EXIT_SUCCESS */

#define ARRLEN (10)

int main(void)
{
    int array[ARRLEN]; /* Allocated but not initialised, as not defined static or global. */
    
    size_t i;
    for(i = 0; i < ARRLEN; ++i) {
        array[i] = 0;
    }

    return EXIT_SUCCESS;
}
```
或者使用 `memset`

### 数组长度
```c
sizeof(array) / sizeof(array[0])
```
### 设置数组值
```c
array[4] = 5;
4[array] = 5;
*(4 + array) = 5;

// 获取值
val = 4[array];
```

### 定义数组并访问元素
### 堆上动态分配空间并初始化数组
### 迭代数组
### 多维数组
```
type name[size1][size2]...[sizeN];

int arr[5][10][4];

// 初始化 3 维数组
double cprogram[3][2][4]={
{{-0.1, 0.22, 0.3, 4.3}, {2.3, 4.7, -0.9, 2}},
{{0.9, 3.6, 4.5, 4}, {1.2, 2.4, 0.22, -1}},
{{8.2, 3.12, 34.2, 0.1}, {2.1, 3.2, 4.3, -2.0}}
};
```

## 断言
在软件遇到断言时，提出的条件必须为真。最常见的是在执行时进行验证的简单断言。静态断言是在编译时检查的。

### 语法
- `assert(expression);`
- `static_assert(expression, message);`
- `_Static_assert(expression, message);`

### 参数

| 参数       | 描述                         |
| ---------- | ---------------------------- |
| expression | 表达式，判断条件是否满足     |
| message    | 诊断消息中包含的字符串字面值 |

`assert` 和 `static_assert` 都在 `assert.h` 中定义。

assert的定义依赖于宏NDEBUG，该宏没有被标准库定义。如果定义了NDEBUG，则assert为无操作:
```c
#ifdef NDEBUG
# define assert(condition) ((void) 0)
#else
# define assert(condition) /* implementation defined */
#endif
```
> 使用 assert 会导致程序退出，生产环境中不要用 assert，最好使用条件判断

`static_assert`扩展为`_Static_assert`，这是一个关键字。`condition` 在编译时被检查，因此`condition`必须是一个常量表达式。没有必要在开发和生产之间以不同的方式处理这个问题

### 例子
#### 前置条件和后置条件
断言的一个用例是前置条件和后置条件。这对于保持不变和契约式设计非常有用。例如，长度总是0或正的，因此该函数必须返回0或正的值。
```c
int length2 (int *a, int count)
{
    int i, result = 0;

    /* Precondition: */
    /* NULL is an invalid vector */
    assert (a != NULL);

    /* Number of dimensions can not be negative.*/
    assert (count >= 0);

    /* Calculation */
    for (i = 0; i < count; ++i) {
        result = result + (a[i] * a[i]);
    }

    /* Postcondition: */
    /* Resulting length can not be negative. */
    assert (result >= 0);

    return result;
}
```

#### 简单断言
```c
#include <stdio.h>
/* Uncomment to disable `assert()` */
/* #define NDEBUG */
#include <assert.h>
int main(void)
{
    int x = -1;
    assert(x >= 0);
    printf("x = %d\n", x);

    return 0;
}
```
运行输出:
```
a.out: main.c:9: main: Assertion `x >= 0' failed.
```

#### 静态断言(C11)
静态断言用于在编译代码时检查条件是否为真。如果不是，则需要编译器发出错误消息并停止编译过程。

```c
#include <assert.h>
enum {N = 5};
_Static_assert(N == 5, "N does not equal 5");
static_assert(N > 10, "N is not greater than 10"); /* compiler error */
```

#### 静态断言(c99)
在`C11`之前，没有对静态断言的直接支持。但是，在`C99`中，可以使用宏模拟静态断言，如果编译时条件为假，则会触发编译失败。与`_Static_assert`不同，第二个参数需要是适当名称，以便可以使用它创建变量名称。如果断言失败，则在编译器错误中可以看到变量名，因为该变量是在语法不正确的数组声明中使用的

```c
#define STATIC_MSG(msg, l) STATIC_MSG2(msg, l)
#define STATIC_MSG2(msg,l) on_line_##l##__##msg
#define STATIC_ASSERT(x, msg) extern char STATIC_MSG(msg, __LINE__) [(x)?1:-1]

enum { N = 5 };
STATIC_ASSERT(N == 5, N_must_equal_5);
STATIC_ASSERT(N > 5, N_must_be_greater_than_5); /* compile error */
```
在C99之前，不能在块中的任意位置声明变量，因此在使用这个宏时必须非常谨慎，确保它只出现在变量声明有效的地方

#### 断言不可能到达的位置(switch-case语法)

## 原子(Atomics)
Atomics作为C语言的一部分是自C11以来可用的可选特性。它们的目的是确保对在不同线程之间共享的变量的无竞争访问。如果没有原子限定，如果两个线程并发访问一个共享变量，那么它的状态将是未定义的。例如，一个自增操作(++)可以被拆分为几个汇编指令、一个读取指令、加法本身和一个存储指令。如果另一个线程要执行相同的操作，它们的两个指令序列可能会交织在一起，导致不一致的结果。
- Types: 除数组类型外，所有对象类型都可以限定为 `_Atomic`
- Operators: 它们上的所有读-修改-写操作符都保证是原子的
- Operations: 还有其他一些被指定为类型泛型函数的操作，例如`atomic_compare_exchange`。
- Threads: 当它们被不同的线程访问时，对它们的访问保证不会产生数据竞争。
- Signal handlers: 如果对原子类型的所有操作都是无状态的，则称为无锁类型。在这种情况下，它们还可以用于处理正常控制流和信号处理程序之间的状态变化。
- 只有一种数据类型保证是无锁的:`atomic_flag`。这是一种最小类型，其操作旨在映射到有效的测试和设置硬件指令。

在C11的线程接口中还有其他避免竞争条件的方法，特别是一个互斥类型`mtx_t`，它可以相互排除线程访问关键数据或代码的关键部分。如果原子不可用，则必须使用它们来防止竞争。

### 语法

```c
#ifdef __STDC_NO_ATOMICS__
# error this implementation needs atomics•
#endif•
#include <stdatomic.h>

unsigned _Atomic counter = ATOMIC_VAR_INIT(0);
```

### 例子
#### 原子和操作
可以在不同的线程之间并发地访问原子变量，而不需要创建竞争条件。
```c
static unsigned _Atomic active = ATOMIC_VAR_INIT(0);

int myThread(void* a) 
{
    ++active; // increment active race free

    // do something

    --active; // decrement active race free

    return 0;
}
```
基类型允许的所有左值操作(修改对象的操作)都是允许的，不会导致访问它们的不同线程之间的竞争条件。
- 对原子对象的操作通常比普通的算术操作慢几个数量级。这也包括简单的加载或存储操作。所以你应该只在关键的任务中使用它们。
- 通常的算术运算和赋值，如a = a+1;实际上在a上有三个操作:首先是加载，然后是加法，最后是存储。只有操作a += 1;和++;是这样的。

## 位域
C语言中大多数变量的大小都是字节的整数。位域是结构的一部分，不一定占用整数字节;它们可以使用任意数量的比特。多个位域可以被打包到一个存储单元中。它们是标准C的一部分，但有许多方面是实现定义的。它们是C语言中最不可移植的部分之一。

### 语法
```c
type-specifier identifier : size;
```
### 参数

| 参数           | 描述                                   |
| -------------- | ---------------------------------------|
| type-specifier | `signed`、`unsigned`、`int` 或 `_Bool` |
| identifier     | 名字                                   |
| size           | 字段中使用的位数                       |

> 位字段的可移植类型只有`signed`、`unsigned`或`_Bool`。也可以使用纯`int`类型，但标准规定对于位域，说明符int是否指定与signed int相同的类型或与unsigned int相同的类型由实现定义。特定的实现可能允许其他整数类型，但使用它们是不可移植的。

### 例子
#### 位域
一个简单的位域可以用来描述可能包含特定位数的事物。
```c
struct encoderPosition 
{
    unsigned int encoderCounts : 23;
    unsigned int encoderTurns : 4;
    unsigned int _reserved : 5;
};
```
在这个例子中，我们考虑一个编码器，它有23位单精度，4位描述多匝。当与输出与特定位数相关联的数据的硬件连接时，经常使用位域。另一个例子可能是与FPGA通信，FPGA以32位段的形式将数据写入内存，允许硬件读取:
```c
struct FPGAInfo 
{
    union {
        struct bits {
            unsigned int bulb1On : 1;
            unsigned int bulb2On : 1;
            unsigned int bulb1Off : 1;
            unsigned int bulb2Off : 1;
            unsigned int jetOn : 1;
        };
    unsigned int data;
    };
};
```
对于这个示例，我们展示了一个常用的构造，它能够访问单个位的数据，或者将数据包作为一个整体写入(模拟FPGA可能做的事情)。然后我们可以像这样访问比特:
```
FPGAInfo fInfo;
fInfo.data = 0xFF34F;

if (fInfo.bits.bulb1On) {
    printf("Bulb 1 is on\n");
}
```
这是有效的，但根据C99标准6.7.2.1，第10项:单位内位域的分配顺序(高阶到低阶或低阶到高阶)是由实现定义的

在以这种方式定义位域时，需要注意字节序。因此，可能需要使用一个预处理器指令来检查机器的字节顺序。下面是一个例子:
```c
typedef union {
    struct bits {
#if defined(WIN32) || defined(LITTLE_ENDIAN)
        uint8_t commFailure :1;
        uint8_t hardwareFailure :1;
        uint8_t _reserved :6;
#else
        uint8_t _reserved :6;
        uint8_t hardwareFailure :1;
        uint8_t commFailure :1;
#endif
    };
    uint8_t data;
} hardwareStatus;
```

#### 使用位域作为小整数
```c
#include <stdio.h>
int main(void)
{
    /* define a small bit-field that can hold values from 0 .. 7 */
    struct {
        unsigned int uint3: 3;
    } small;

    /* extract the right 3 bits from a value */
    unsigned int value = 255 - 2; /* Binary 11111101 */
    small.uint3 = value; /* Binary 101 */
    printf("%d", small.uint3);

    /* This is in effect an infinite loop */
    for (small.uint3 = 0; small.uint3 < 8; small.uint3++) {
        printf("%d\n", small.uint3);
    }

    return 0;
}
```

#### 位域对齐
位域提供了声明小于字符宽度的结构域的能力。位域通过字节级或字级掩码实现。下面的示例产生一个8字节的结构。
```c
struct C
{
short s; /* 2 bytes */
char c; /* 1 byte */
int bit1 : 1; /* 1 bit */
int nib : 4; /* 4 bits padded up to boundary of 8 bits. Thus 3 bits are padded */
int sept : 7; /* 7 Bits septet, padded up to boundary of 32 bits. */
};
```
注释描述了一种可能的布局，但由于标准说可寻址存储单元的对齐方式未指定，所以也可能有其他布局。

未命名位域可以是任何大小，但不能初始化或引用它们。

不能给零宽度位域指定名称并将下一个字段与位域的数据类型定义的边界对齐。这是通过在位域之间填充位来实现的。

`struct A`的大小是1字节。
```c
struct A
{
    unsigned char c1 : 3;
    unsigned char c2 : 4;
    unsigned char c3 : 1;
};
```

在`struct B`中，第一个未命名位域跳过2位;c2之后的零宽度位域导致c3从字符边界开始(因此在c2和c3之间跳过3位。c4之后有3个填充位。因此，该结构的大小为2字节。
```c
struct B
{
    unsigned char c1 : 1;
    unsigned char : 2; /* Skips 2 bits in the layout */
    unsigned char c2 : 2;
    unsigned char : 0; /* Causes padding up to next container boundary */
    unsigned char c3 : 4;
    unsigned char c4 : 1;
};
```

#### 什么时候使用位域
位域用于将多个变量组合成一个对象，类似于结构。这允许减少内存使用，在嵌入式环境中尤其有用

## Boolean
要使用预定义的类型`_Bool`和头文件`<stdbool.h>`，必须使用`C99/C11`版本的C语言。为了避免编译器警告和可能的错误，只有在使用C89和该语言以前的版本时，才应该使用typedef/define示例。

### 例子

## 命令行参数
```c
int main(int argc, char *argv[])
```
### 例子

## 注释

```c
/* ... */

// c99 及其之后
// ... 
```

## 常见的C编程习惯用法和开发人员实践

### 比较
```c
// 不好
if ( i == 2) //Bad-way
{
    doSomething;
}

// 好
if( 2 == i) //Good-way
{
    doSomething;
}
```

## 常见陷阱

## 编译

## 复合类型
像字符串字面量一样，const限定的复合字面量可以放在只读内存中，甚至可以共享。例如,
```c
(const char []){"abc"} == "abc"
```
### 语法
```c
(type){ initializer-list }
```

## 头文件包含

## 数据类型

## 声明与定义

## 错误处理
### 语法
```c
#include <errno.h>•
int errno; /* implementation defined */•

#include <string.h>•
char *strerror(int errnum);•

#include <stdio.h>•
void perror(const char *s);
```

## 文件和 I/O 流
### 语法
```c
#include <stdio.h> /* Include this to use any of the following sections */•
FILE *fopen(const char *path, const char *mode); /* Open a stream on the file at path with
the specified mode */

FILE *freopen(const char *path, const char *mode, FILE *stream); /* Re-open an existing
stream on the file at path with the specified mode */

int fclose(FILE *stream); /* Close an opened stream */•
size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream); /* Read at most nmemb
elements of size bytes each from the stream and write them in ptr. Returns the number of
read elements. */

size_t fwrite(const void *ptr, size_t size, size_t nmemb, FILE *stream); /* Write nmemb
elements of size bytes each from ptr to the stream. Returns the number of written elements.
*/

int fseek(FILE *stream, long offset, int whence); /* Set the cursor of the stream to offset,
relative to the offset told by whence, and returns 0 if it succeeded. */

long ftell(FILE *stream); /* Return the offset of the current cursor position from the beginning
of the stream. */

void rewind(FILE *stream); /* Set the cursor position to the beginning of the file. */•
int fprintf(FILE *fout, const char *fmt, ...); /* Writes printf format string on fout */•
FILE *stdin; /* Standard input stream */•
FILE *stdout; /* Standard output stream */•
FILE *stderr; /* Standard error stream */
```

## 格式化输入/输出

## 函数参数

## 函数指针
### 语法
```c
- `returnType (*name)(parameters);`
- `typedef returnType (*name)(parameters);`
- `typedef returnType Name(parameters);`
    Name *name;
- `typedef returnType Name(parameters);`
  `typedef Name *NamePtr;`
```

## 使用 typedef

## 上下文指针

## 标识符作用域

## 实现定义的行为

## 隐式转换和显示转换

## 初始化

## 内联汇编

## 内联

## 进程间通信(IPC)

## 迭代语句/循环

### 语法
```c
/* all versions */•
for ([expression]; [expression]; [expression]) one_statement•
for ([expression]; [expression]; [expression]) { zero or several statements }•
while (expression) one_statement•
while (expression) { zero or several statements }•
do one_statement while (expression);•
do { one or more statements } while (expression);•

// since C99 in addition to the form above•
for (declaration; [expression]; [expression]) one_statement;•
for (declaration; [expression]; [expression]) { zero or several statements }
```

## 跳转
### 语法
```c
return val; /* Returns from the current function. val can be a value of any type that is converts to the function's return type. */

return; /* Returns from the current void-function. */•
break; /* Unconditionally jumps beyond the end ("breaks out") of an Iteration Statement (loop) or out of the innermost switch statement. */

continue; /* Unconditionally jumps to the beginning of an Iteration Statement (loop). */•
goto LBL; /* Jumps to label LBL. */•
LBL: statement /* any statement in the same function. */
```

## 链表

## 数字、字符和字符串的字面值

## 内存管理
### 语法
```c
void *aligned_alloc(size_t alignment, size_t size); /* Only since C11 */•
void *calloc(size_t nelements, size_t size);•
void free(void *ptr);•
void *malloc(size_t size);•
void *realloc(void *ptr, size_t size);•
void *alloca(size_t size); /* from alloca.h, not standard, not portable, dangerous. */
```

## 多字字符序列

## 多线程
### 语法
```c
thrd_t // Implementation-defined complete object type identifying a thread•
int thrd_create( thrd_t *thr, thrd_start_t func, void *arg ); // Creates a thread•
int thrd_equal( thrd_t thr0, thrd_t thr1 ); // Check if arguments refer to the same thread•
thr_t thrd_current(void); // Returns identifier of the thread that calls it•
int thrd_sleep( const struct timespec *duration, struct timespec *remaining ); // Suspend call
thread execution for at least a given time

void thrd_yield(void); // Permit other threads to run instead of the thread that calls it•
_Noreturn void thrd_exit( int res ); // Terminates the thread the thread that calls it•
int thrd_detatch( thrd_t thr; // Detaches a given thread from the current environment•
int thrd_join( thrd_t thr, int *res ); // Blocks the current thread until the given thread finishes
```

## 操作符
### 语法
```c
expr1 operator•
operator expr2•
expr1 operator expr2•
expr1 ? expr2 : expr3
```

## 将 2 维数组传给函数

## 指针
### 语法
```c
<Data type> *<Variable name>;•
int *ptrToInt;•
void *ptrToVoid; /* C89+ */•
struct someStruct *ptrToStruct;•
int **ptrToPtrToInt;•
int arr[length]; int *ptrToFirstElem = arr; /* For <C99 'length' needs to be a compile time constant, for >=C11 it might need to be one. */

int *arrayOfPtrsToInt[length]; /* For <C99 'length' needs to be a compile time constant, for >=C11 it might need to be one. */
```

## 预处理器和宏
## 随机数生成
### 语法
```c
arc4random() (available on OS X and BSD)•
random() (available on Linux)•
drand48() (available on POSIX)
```

## 条件

## 信号处理

## 标准数学

## 字符串

## 结构体

## Structure Padding and Packing

## 测试框架
### CppUTest

```c
#include <CppUTest/CommandLineTestRunner.h>
#include <CppUTest/TestHarness.h>
TEST_GROUP(Foo_Group) {}
TEST(Foo_Group, Foo_TestOne) {}
/* Test runner may be provided options, such
```

## 线程(原生)
### 语法
```c
#ifndef __STDC_NO_THREADS__•
# include <threads.h>•
#endif•
void call_once(once_flag *flag, void (*func)(void));•
int cnd_broadcast(cnd_t *cond);•
void cnd_destroy(cnd_t *cond);•
int cnd_init(cnd_t *cond);•
int cnd_signal(cnd_t *cond);•
int cnd_timedwait(cnd_t *restrict cond, mtx_t *restrict mtx, const struct timespec *restrict ts);

int cnd_wait(cnd_t *cond, mtx_t *mtx);•
void mtx_destroy(mtx_t *mtx);•
int mtx_init(mtx_t *mtx, int type);•
int mtx_lock(mtx_t *mtx);•
int mtx_timedlock(mtx_t *restrict mtx, const struct timespec *restrict ts);•
int mtx_trylock(mtx_t *mtx);•
int mtx_unlock(mtx_t *mtx);•
int thrd_create(thrd_t *thr, thrd_start_t func, void *arg);•
thrd_t thrd_current(void);•
int thrd_detach(thrd_t thr);•
int thrd_equal(thrd_t thr0, thrd_t thr1);•
_Noreturn void thrd_exit(int res);•
int thrd_join(thrd_t thr, int *res);•
int thrd_sleep(const struct timespec *duration, struct timespec* remaining);•
void thrd_yield(void);•
int tss_create(tss_t *key, tss_dtor_t dtor);•
void tss_delete(tss_t key);•
void *tss_get(tss_t key);•
int tss_set(tss_t key, void *val);
```

## 类型限定符

## typedef

## 未定义类型

## 联合体

## Valgrind
Valgrind是一个调试工具，可用于诊断C程序中有关内存管理的错误。Valgrind可用于检测诸如无效指针使用等错误，包括超出已分配空间的写入或读取，或对free()进行无效调用。它还可以通过执行内存分析的函数来改进应用程序。

## 变量参数

## X-macros
x -宏是一种基于预处理程序的技术，用于最小化重复代码和维护数据/代码的对应关系。基于一组公共数据的多个不同的宏展开可以通过单个主宏来表示整个组展开，该宏的替换文本由内部宏展开的序列组成，每个数据对应一个。内部宏通常被命名为X()，这就是该技术的名称。

