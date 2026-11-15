# 导出

:::{danger}

CMake 3.15 中，导出的默认行为发生了变化。由于更改用户主目录中的文件被认为是“令人惊讶的”（确实如此，这就是本章存在的原因），因此它不再是默认行为。若将 CMake 的最低或最高版本设置为 3.15 及以上，除非按如下所示设置 `CMAKE_EXPORT_PACKAGE_REGISTRY`，否则这一行为将不再发生。

:::

从另一个项目访问当前项目有三种方式：子目录、导出构建目录以及安装。要在另一个项目中使用某个项目的构建目录，就需要导出目标。规范的安装流程本就需要导出目标，而允许直接使用构建目录只需额外增加两行代码。这通常不是我推荐的工作方式，但在开发阶段会很有用，也可以作为准备稍后讨论的安装流程的一种方式。

你应该创建一个导出集，通常放在主 `CMakeLists.txt` 文件的末尾附近：

```cmake
export(TARGETS MyLib1 MyLib2 NAMESPACE MyLib:: FILE MyLibTargets.cmake)
```

这会将你列出的目标写入构建目录中的一个文件里，并可选择性地为它们添加命名空间前缀。现在，为了让 CMake 能够找到这个包，请将其导出到 `$HOME/.cmake/packages` 文件夹下：

```cmake
set(CMAKE_EXPORT_PACKAGE_REGISTRY ON)
export(PACKAGE MyLib)
```

现在，如果调用 `find_package(MyLib)`，CMake 就可以找到该构建文件夹了。不妨查看一下生成的 `MyLibTargets.cmake` 文件，这有助于你准确理解它到底创建了什么；它其实只是一个包含导出目标的普通 CMake 文件。

注意，这种方式有一个缺点：如果你有导入的依赖项，则必须在调用 `find_package` 之前先导入它们。这个问题将在下一种方法中得到解决。
