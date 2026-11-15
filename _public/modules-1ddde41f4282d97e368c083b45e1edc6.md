# CMake 中一些有用的模块

CMake 的 [modules](https://cmake.org/cmake/help/latest/manual/cmake-modules.7.html) 集合中包含了大量实用的模块，其中有一些尤为实用。下面介绍几个比较出彩的模块。

## [CMakeDependentOption](https://cmake.org/cmake/help/latest/module/CMakeDependentOption.html)

该模块增加了 `cmake_dependent_option` 命令，用于根据另一组变量是否为真来设置一个选项。用法如下所示：

```cmake
include(CMakeDependentOption)
cmake_dependent_option(BUILD_TESTS "Build your tests" ON "VAL1;VAL2" OFF)
```

当条件（此处为 `VAL1 AND VAL2`）为真时，它的行为与普通的 `option(BUILD_TESTS "Build your tests" ON)` 相同。然而当条件为假时，它并不仅仅是隐藏该选项，而是会将 `BUILD_TESTS` 作为普通（非缓存）变量设置为回退值（此处为 `OFF`），从而遮蔽用户可能在缓存中设置的任何值。因此，只有当条件成立时，`BUILD_TESTS` 才是可供用户配置的；其逻辑大致相当于：

```cmake
if(VAL1 AND VAL2)
    option(BUILD_TESTS "Build your tests" ON)
else()
    set(BUILD_TESTS OFF)
endif()
```

需要注意的是，如果你使用了 `include(CTest)`，通过 `BUILD_TESTING` 来检查测试是否启用是更好的方式，因为它已经为你定义好了。这里只是展示一个 `CMakeDependentOption` 的例子。

## [CMakePrintHelpers](https://cmake.org/cmake/help/latest/module/CMakePrintHelpers.html)

这个模块包含了几个方便的输出函数。`cmake_print_properties` 可以让你轻松地打印属性，而 `cmake_print_variables` 则会打印出你传给它的任意变量的名称和值。

## [CheckCXXCompilerFlag](https://cmake.org/cmake/help/latest/module/CheckCXXCompilerFlag.html)

这个模块用于检查编译器是否支持某个标志。例如：

```cmake
include(CheckCXXCompilerFlag)
check_cxx_compiler_flag(-someflag OUTPUT_VARIABLE)
```

需要注意的是，`OUTPUT_VARIABLE` 也会出现在配置阶段打印的输出中，所以请选个好懂的变量名。

这只是众多类似模块中的一个，其他还有 `CheckIncludeFileCXX`、`CheckStructHasMember`、`TestBigEndian` 以及 `CheckTypeSize` 等，它们允许你检查系统的相关信息（并且你可以将这些信息传递给源代码使用）。

## [`try_compile`](https://cmake.org/cmake/help/latest/command/try_compile.html)/[`try_run`](https://cmake.org/cmake/help/latest/command/try_run.html)

准确地说，这并不是一个模块，但它对上面列出的许多模块至关重要。你可以在配置阶段尝试编译（甚至运行）一小段代码，从而获取关于系统能力的信息。基本语法如下：

```cmake
try_compile(
  RESULT_VAR
    bindir
  SOURCES
    source.cpp
)
```

这里可以添加许多选项，例如 `COMPILE_DEFINITIONS`。在 CMake 3.8+ 中，它会遵循 CMake 的 C/C++/CUDA 标准设置。如果你改用 `try_run`，它还会运行生成的程序，并将运行输出存储在 `RUN_OUTPUT_VARIABLE` 中。

## [FeatureSummary](https://cmake.org/cmake/help/latest/module/FeatureSummary.html)

这是一个相当实用但也有些奇特的模块。它允许你打印出搜索过的所有软件包列表，以及你显式标记的所有选项。它与 [`find_package`](https://cmake.org/cmake/help/latest/command/find_package.html) 有一定关联，但并非完全绑定。和往常一样，你首先需要引入该模块：

```cmake
include(FeatureSummary)
```

然后，对于任何你已经运行或将要运行的 `find_package`，都可以这样扩展它的默认信息：

```cmake
set_package_properties(OpenMP PROPERTIES
    URL "http://www.openmp.org"
    DESCRIPTION "Parallel compiler directives"
    PURPOSE "This is what it does in my package")
```

你也可以将包的 `TYPE` 设置为 `RUNTIME`、`OPTIONAL`、`RECOMMENDED` 或 `REQUIRED`；不过，你不能降低包的类型级别——如果你已经根据某个选项通过 [`find_package`](https://cmake.org/cmake/help/latest/command/find_package.html) 添加了一个 `REQUIRED` 类型的包，最终列表中它仍会显示为 `REQUIRED`。

此外，你还可以将任意选项标记为特性摘要的一部分。如果你选择的名称与某个包同名，它们二者会相互关联。

```cmake
add_feature_info(WITH_OPENMP OpenMP_CXX_FOUND "OpenMP (Thread safe FCNs only)")
```

最后，你可以将特性的汇总信息打印到屏幕或日志文件中：

```cmake
if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)
    feature_summary(WHAT ENABLED_FEATURES DISABLED_FEATURES PACKAGES_FOUND)
    feature_summary(FILENAME ${CMAKE_CURRENT_BINARY_DIR}/features.log WHAT ALL)
endif()
```

你可以自由组合任意你想查看的 `WHAT` 条目，或者直接使用 `ALL`。
