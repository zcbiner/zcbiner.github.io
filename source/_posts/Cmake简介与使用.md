---
title: CMake入门
date: 2018-07-15 16:24:30
tags: Android

---

> CMake 是一个跨平台的开源构建工具，Android NDK开发就是使用CMake进行构建的。

<!--more-->

### 一、CMake基础

#### 1.``CMakeLists.txt``

CMakeLists.txt是一个纯文本文件，是cmake项目配置文件，基本上更改项目构建配置都是围绕着这个文件进行。如果工程存在多个目录，可以在每个要管理的目录都添加一个 CMakeLists.txt 文件。

CMakeList.txt 文件来定制整个编译流程，然后再根据目标用户的平台进一步生成所需的本地化 Makefile 和工程文件。

#### 2.语法规则

- 使用 `#` 作为注释
- 变量使用 ${} 方式取值，但是在 IF 控制语句中是直接使用变量名
- 指令名(参数1 参数2 …)，其中参数之间使用空格或分号隔开
- 指令与大小写无关，但参数和变量是大小写相关的

#### 3.命令

如果要在命令行执行cmake命令，需要了解CMake常用执行命令格式。

```
cmake [<options>] (<path-to-source> | <path-to-existing-build>)
```

- 内部构建

直接在源码目录执行cmake被称为内部构建，如默认命令：

```
cmake .
```



- 外部构建

执行cmake命令时，指定生成产物路径被称为外部构建。

```shell
cmake ..
```

或者：

```shell
cmake [参数] [指定进行编译的目录或存放Makefile文件的目录] [指定CMakeLists.txt文件所在的目录]
```

由于内部构建生成的临时文件可能比源代码还要多，非常影响工程的目录结构和可读性，因此实际使用时，尽量使用外部构建。

#### 4.步骤

在 linux 平台下使用 CMake 生成 Makefile 并编译的流程如下：

1. 在源码目录编写 CMake 配置文件 CMakeLists.txt 。

2. 执行命令 `cmake PATH` 。其中， `PATH` 是 CMakeLists.txt 所在的目录。

3. 使用 `make` 命令进行编译。

### 二、使用cmake构建运行

示例使用cmake构建运行一个简单项目。假设有三个文件：

```cpp
// PrintTest.h
#ifndef CMAKEDEMO_PRINTTEST_H
#define CMAKEDEMO_PRINTTEST_H
#include <string>
class PrintTest {
	public:
    void print(const std::string& content);
};
#endif //CMAKEDEMO_PRINTTEST_H


// PrintTest.cpp
#include "PrintTest.h"
void PrintTest::print(const std::string& content) {
    printf("%s", content.data());
}

// main.cpp
#include "PrintTest.h"
int main() {
    PrintTest printTest = PrintTest();
    printTest.print("---- let me test cmake! ----");
    return 0;
}

```

创建CMakeLists.txt文件，并进行如下配置：

```shell
# 指定最低cmake版本要求
cmake_minimum_required(VERSION 3.16)
# 创建项目标识
project(cmakeDemo)
# 设置c++ 11
set(CMAKE_CXX_STANDARD 11)

# 添加名为testCmake的目标，类型为可执行文件
add_executable(testCmake main.cpp PrintTest.cpp PrintTest.h)
```

使用外部构建，在源码目录新建build文件夹，``cd /build``，然后依次执行如下命令：

```shell
cmake ..
make
./testCmake
```

cmake用于构建，make用于编译和链接。make执行完后会生成的可执行文件为：testCmake，在命令行执行这个文件即可看到输出。

### 三、CMake指令

ps: 指令与大小写无关.

#### 1.project

```shell
project(cmakeDemo)
```

指定项目名。

#### 2.set

```sh
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_FLAGS -g -Wall)
```

定义变量， 多个变量用空格或分号隔开。当需要用到定义的 CMAKE_CXX_STANDARD 变量时，需要用${var}的形式来引用，如：${SRC_LIST}
不过，在 IF 控制语句中可以直接使用变量名。

#### 3.message

```sh
message(STATUS "CMAKE_CXX_FLAGS: " "${CMAKE_CXX_FLAGS}")
```

