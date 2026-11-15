# CMake 对 IDE 的支持

一般来说，标准的 CMake 项目本身就已经支持各种 IDE。不过还有一些额外的设置，可以让 IDE 表现得更好。

## 用文件夹来组织目标（target）

一些 IDE（例如 Xcode）支持文件夹功能。自 CMake 3.26 起（参见策略 [CMP0143](https://cmake.org/cmake/help/latest/policy/CMP0143.html)），全局属性 `USE_FOLDERS` 默认即为 `ON`；在此之前，你需要手动启用它，以允许 CMake 按文件夹组织你的文件：

```cmake
set_property(GLOBAL PROPERTY USE_FOLDERS ON)
```

然后，你可以在创建目标之后，将目标归入指定的文件夹中：

```cmake
set_property(TARGET MyFile PROPERTY FOLDER "Scripts")
```

文件夹可以使用 `/` 进行嵌套。

## 用文件夹来组织文件

你也可以控制目标内部的文件夹如何呈现。有两种方式，均使用 [`source_group`](https://cmake.org/cmake/help/latest/command/source_group.html) 命令。传统的方式是：

```cmake
source_group("Source Files\\New Directory" REGULAR_EXPRESSION ".*\\.c[ucp]p?")
```

你可以用 `FILES` 显式列出文件，或者使用 `REGULAR_EXPRESSION`（正则表达式）进行匹配。通过这种方式，你可以完全掌控文件夹结构。不过，如果你在磁盘上的目录结构本身就已经设计得很好，你可能只想直接在 IDE 中复现这种结构。在 CMake 3.8+ 中，使用新版 [`source_group`](https://cmake.org/cmake/help/latest/command/source_group.html) 命令就能非常轻松地做到这一点：

```cmake
source_group(TREE "${CMAKE_CURRENT_SOURCE_DIR}/base/dir" PREFIX "Header Files" FILES ${FILE_LIST})
```

对于 `TREE` 选项，通常应该给出一个以 `${CMAKE_CURRENT_SOURCE_DIR}/` 开头的完整路径（因为此命令会相对于构建目录来解析路径）。`PREFIX` 指定了文件在 IDE 目录结构中放置的位置，而 `FILES` 选项则接收文件列表。CMake 会从 `FILE_LIST` 的路径中剥离 `TREE` 路径前缀，再冠以 `PREFIX`，以此构成 IDE 中的文件夹结构。

> 注意：如果你需要支持低于 3.8 版本的 CMake，我建议用版本判断保护上述命令，仅在 CMake 3.8+ 上支持这种美观的文件夹布局。关于旧版本实现此类文件夹布局的方法，请参见[这篇博文][sorting]。

## 在 IDE 中运行 CMake

要使用 IDE，如果 CMake 能够生成对应 IDE 的项目文件（例如 Xcode、Visual Studio），可以通过传入 `-G"name of IDE"` 来实现；如果 IDE 已经内置了对 CMake 的支持（例如 CLion、QtCreator 以及许多其他 IDE），直接从 IDE 中打开 `CMakeLists.txt` 文件即可。

[sorting]: https://web.archive.org/web/20231205143313/https://blog.audio-tk.com/2015/09/01/sorting-source-files-and-projects-in-folders-with-cmake-and-visual-studioxcode/
