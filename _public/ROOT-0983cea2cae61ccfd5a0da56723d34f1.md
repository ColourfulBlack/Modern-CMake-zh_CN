# ROOT

[ROOT](https://root.cern/) 是用于高能物理学的 C++ 工具包，体量非常庞大。在 CMake 中确实有很多使用它的方法，不过你能找到的许多甚至大多数示例可能都是错的。下面给出我推荐的方式。

最重要的是，当前任何版本的 ROOT（撰写本文时为 6.3x 系列）都表现得像一个规范的 CMake 项目：它提供导入目标、为这些目标导出 C++ 标准特性，并会自动加载其 CMake 宏。若确实必须支持非常古老的 ROOT 版本（6.20 之前），请参阅文末的附录。

## 查找 ROOT

ROOT 支持配置文件查找机制，因此只需编写如下代码：

```{literalinclude} ../../examples/root-simple/CMakeLists.txt
:start-after: [find_package]
:end-before: [find_package]
:language: cmake
```

即可尝试查找 ROOT 包。若尚未配置好路径，可以将 `-DROOT_DIR=$ROOTSYS/cmake` 传递给 CMake 来定位 ROOT。（不过，实际上你应该先执行 `source thisroot.sh`。）

## 正确的方式（使用目标）

ROOT 的导入目标携带了所需的包含目录、编译标志与依赖项。链接时，只需挑选想要使用的库即可：

```{literalinclude} ../../examples/root-simple/CMakeLists.txt
:start-after: [add_and_link]
:end-before: [add_and_link]
:language: cmake
```

若想查看默认库列表，可以在命令行运行 `root-config --libs`。对于典型的 ROOT 安装，输出大致如下：

- `ROOT::Core`
- `ROOT::Gpad`
- `ROOT::Graf3d`
- `ROOT::Graf`
- `ROOT::Hist`
- `ROOT::Imt`
- `ROOT::MathCore`
- `ROOT::Matrix`
- `ROOT::MultiProc`
- `ROOT::Net`
- `ROOT::Physics`
- `ROOT::Postscript`
- `ROOT::RIO`
- `ROOT::ROOTDataFrame`
- `ROOT::ROOTVecOps`
- `ROOT::Rint`
- `ROOT::Thread`
- `ROOT::TreePlayer`
- `ROOT::Tree`

## 旧的全局方式

ROOT [提供了一个工具](https://root.cern/manual/integrate_root_into_my_cmake_project/)来配置 ROOT 项目，可以使用 `include("${ROOT_USE_FILE}")` 激活它。这会自动创建不美观的目录级设置和全局变量。它在初期配置时的确能节省一点时间，但如果后续想处理任何稍微复杂的需求，就会浪费大量时间。只要不是构建库，对于简单脚本而言倒也无妨。包含目录和编译标志会被全局设置，但你仍然需要手动链接 `${ROOT_LIBRARIES}`，还可能需要加上 `ROOT_EXE_LINKER_FLAGS`（必须先对其调用 `separate_arguments` 再进行链接，否则在像 macOS 这样存在多个标志的系统上会报错）。

具体写法如下：

```{literalinclude} ../../examples/root-usefile/CMakeLists.txt
:start-after: [core]
:end-before: [core]
:language: cmake
```

## 组件

查找 ROOT 时允许指定组件。CMake 会将列出的组件添加到 `${ROOT_LIBRARIES}` 中，因此若使用该变量构建自己的目标，可以避免把组件列表写两遍。不过，这并没有解决依赖关系问题；过去如果只列出 `RooFit` 而不列出 `RooFitCore` 就会报错。如果链接的是 `ROOT::RooFit` 而非 `${ROOT_LIBRARIES}`，则无需再单独指定 `RooFitCore`。

## 生成字典

生成字典是 ROOT 弥补 C++ 缺乏反射特性的一种手段。它让 ROOT 能够了解自定义类的细节，从而保存对象、在 Cling 解释器中展示方法等。源代码需要进行如下修改以支持字典生成：

- 类定义应当以 `ClassDef(MyClassName, 1)` 结尾
- 类实现中应当包含 `ClassImp(MyClassName)`

ROOT 提供了 `rootcling` 和 `genreflex`（`rootcling` 的遗留接口）这两个可执行程序，用于生成构建字典所需的源文件；同时还定义了 CMake 函数 `root_generate_dictionary`，用于在构建过程中调用 `rootcling`。定义该函数的宏文件会自动加载，因此只要运行了 `find_package(ROOT)`，该函数便立即可用。（只有非常古老的 ROOT 版本才需要手动包含宏文件；该文件在 6.20 中已重命名为 `RootMacros.cmake` 并改为自动加载 —— 详见文末附录。）

`rootcling` 使用遵循[特定格式][linkdef-root]的特殊头文件来决定为哪些内容生成字典。该文件名可以使用任意前缀，但**必须**以 `LinkDef.h` 结尾。编写好这个头文件后，就可以调用字典生成函数了。

### 手动构建字典

有时，你可能希望让 ROOT 生成字典，然后自行将生成的源文件添加到库目标中。调用 `root_generate_dictionary` 时，依次传入字典名称（例如 `G__Example`）、所需的头文件，最后在 `LINKDEF` 关键字之后列出特殊的 `LinkDef.h` 文件：

```cmake
root_generate_dictionary(G__Example Example.h LINKDEF ExampleLinkDef.h)
```

该命令将创建三个文件：

- `${NAME}.cxx`：创建库时，应将该文件包含在源文件列表中。
- `lib{NAME}.rootmap`（移除 `G__` 前缀）：纯文本格式的 rootmap 文件
- `lib{NAME}_rdict.pcm`（移除 `G__` 前缀）：[ROOT 预编译模块文件][root pre-compiled module file]

你必须创建的目标名称（`${NAME}`）由字典名决定：若字典名以 `G__` 开头，则会将该前缀移除；否则直接使用字典名。

最后两个输出文件必须与生成的库文件位于同一目录下。这是通过检查 `CMAKE_LIBRARY_OUTPUT_DIRECTORY` 实现的（它不会读取局部目标的属性设置）。如果设置了库目录（libdir），但没有设置（全局）安装位置，还需要将 `ARG_NOINSTALL` 设置为 `TRUE`。

### 使用现有目标构建字典

除了手动将生成的字典源文件添加到库源码中，还可以通过传入 `MODULE` 参数让 ROOT 自动完成这一步。该参数应当指定一个已存在的构建目标名称：

```cmake
add_library(Example)
root_generate_dictionary(G__Example Example.h MODULE Example LINKDEF ExampleLinkDef.h)
```

字典的完整名称（例如 `G__Example`）不应与 `MODULE` 参数相同。

[linkdef-root]: https://root.cern/manual/io_custom_classes/#selecting-dictionary-entries-linkdefh
[root pre-compiled module file]: https://inspirehep.net/literature/1413967

---

## 使用旧版 ROOT

如果确实必须使用非常古老的 ROOT 版本（6.20 之前），在使用 `root_generate_dictionary` 之前需要手动加载字典宏：

```cmake
include("${ROOT_DIR}/modules/RootNewMacros.cmake")
# 对于早于 6.16 的 ROOT 版本，如果全局包含目录列表为空，
# 构建就会出错！
if (${ROOT_VERSION} VERSION_LESS "6.16")
    include_directories(ROOT_NONEXISTENT_DIRECTORY_HACK)
endif()
```

（`include_directories` 这一行是为了绕过 `RootNewMacros` 文件中的一个 bug：除非至少存在一个全局包含目录或 `inc` 文件夹，否则字典生成就会失败；这里列出的目录本身并不需要真实存在。）

此外，还需要自行修补缺失的目标属性和标志，写法如下：

```cmake
# 在 ROOT 6.10 和 6.12 中，ROOT 目标缺失了包含目录和编译标志
set_property(TARGET ROOT::Core PROPERTY
    INTERFACE_INCLUDE_DIRECTORIES "${ROOT_INCLUDE_DIRS}")

# 早期版本的 ROOT 没有在目标上包含所需的编译标志
add_library(ROOT::Flags_CXX IMPORTED INTERFACE)


# ROOT 6.14 及更早版本的链接器标志中存在空格 bug
string(REPLACE "-L " "-L" ROOT_EXE_LINKER_FLAGS "${ROOT_EXE_LINKER_FLAGS}")

# 修复 ROOT_CXX_FLAGS 实际上并不是 CMake 列表的问题
separate_arguments(ROOT_CXX_FLAGS)
set_property(TARGET ROOT::Flags_CXX APPEND PROPERTY
    INTERFACE_COMPILE_OPTIONS ${ROOT_CXX_FLAGS})

# 添加宏定义
separate_arguments(ROOT_DEFINITIONS)
foreach(_flag ${ROOT_DEFINITIONS})
    # 移除可能存在的 -D 或 /D 前缀
    string(REGEX REPLACE [=[^[-//]D]=] "" _flag ${_flag})
    set_property(TARGET ROOT::Flags_CXX APPEND PROPERTY INTERFACE_COMPILE_DEFINITIONS ${_flag})
endforeach()

# 这同样修复了链接器标志中的一个 bug
separate_arguments(ROOT_EXE_LINKER_FLAGS)
set_property(TARGET ROOT::Flags_CXX APPEND PROPERTY
    INTERFACE_LINK_LIBRARIES ${ROOT_EXE_LINKER_FLAGS})

# 请确保同时也链接了 ROOT::Flags_CXX！
```
