# Qt 5.11.3编译过程中错误


## `qrandom.cpp:455:62: error: no matching function for call to ‘std::mersenne_twister_engine`

解决方法:

打开 `qrandom.cpp` 文件，文件编辑器打开后，在`220`行添加类型声明：`typedef quint32 result_type`;

## `qbytearraymatcher.h:103:38: error: ‘numeric_limits’ is not a member of ‘std’`

解决方法：

因为`qbytearraymatcher.h` 文件缺少 `limits` 头文件，所以打开此文件添加 `#include <limits>`

## `socketcanbackend.cpp:697:41: error: ‘SIOCGSTAMP’ was not declared in this scope`

解决方法：

在qt源码中搜索`socketcanbackend.cpp`，在头文件中添加`#include <linux/sockios.h>`

## `qjp2handler.cpp:853:41: error: ‘pow’ was not declared in this scope`

解决方法：

在Qt源码中查找`qjp2handler.cpp`，在头文件中添加`#include <math.h>`



