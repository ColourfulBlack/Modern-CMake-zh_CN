# CCache 和一些其他的实用工具

随着版本演进，许多能够帮助你写出优质代码的常见实用工具都已被集成到 CMake 的支持中。这通常表现为某个目标属性以及与之配对的 `CMAKE_*` 初始化变量。这一特性并非绑定于某个特定程序，而是适用于任何行为相似的程序。

所有这些变量或属性都接收以 `;` 分隔的值（CMake 中的标准列表），用于描述要对该目标的源文件运行的程序及其选项。

## CCache

通过设置 `CMAKE_<LANG>_COMPILER_LAUNCHER` 变量或设置目标的 `<LANG>_COMPILER_LAUNCHER` 属性，可以使用类似 CCache[^1] 的工具来“包装”目标的编译过程。在较新的 CMake 版本中，对 CCache 的支持也在不断扩展。实际使用时通常如下所示：

```cmake
find_program(CCACHE_PROGRAM ccache)
if(CCACHE_PROGRAM)
    set(CMAKE_CXX_COMPILER_LAUNCHER "${CCACHE_PROGRAM}")
    set(CMAKE_CUDA_COMPILER_LAUNCHER "${CCACHE_PROGRAM}") # CMake 3.9+
endif()
```

## 一些实用工具

将以下属性或 `CMAKE_*` 初始化变量设置为对应工具的命令行即可启用它们。其中大多数工具仅限于在使用 Make 或 Ninja 生成器构建 C 或 CXX 项目时生效。

- `<LANG>_CLANG_TIDY`: CMake 3.6+
- `<LANG>_CPPCHECK`
- `<LANG>_CPPLINT`
- `<LANG>_INCLUDE_WHAT_YOU_USE`

## Clang tidy

这是以列表形式指定的运行 clang-tidy[^2] 的命令行（请记住，分号分隔的字符串在 CMake 中就是一个列表）。

下面是一个使用 Clang-Tidy 的简单示例：

```bash
~/package # cmake -S . -B build-tidy -DCMAKE_CXX_CLANG_TIDY="$(which clang-tidy);-fix"
~/package # cmake --build build-tidy -j 1
```

其中的 `-fix` 部分是可选的，它会修改你的源文件以尝试修复发出的 clang-tidy 警告（warning）。如果你在 git 仓库中工作，使用 `-fix` 相当安全，因为你可以清楚地看到哪些代码发生了改动。不过，请务必确保**不要并行运行你的 Makefile/Ninja 构建**！如果它尝试对同一个头文件修复两次，效果会非常糟糕。

如果你想显式使用目标属性的形式，以确保只对项目本地的目标调用它，可以设置一个普通变量（例如 `DO_CLANG_TIDY`，而不是 `CMAKE_CXX_CLANG_TIDY` 变量），然后在创建目标时将其添加到目标属性中。你可以通过以下方式在系统路径中查找 clang-tidy：

```cmake
find_program(
    CLANG_TIDY_EXE
    NAMES "clang-tidy"
    DOC "Path to clang-tidy executable"
)
```

## Include what you use

下面是一个使用 include what you use[^3] 的示例。首先，你需要准备好该工具，例如在 Docker 容器中使用，或在 macOS 上通过 Homebrew 执行 `brew install include-what-you-use` 来安装。随后，你可以在不修改源代码的情况下将它传入构建过程：

```bash
~/package # cmake -S . -B build-iwyu -DCMAKE_CXX_INCLUDE_WHAT_YOU_USE=include-what-you-use
```

最后，你可以收集输出并（可选地）应用修复：

```bash
~/package # cmake --build build-iwyu 2> iwyu.out
~/package # fix_includes.py < iwyu.out
```

（你应该先检查一下这些修复是否正确，或者在应用修复后再对代码稍作润色！）

## Link what you use

有一个布尔类型的目标属性 `LINK_WHAT_YOU_USE`，它会在链接时检查是否存在多余的文件。

## Clang-format

遗憾的是，Clang-format[^4] 并没有真正与 CMake 集成。你可以创建一个自定义目标（参见[这篇文章](https://arcanis.me/en/2015/10/17/cppcheck-and-clang-format)），或者直接手动运行它。[这里](https://github.com/kbenzie/git-cmake-format)有一个有趣的项目，不过我还没有真正尝试过；它添加了一个 `format` 目标，甚至还能确保你无法提交未经格式化的文件。

下面这两行命令可以在 git 仓库的 `bash` 中完成格式化检查（假设你已经有一个 `.clang-format` 文件）：

```bash
$ git ls-files -- '*.cpp' '*.h' | xargs clang-format -i -style=file
$ git diff --exit-code --color
```

:::{note}

译者注：以下所有的脚注说明均为译者添加，原文并不包含此信息。脚注的说明资料均整理自互联网。

:::

[^1]: Ccache（或“ccache”）是一个编译器缓存。它通过缓存先前的编译结果并检测何时在重复执行相同的编译来[加速重编译](https://ccache.dev/performance.html)。Ccache 是自由软件，基于 [GNU General Public License version 3](http://www.gnu.org/licenses/gpl.html) 或更高版本发布。可以查看这里的[许可协议页面](https://ccache.dev/license.html)。
[^2]: **clang-tidy** 是一个基于 Clang 的 C++ “linter” 代码分析工具。它旨在提供一个可扩展的框架，用于诊断和修复典型的编程错误，如风格违规、接口误用，或可通过静态分析推断出的缺陷。**clang-tidy** 采用模块化设计，为编写新的检查规则提供了便捷的接口。
[^3]: 一个与 **Clang** 配合使用，用于分析 C 和 C++ 源文件中 `#include` 的工具。
[^4]: ClangFormat 描述了一组建立在 [LibFormat](https://clang.llvm.org/docs/LibFormat.html) 之上的工具。它可以以多种方式支持你的工作流，包括独立工具以及编辑器集成。
