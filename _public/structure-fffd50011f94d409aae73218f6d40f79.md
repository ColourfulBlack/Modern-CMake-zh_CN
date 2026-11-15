# 如何组织你的项目

下面的内容带有一定的主观偏好，但我认为这是有益的。我将会讲解如何组织项目的目录结构。这建立在惯例之上，能够帮助你：

- 轻松阅读遵循相同模式的其他项目，
- 避免会导致冲突的组织模式，
- 防止构建过程变得混乱和复杂。

首先，假设你富有创意地创建了一个名为 `project` 的项目，其中包含一个名为 `lib` 的库和一个名为 `app` 的可执行文件，那么起步时的文件结构应该如下所示：

```
- project
  - .gitignore
  - README.md
  - LICENSE.md
  - CMakeLists.txt
  - cmake
    - FindSomeLib.cmake
    - something_else.cmake
  - include
    - project
      - lib.hpp
  - src
    - CMakeLists.txt
    - lib.cpp
  - apps
    - CMakeLists.txt
    - app.cpp
  - tests
    - CMakeLists.txt
    - testlib.cpp
  - docs
    - CMakeLists.txt
  - extern
    - googletest
  - scripts
    - helper.py
```

这些名称并非一成不变；你会看到关于 `test/` 与 `tests/` 的争论，应用程序文件夹也可能叫其他名字（或者在纯库项目中根本不存在）。有时你还会看到一个存放 Python 绑定的 `python` 文件夹，或者一个存放 CMake 辅助文件（如 `Find<library>.cmake`）的 `cmake` 文件夹。但最基本的结构都在上面了。

可以注意到几个明显的特点：`CMakeLists.txt` 文件分散在各个源代码目录中，但不在 `include` 目录下。这是因为你应该能够将 `include` 目录下的内容直接复制到 `/usr/include` 或类似目录中（配置生成的头文件除外，我会在另一章中介绍），而不会夹杂任何多余文件或引发冲突。这也是为什么在 `include` 目录内部还会有一层以项目命名的子目录。请使用 `add_subdirectory` 来添加包含 `CMakeLists.txt` 的子目录。

你通常还会需要一个 `cmake` 文件夹来存放所有的辅助模块。你的 `Find*.cmake` 文件就放在这里。在 [github.com/CLIUtils/cmake](https://github.com/CLIUtils/cmake) 可以找到一组常用的辅助模块。要将该文件夹添加到你的 CMake 模块搜索路径中，可以这样写：

```cmake
set(CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake" ${CMAKE_MODULE_PATH})
```

你的 `extern` 文件夹几乎应该只包含 Git 子模块（submodule）。通过这种方式，你可以明确控制依赖项的版本，同时依然能够轻松升级。关于添加子模块的例子，请参见后面的“测试”（Testing）章节。

你应该在 `.gitignore` 中添加类似 `/build*` 的规则，这样用户就可以在源代码目录内创建构建目录并在其中执行构建。少数软件包禁止这种做法，但这比要求做完全置于源码树之外的构建（true out-of-source build）、导致每构建一个包都得输入不同路径要好得多。

如果你希望防止构建目录直接选在有效的源代码目录中，可以在顶层 `CMakeLists.txt` 靠近开头的位置添加如下代码：

```cmake
### 要求源码外构建（out-of-source build）
file(TO_CMAKE_PATH "${PROJECT_BINARY_DIR}/CMakeLists.txt" LOC_PATH)
if(EXISTS "${LOC_PATH}")
    message(FATAL_ERROR "You cannot build in a source directory (or any directory with a CMakeLists.txt file). Please make a build subdirectory. Feel free to remove CMakeCache.txt and CMakeFiles.")
endif()
```

可以在这里查看[扩展代码示例](https://github.com/Modern-CMake-CN/Modern-CMake-zh_CN/tree/master/examples/extended-project)。
