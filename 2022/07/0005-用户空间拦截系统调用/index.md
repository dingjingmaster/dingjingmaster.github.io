# 用户空间拦截系统调用


## 原理

Linux 允许让我们自己的动态库加载在其它动态库之前，甚至是系统库(`libc.so.6`)，如此我们可以通过自己实现动态库并提前加载来拦截系统调用。<br/><br/>
具体例子参看: [https://github.com/dingjingmaster/demo/tree/master/syscall/dlopen](https://github.com/dingjingmaster/demo/tree/master/syscall/dlopen)<br/>
编译之后根据Makefile提示设置环境变量，然后在终端内执行任何包含读写操作的命令，都会有相关打印输出。

## 实现过程
1. 通过 dlopen() 打开动态库
2. 使用 dlsym() 确对应系统调用的地址
3. 自定义系统调用(自定义函数名并保证函数类型与系统调用一致)，可以在自定义函数中实现系统调用拦截操作
4. 将以上代码打包成动态库并编译为 `xxx.so`
5. 当前终端配置环境变量: `export LD_PRELOAD=/path/to/xxx.so`
6. 在当前终端执行包含要拦截系统调用的命令

## 以拦截 `open()` `read()` 系统调用为例

1. 动态库源码
```c
#include <stdio.h>
#include <dlfcn.h>
#include <stdarg.h>
#include <string.h>
#include <unistd.h>
#include <stdbool.h>

void* libc_handle = NULL;
int (*open_ptr) (const char*, int) = NULL;
int (*close_ptr) (int) = NULL;
ssize_t (*read_ptr) (int, void*, size_t) = NULL;

static bool inited = false;

_Noreturn void die (const char* fmt, ...)
{
    va_list va;
    va_start (va, fmt);
    vprintf (fmt, va);
    _exit (0);
}

static void find_original_function ()
{
    if (inited) return;

    printf ("libc path: %s\n", LIBC);

    libc_handle = dlopen (LIBC, RTLD_LAZY);

    if (libc_handle == NULL) {
        die ("cannot open libc.so\n");
    }

    open_ptr = dlsym (libc_handle, "open");
    if (open_ptr == NULL) {
        die ("cannot find open()\n");
    }

    close_ptr = dlsym (libc_handle, "close");
    if (close_ptr == NULL) {
        die ("cannot find close()\n");
    }

    read_ptr = dlsym (libc_handle, "read");
    if (read_ptr == NULL) {
        die ("cannot find read()\n");
    }

    inited = true;
}

int open (const char* pathName, int flag)
{
    find_original_function();

    printf ("start open()\n");
    int fd = (*open_ptr) (pathName, flag);
    printf ("end open()\n");

    return fd;
}

int close (int fd)
{
    find_original_function();

    printf ("start close()\n");
    int ret = (*close_ptr) (fd);
    printf ("end close()\n");

    return ret;
}

ssize_t read (int fd, void* buf, size_t count)
{
    find_original_function();

    printf ("start read()\n");
    ssize_t ret = (*read_ptr) (fd, buf, count);
    printf ("end read()\n");

    return ret;
}
```
2. 编译成动态库
```shell
gcc -fpic -shared -Wall -o dlopen-shared-lib.so dlopen-shared-lib.c -ldl -DLIBC=\"`find /usr/lib -name "libc.so.6"`\"
```

3. 通过uptime命令验证
```shell
LD_PRELOAD=$PWD/dlopen-shared-lib.so && export LD_PRELOAD && uptime
```

> uptime 命令会读取 `/proc/uptime` 文件

