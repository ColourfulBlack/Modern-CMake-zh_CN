# FetchContent (CMake 3.14+)

有时你希望在配置阶段（而非构建阶段）下载数据或软件包。这种做法曾在第三方模块中被重复“发明”过好几次，但最终作为 [FetchContent] 模块被正式加入到了 CMake 本身中。

[FetchContent] 模块拥有出色的文档，我在此不再赘述。其核心思想如下：

- 使用 `FetchContent_Declare(MyName)` 来获取数据或软件包。你可以指定 URL、Git 仓库等。
- 使用 `FetchContent_MakeAvailable(MyName)` 使其可用；如果它是一个带有 `CMakeLists.txt` 的软件包，这会将它添加到你的构建中（就像 `add_subdirectory` 所做的那样）。

例如，下载 Catch2：

```cmake
FetchContent_Declare(
  catch
  GIT_REPOSITORY https://github.com/catchorg/Catch2.git
  GIT_TAG        v3.15.0
)

FetchContent_MakeAvailable(catch)
```

如果你正在维护较旧的代码，你可能还会遇到早期那种手动检查 `MyName_POPULATED` 并调用 `FetchContent_Populate(MyName)` 的写法；该用法已在 CMake 3.30 中被弃用（见 [CMP0169](https://cmake.org/cmake/help/latest/policy/CMP0169.html)），并且在当前版本的 CMake 中会产生警告，因此直接使用 `FetchContent_MakeAvailable` 即可。

可以在[这里](https://gitlab.com/CLIUtils/modern-cmake/-/tree/master/examples/fetch)查看示例。

[fetchcontent]: https://cmake.org/cmake/help/latest/module/FetchContent.html
