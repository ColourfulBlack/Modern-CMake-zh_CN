# 导出与安装

让别人使用你的库有三种好方法和一种坏方法。这三种好方法分别是：将项目作为子目录引入、导出构建树，以及安装 `*Config.cmake` 脚本：

## 查找模块（不好的方式）

如果你是库的作者，不要创建 `Find<mypackage>.cmake` 脚本！这类脚本是为那些作者本身不支持 CMake 的库设计的。请改用下文介绍的 `<mypackage>Config.cmake`。

## 添加子项目

其他包可以将你的项目放在子目录中，接着对该子目录调用 `add_subdirectory`。这适用于纯头文件库和编译速度较快的库。需要注意的是，安装命令可能会干扰父项目，因此可以向 `add_subdirectory` 命令添加 `EXCLUDE_FROM_ALL` 选项；显式使用的目标仍然会被构建。

为了支持这种用法，作为库的作者，请确保使用 `CMAKE_CURRENT_SOURCE_DIR` 而非 `PROJECT_SOURCE_DIR`（其他变量也是如此，比如 `CMAKE_CURRENT_BINARY_DIR` 等二进制目录变量）。你可以通过检查 `CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME`，仅在当前项目作为独立项目构建时添加有意义的选项或默认值。

此外，使用命名空间是个好主意，而且库的使用方式应当与下文介绍的其他方法保持一致，因此你应该添加：

```cmake
add_library(MyLib::MyLib ALIAS MyLib)
```

从而统一所有方法下的使用方式。这里的 ALIAS（别名）目标不会在下文中被导出。

## 导出与安装

另外两种好方法（导出构建树和安装 `*Config.cmake` 脚本）共享了大部分机制，它们将是本节后续章节的主题。
