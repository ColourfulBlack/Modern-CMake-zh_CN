# 测试

## 通用测试信息

你需要在你的主 CMakeLists.txt 文件中添加如下函数调用（而不是在子文件夹中）：

```cmake
if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)
    include(CTest)
endif()
```

这会启用测试功能，并设置一个 `BUILD_TESTING` 选项，让用户可以开启或关闭测试（同时还会配置[一些其他设置](https://gitlab.kitware.com/cmake/cmake/blob/master/Modules/CTest.cmake)）。或者你也可以通过直接调用 `enable_testing()` 来自行开启测试。

当你添加测试文件夹时，你应该这么做：

```cmake
if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME AND BUILD_TESTING)
    add_subdirectory(tests)
endif()
```

这么做（译者注：添加 `CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME`）的原因是，如果有其他人包含了你的包，并且他们开启了 `BUILD_TESTING` 选项，他们通常并不希望构建你包内的测试。在极少数情况下，如果你确实希望同时开启两个包的测试功能，你可以提供一个用于覆盖的变量：

```cmake
if((CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME OR MYPROJECT_BUILD_TESTING) AND BUILD_TESTING)
    add_subdirectory(tests)
endif()
```

上述覆盖方式的主要使用场景其实就是本书自身的示例，因为主 CMake 项目确实需要运行所有子项目的测试。

你可以这样注册测试目标（target）：

```cmake
add_test(NAME TestName COMMAND TargetName)
```

如果你在 `COMMAND` 后写了除目标名之外的其他内容，它将会被注册为一条要运行的命令行指令。在这里使用生成器表达式（generator expression）也是有效的：

```cmake
add_test(NAME TestName COMMAND $<TARGET_FILE:TargetName>)
```

这么写将会使用该目标生成的文件（即可执行文件）的输出路径。

## 将构建作为测试的一部分

如果你想在测试时运行 CMake 来构建一个项目，这也是可以做到的（事实上，CMake 自身就是这样进行自我测试的）。例如，如果你的主项目名为 `MyProject`，并且你有一个可以独立构建的 `examples/simple` 项目，那么可以这么写：

```cmake
add_test(
  NAME
    ExampleCMakeBuild
  COMMAND
    "${CMAKE_CTEST_COMMAND}"
             --build-and-test "${MyProject_SOURCE_DIR}/examples/simple"
                              "${CMAKE_CURRENT_BINARY_DIR}/simple"
             --build-generator "${CMAKE_GENERATOR}"
             --test-command "${CMAKE_CTEST_COMMAND}"
)
```

## 测试框架

可以查看子章节了解主流测试框架的配置方法：

- [GoogleTest](testing/googletest.md)：Google 出品的主流选择。不过开发节奏可能有点慢。
- [Catch2](testing/catch.md)：一个现代的、类似 PyTest 风格且拥有巧妙宏设计的测试框架。
- [DocTest](https://github.com/doctest/doctest)：Catch2 的替代品，据称编译速度更快且更清爽。请参考 Catch2 章节并将其替换为 DocTest 即可。
