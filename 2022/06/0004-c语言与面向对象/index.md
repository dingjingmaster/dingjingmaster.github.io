# C语言与面向对象


## 为什么要使用面向对象

首先需要澄清一点，面向对象是一种思想，它的主要作用为了改善程序结构、降低各部分耦合、降低代码维护成本的。

而接下来通过一些例子来介绍如何使用 C 实现面向对象编程。

## C的模块化和面向对象

以定义栈为例

### C 与 模块化
传统写法
```c
// stack.h
#ifndef _STACK_H_
#define _STACK_H_

#ifdef __cplusplus
extern "C" {
#endif

bool push(int val);
bool pop(int *pRet);

#ifdef __cplusplus
}
#endif

#endif
```

```c
// stack.c
#include <stdbool.h>
#include "stack.h"

int buf[16];
int top = 0;

bool isStackFull(void) {
  return top == sizeof(buf) / sizeof(int);
}

bool isStackEmpty(void) {
  return top == 0;
}

// true: 成功, false: 失敗
bool push(int val) {
    if (isStackFull()) return false;
    buf[top++] = val;
    return true;
}

// true: 成功, false: 失敗
bool pop(int *pRet) {
    if (isStackEmpty()) return false;
    *pRet = buf[--top];
    return true;
}
```

如果程序很小，这样编写就足够了。但是这种代码一般会暴露一些问题，比如：变量、函数的作用域。查看 `stack.h` 会发现`push()`和`pop()`都是提供给外部函数调用的函数。但是在 `stack.c` 中，`buf`、`top`、`isStackEmpty()`、`isStackFull()` 都被公开在全局命名空间中。如果其他代码块中使用了相同名字的变量和函数就会发生冲突，导致链接错误。

为了解决这个问题，我们可以对变量、函数用 `static` 修饰

```c
#include <stdbool.h>

static int buf[16];
static int top = 0;

static bool isStackFull(void) 
{
  return top == sizeof(buf) / sizeof(int);
}
```

> 像这样在头文件中声明对外公开的函数和变量，对其它无需对外公开的函数/变量指定`static`修饰符修饰，使函数和变量只在该编译单位内有效是C开发中常用的模块化方法。

> 这里有个问题，上述代码中栈相关全局变量定义在 `.c` 文件中是有一定道理的，如果放在 `.h` 中一来可能会与其他代码中全局变量冲突；二来暴露了栈的数据结构，使用者可以不必使用提供的API而直接访问全局变量，这会破坏栈结构。
> <br/>同理：为什么不能提供栈的创建函数也是同样道理。

### 使用结构体将数据与代码块分离
针对上述`stack`，只能是一个栈，如果多个栈，则可以使用结构体做封装。
```c
// stack.h
#ifndef _STACK_H_
#define _STACK_H_

#include <stddef.h>

#ifdef __cplusplus
extern "C" {
#endif

// 此处要实现封装，可以把定义写到 .c 中
typedef struct {
    int top;
    const size_t size;
    int * const pBuf;
} Stack;

bool push(Stack *p, int val);
bool pop(Stack *p, int *pRet);

// 此宏方便将结构体初始化
#define newStack(buf) {                 \
    0, sizeof(buf) / sizeof(int), (buf) \
}

#ifdef __cplusplus
}
#endif
#endif
```

```c
#include <stdbool.h>
#include "stack.h"

static bool isStackFull(const Stack *p) 
{
    return p->top == p->size;
}

static bool isStackEmpty(const Stack *p)
{
    return p->top == 0;
}

// true: 成功, false: 失敗
bool push(Stack *p, int val) 
{
    if (isStackFull(p)) return false;
    p->pBuf[p->top++] = val;
    return true;
}

// true: 成功, false: 失敗
bool pop(Stack *p, int *pRet) 
{
    if (isStackEmpty(p)) return false;
    *pRet = p->pBuf[--p->top];
    return true;
}

```

### 使用c进行面向对象编程
#### 带检查功能的栈
比如栈中存放数据的值有范围限制，范围以外的值不能放入，可以使用如下函数检查：
```c
bool pushWithRangeCheck (Stack* p, int val, int min, int max);
```

但是这种方法传递参数次数太多，而且每次调用 `push()` 函数都要传递范围参数，显然是不合理的。

再次改进后:
```c
#include <stddef.h>

#ifdef __cplusplus
extern "C" {
#endif

typedef struct {
    int top;
    const size_t size;
    int * const pBuf;

    const bool needRangeCheck;
    const int min;
    const int max;
} Stack;

bool push(Stack *p, int val);
bool pop(Stack *p, int *pRet);

#define newStack(buf) {                    \
    0, sizeof(buf) / sizeof(int), (buf),   \
    false, 0, 0                            \
}

#define newStackWithRangeCheck(buf, min, max) { \
    0, sizeof(buf) / sizeof(int), (buf),        \
    true, min, max                              \
}

#ifdef __cplusplus
}
#endif

#endif

```
仅仅在创建栈的时候传入范围参数，这样解决以上问题点，显然此方法也有如下弊端：
1. 即使生成不带参数范围检查功能的栈，结构体中也会保存栈的最大最小值(虽然不起作用，但是浪费内存)
2. 如果还想在栈内增加其他校验功能，那还会加入新的成员，最终导致栈变的臃肿。

> 解决方案：很明显，需要把校验相关功能分离出来。

#### 检查功能分离出来的栈

```c
#ifndef _STACK_H_
#define _STACK_H_

#include <stddef.h>

#ifdef __cplusplus
extern "C" {
#endif
typedef struct {
    const int min;
    const int max;
} Range;

typedef struct {
    int top;
    const size_t size;
    int * const pBuf;

    const Range* const pRange;
} Stack;

bool push(Stack *p, int val);
bool pop(Stack *p, int *pRet);

#define newStack(buf) {                     \
    0, sizeof(buf) / sizeof(int), (buf),    \
    NULL                                    \
}

#define newStackWithRangeCheck(buf, pRange) {       \
    0, sizeof(buf) / sizeof(int), (buf),            \
    pRange                                          \
}

#ifdef __cplusplus
}
#endif

#endif

```

#### 检查功能的通用化

之前我们只是加入了输入值的范围检查功能，但是一般情况下，检查并不限于范围检查。例如：要求每次push到栈中的值必须比上次的(最开始是0)值大。

首先，将检查输入值的通用指责转移到 Validator 结构体中(引入校验器)

```c
#ifndef _STACK_H_
#define _STACK_H_

#include <stddef.h>

#ifdef __cplusplus
extern "C" {
#endif

typedef struct _Validator {
    bool (* const validate)(struct _Validator *pThis, int val);
    void * const pData;
} Validator;

typedef struct {
    const int min;
    const int max;
} Range;

typedef struct {
    int previousValue;
} PreviousValue;

typedef struct {
    int top;
    const size_t size;
    int * const pBuf;
    Validator * const pValidator;
} Stack;

bool validateRange(Validator *pThis, int val);
bool validatePrevious(Validator *pThis, int val);

bool push(Stack *p, int val);
bool pop(Stack *p, int *pRet);

#define newStack(buf) {                  \
    0, sizeof(buf) / sizeof(int), (buf), \
    NULL                                 \
}

#define rangeValidator(pRange) { \
    validateRange,               \
    pRange                       \
}

#define previousValidator(pPrevious) { \
    validatePrevious,                  \
    pPrevious                          \
}

#define newStackWithValidator(buf, pValidator) { \
    0, sizeof(buf) / sizeof(int), (buf),         \
    pValidator                                   \
}

#ifdef __cplusplus
}
#endif
```



