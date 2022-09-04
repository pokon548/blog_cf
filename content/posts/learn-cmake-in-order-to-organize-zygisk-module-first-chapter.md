+++
title = "从零开始的 Android Inline Hook —— CMake 学习（1）"
date = "2022-09-04T16:02:57+08:00"
author = ""
authorTwitter = "" #do not include @
cover = "https://s1.ax1x.com/2022/09/04/vTiwPx.png"
tags = ["Inline Hook", "CMake"]
keywords = ["inline hook", "cmake"]
description = "将 Zygisk 与 Dobby 融合在一起，形成无懈可击的 Hook 利刃！"
showFullContent = false
readingTime = false
hideComments = false
+++

> 以下的大部分见解源于我对[这篇](https://zhuanlan.zhihu.com/p/367808125) 文章的吸收和理解。非常感谢原作者的分享！:)


## 概览
``CMake`` 是一款实用的项目组织工具，可以用 ``CMakeList.txt`` 指定其所需进行的行为，并得到 ``Makefile`` 或者 ``project`` 文件。在得到这些文件后，可以让 ``make`` 程序[^1]继续执行项目的构建过程，得到所需的构建文件。

CMake 可降低手动编写 ``Makefile(s)`` 的难度和精力成本，实现自动化项目构建与测试。

## 为什么要学习这个
- CMake 是 Android Native 开发中用到的构建工具。而 Zygisk 开发就隶属于 Android Native Development；
- 后续需要用 CMake 将 Dobby 编译到 Zygisk 模块内，便于实现 inline hook。

## 需要学习的地方
- CMake 基础语法；
- 基于其上的 C / C++ 程序项目组织方法。

## CMake 一般使用流程
1. 生成 makefile(s)；
2. 执行构建操作，生成目标文件；
3. 进行其它操作（测试、安装、打包...）。

### 生成构建文件
在开始编译和打包前，需要先生成项目所需的构建文件：

| 参数 | 用途                                                   |
| ---- | ------------------------------------------------------ |
| -S   | 指定源文件的根目录。其需要包含一个 CMakeLists.txt 文件 |
| -B   | 指定构建产物（中间文件、目标文件）存放的位置           |
| -D   | 指定构建过程中用到的变量                               |

如果转换为实际使用到的命令，就是这个样子：
```shell
cmake -S /path/to/source/code/ -B <name_of_build_folder> -D<VARIABLE_NAME> ...
```
实际使用时，通常只需要组织好项目结构，并在恰当的位置写好 CMakelist.txt，即可简单的在当前项目位置里执行：
```shell
cmake -B build
```
生成需要的构建文件。对于该命令而言，会生成在 build 文件夹内。

### 执行构建
在需要的构建文件生成完毕后，就可以开始执行构建操作了[^2]：
| 参数 | 用途 |
| ---- | ----|
| --target | 指定构建目标，以代替默认的构建目标 |
| --parallel/-j | 指定构建时使用的线程数量 |

通常，我们会使用这样的指令：
```shell
cmake --build [<dir> | --preset <preset>]
```

告诉 CMake 构建 --build 后的文件夹。

### 食用例子
假设我们要编译一个经典的 hello world 类程序：
```c
#include <stdio.h>

int main(void) {
    printf("Hello, CMake!");
}
```

我们将该程序的源文件命名为 ``main.c``，放置于项目的 ``src`` 子目录下。包含即将编写的 ``CMakeLists.txt``，整个项目的树状图为：
```
.
├── CMakeLists.txt
└── src
    └── main.c
```

之后，我们开始编写 CMakeLists.txt 文件：
```cmake
# 指定 CMake 的最低版本。3.24 是我目前 Arch Linux 下安装的版本，为方便起见，直接指定了该版本
cmake_minimum_required(VERSION 3.24)

# 指定项目名为 hello_cmake，项目版本号为 1.0.0，项目源代码用到 C 语言
project(hello_cmake VERSION 1.0.0 LANGUAGES C)

# 将 main.c 添加到需要构建的可执行文件里
add_executable(hello_cmake src/main.c)
```

编写完成，回到项目文件夹内，使用如下命令执行 Makefile 的构建操作：
```shell
cmake -B build
```

如果没有问题的话，大概会生成这样的运行日志：
```log
-- The C compiler identification is GNU 12.2.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: <somewhere>/hello_cmake/build
```

项目文件夹内也会生成一个 ``build`` 文件夹，内含大量的中间内容：
```
.
├── build
│   ├── CMakeCache.txt
│   ├── CMakeFiles
│   │   ├── 3.24.1
│   │   │   ├── CMakeCCompiler.cmake
│   │   │   ├── CMakeDetermineCompilerABI_C.bin
│   │   │   ├── CMakeSystem.cmake
│   │   │   └── CompilerIdC
│   │   │       ├── a.out
│   │   │       ├── CMakeCCompilerId.c
│   │   │       └── tmp
│   │   ├── cmake.check_cache
│   │   ├── CMakeDirectoryInformation.cmake
│   │   ├── CMakeOutput.log
│   │   ├── CMakeTmp
│   │   ├── hello_cmake.dir
│   │   │   ├── build.make
│   │   │   ├── cmake_clean.cmake
│   │   │   ├── compiler_depend.make
│   │   │   ├── compiler_depend.ts
│   │   │   ├── DependInfo.cmake
│   │   │   ├── depend.make
│   │   │   ├── flags.make
│   │   │   ├── link.txt
│   │   │   ├── progress.make
│   │   │   └── src
│   │   ├── Makefile2
│   │   ├── Makefile.cmake
│   │   ├── pkgRedirects
│   │   ├── progress.marks
│   │   └── TargetDirectories.txt
│   ├── cmake_install.cmake
│   └── Makefile
├── CMakeLists.txt
└── src
    └── main.c
```

可以发现，我们需要的 ``Makefile`` 已经顺利生成。

之后，我们执行第二个指令执行实际的构建操作：
```shell
cmake --build build
```

大概会生成类似这样的运行日志：
```cmake
[ 50%] Building C object CMakeFiles/hello_cmake.dir/src/main.c.o
[100%] Linking C executable hello_cmake
[100%] Built target hello_cmake
```
此时，我们想要的 ``hello_cmake`` 可执行文件已经生成，位于 ``build`` 文件夹下。执行它：

```shell
./build/hello_cmake
```

顺利得到想要的输出结果：
```
Hello, CMake!
```

[^1]: cmake 也可使用其它的程序执行构建过程。在这里我则使用 make 执行底层的构建操作。
[^2]: 下面的参数都是带有两个 - 的。不过，貌似博客会将两个 - 转译为别的东西 :(