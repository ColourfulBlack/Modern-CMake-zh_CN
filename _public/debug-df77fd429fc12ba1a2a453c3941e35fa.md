# 调试代码

你可能需要调试你的 CMake 构建过程，或是调试你的 C++ 代码。本文将同时涵盖这两者。

## 调试 CMake

首先，让我们来看看调试 `CMakeLists.txt` 及其他 CMake 文件的方法。

### 打印变量

在 CMake 中，历史悠久的打印语句调试法长这样：

```cmake
message(STATUS "MY_VARIABLE=${MY_VARIABLE}")
```

不过，使用一个内置模块可以让这件事变得更加轻松：

```cmake
include(CMakePrintHelpers)
cmake_print_variables(MY_VARIABLE)
```

如果你想要打印属性，这个模块就更加好用了！与其逐个获取每个目标（或其他带有属性的条目，例如 `SOURCES`、`DIRECTORIES`、`TESTS` 或 `CACHE_ENTRIES`——不知为何唯独缺少了全局属性）的属性，不如直接列出它们并一次性打印出来：

```cmake
cmake_print_properties(
    TARGETS my_target
    PROPERTIES POSITION_INDEPENDENT_CODE
)
```

### 跟踪运行

你是否曾想精确观察 CMake 文件中究竟执行了什么、以及何时执行的？`--trace-source="filename"` 选项非常好用。你所指定文件中执行的每一行都会在运行时回显到屏幕上，让你能够清楚地跟踪当前正在发生什么。此外还有一些相关选项，但它们输出的信息往往铺天盖地，容易把你淹没。

例如：

```bash
cmake -S . -B build --trace-source=CMakeLists.txt
```

如果再加上 `--trace-expand` 选项，变量还会直接展开为它们的值。

### 调试与跟踪消息

`message` 命令支持除 `STATUS` 之外的更多日志级别，包括 `VERBOSE`、`DEBUG` 和 `TRACE`（CMake 3.15+）。除非你在配置时提高日志级别，否则这些更安静级别的消息默认会被隐藏：

```bash
cmake -S . -B build --log-level=DEBUG
```

你也可以设置 [`CMAKE_MESSAGE_LOG_LEVEL`](https://cmake.org/cmake/help/latest/variable/CMAKE_MESSAGE_LOG_LEVEL.html) 变量（如果希望在多次重新运行时保持生效，可以将其存入缓存）。这样你就可以通过 `message(DEBUG ...)` 在 CMake 代码中保留详细的日志记录，而不会让日常的配置输出变得杂乱。

### CMake 调试器

CMake 3.27+ 内置了一个支持调试适配器协议（Debug Adapter Protocol，简称 DAP）的交互式调试器，这正是大多数编辑器用于调试代码的同款协议。你可以直接启动它：

```bash
cmake -S . -B build --debugger
```

不过，配合支持 DAP 的工具（例如 VS Code 的 CMake Tools 扩展）使用体验最佳；你可以在 `CMakeLists.txt` 中设置断点、单步执行配置过程，并查看变量与属性。

## 以 Debug 模式构建

对于单配置生成器（single-configuration generators），你可以使用 `-DCMAKE_BUILD_TYPE=Debug` 参数来构建代码，以获取调试标志（debugging flags）。对于多配置生成器（multi-configuration generators，如大多数 IDE），你可以在 IDE 中直接选择构建配置。该模式拥有独立的编译标志（变量名以 `_DEBUG` 而非 `_RELEASE` 结尾），以及形如 `$<CONFIG:Debug>` 或 `$<CONFIG:Release>` 的生成器表达式。

完成 Debug 模式的构建后，你就可以在其上运行调试器（如 gdb 或 lldb）了。