message 可以在构建的过程中向 stdout 输出一些信息。包含了三种类型：

- SEND_ERROR，产生错误，生成过程被跳过；
- STATUS，输出前缀为--的信息；（由上面例子也可以看到会在终端输出相关信息）
- FATAL_ERROR，立即终止所有 CMake 过程；

#### 4.add_executable

```sh
add_executable(testCmake main.cpp PrintTest.cpp PrintTest.h)
```

将一组源文件 source 生成一个可执行文件。 source 可以是多个源文件，也可以是对应定义的变量.

#### 5.cmake_minimun_required

```sh
cmake_minimum_required(VERSION 3.10)
```

用来指定 CMake 最低版本为3.10，如果没指定，执行 cmake 命令时可能会出错

#### 6.add_subdirectory

```sh
add_subdirectory(subProjectName)
```

用于向当前工程添加存放源文件的子目录，并可以指定中间二进制和目标二进制存放的位置。

#### 7.add_library

```sh
# 编译生成libtestCmake.so
add_library(testCmake SHARED main.cpp PrintTest.cpp PrintTest.h)
```


将一组源文件 source 编译出一个库文件，并保存为 libname.so (lib 前缀是生成文件时 CMake自动添加上去的)。其中有三种库文件类型，不写的话，默认为 STATIC。

- SHARED: 表示动态库，可以在(Java)代码中使用 `System.loadLibrary(name)` 动态调用；
- STATIC: 表示静态库，集成到代码中会在编译时调用；
- MODULE: 只有在使用 dyId 的系统有效，如果不支持 dyId，则被当作 SHARED 对待；
- EXCLUDE_FROM_ALL: 表示这个库不被默认构建，除非其他组件依赖或手工构建

#### 8.find_library

```sh
#语法如下：
find_library (<VAR> name1 [path1 path2 ...]) # 在path1等路径中寻找name1库，找到后存放到变量<VAR>中。

#例子：查找本地 log 库
find_library(log-lib log)
```

用于查找库。

#### 9.include_directories和target_include_directories

两个命令的作用都是：指定**目标**包含的头文件路径。区别是：

include_directories是一个全局包含，向下传递。也就是说在顶层的CMakeLists.txt加了include_directories，那么不管子目录需不需要都会包含。

```SH
target_include_directories(mylib PUBLIC include)
```

#### 10.target_link_libraries

该指令的作用为将目标文件与库文件进行链接。
如：

```
target_link_libraries (Demo ${EXTRA_LIBS})
```

#### 11.set_target_properties

这条指令可以用来设置输出的名称，对于动态库，还可以用来指定动态库版本和API版本。

```sh
set_target_properties(target1 target2 ...
                      PROPERTIES prop1 value1
                      prop2 value2 ...)
```

### 四、CMake在Android中的使用

本地使用 libjpeg.so 示例：

```sh
cmake_minimum_required(VERSION 3.4.1)
#设置生成的so动态库最后输出的路径
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/src/main/jniLibs/${ANDROID_ABI})
#指定要引用的libjpeg.so的头文件目录
set(LIBJPEG_INCLUDE_DIR src/main/cpp/include)
include_directories(${LIBJPEG_INCLUDE_DIR})
#导入libjpeg动态库 SHARED；静态库为STATIC
add_library(jpeg SHARED IMPORTED)
#对应so目录，注意要先 add_library，再set_target_properties
set_target_properties(jpeg PROPERTIES IMPORTED_LOCATION ${PROJECT_SOURCE_DIR}/libs/${ANDROID_ABI}/libjpeg.so)
add_library(compress SHARED src/main/cpp/compress.c)
find_library(graphics jnigraphics)
find_library(log-lib log)
#添加链接上面所有find 和 add 的 library
target_link_libraries(compress jpeg ${log-lib} ${graphics})
```

参考链接：

- [Android NDK 开发：CMake 使用](https://www.jianshu.com/p/c71ec5d63f0d)
- [CMake Tutorial](https://cmake.org/cmake/help/latest/guide/tutorial/index.html)