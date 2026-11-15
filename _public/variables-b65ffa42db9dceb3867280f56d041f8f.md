# 变量与缓存

## 本地变量

我们首先讨论变量。你可以这样设置一个本地（local）变量：

```CMake
set(MY_VARIABLE "value")
```

变量名通常全部用大写，变量值紧随其后。你可以通过 `${}` 来访问一个变量，例如 `${MY_VARIABLE}`[^1]。CMake 有作用域的概念；在设置一个变量后，只要处于同一作用域内，你就可以访问它的值。一旦离开函数或子目录中的文件，该变量就不再被定义。你可以通过在末尾添加 `PARENT_SCOPE`，将变量设置到紧邻当前作用域的上一级作用域中。

设置列表时，只需直接传入一系列值：

```cmake
set(MY_LIST "one" "two")
```

它们在内部会变成以 `;` 分隔的值。因此下面这条语句是完全等价的：

```cmake
set(MY_LIST "one;two")
```

`list(` 命令提供了用于操作列表的工具，而 `separate_arguments` 可以把以空格分隔的字符串（就地）拆分成一个列表。需要注意的是，在 CMake 中，如果一个值不包含空格，那么加引号和不加引号的效果是一样的；这使得你在处理确定不可能包含空格的值时，大多数情况下可以省略引号。

当使用 `${}` 语法展开变量时，上述关于空格的所有规则同样适用。对于路径要格外小心；路径随时都可能包含空格，因此在作为变量使用时应当始终加上引号（永远不要写 `${MY_PATH}`，而应该始终写成 `"${MY_PATH}"`）。

## 缓存变量

如果你希望从命令行设置变量，CMake 提供了变量缓存机制。有一些变量已经预置在其中了，比如 `CMAKE_BUILD_TYPE`。声明一个缓存变量并在其尚未被设置时赋予初值的语法如下：

```cmake
set(MY_CACHE_VARIABLE "VALUE" CACHE STRING "Description")
```

这么写**不会**覆盖已有的值。这样设计的目的是让你可以在命令行上设置这些变量，而不会在 CMake 文件执行时被覆盖掉。如果你想把这类变量当作权宜的全局变量来使用，那么可以这样做：

```cmake
set(MY_CACHE_VARIABLE "VALUE" CACHE STRING "" FORCE)
mark_as_advanced(MY_CACHE_VARIABLE)
```

第一行无论如何都会强制设置该变量的值，第二行则会使该变量在你运行 `cmake -L ..` 或使用 GUI 时不会显示在变量列表中。由于这种用法非常普遍，你也可以使用 `INTERNAL` 类型来达到同样的效果（`INTERNAL` 类型隐含了 `FORCE`，并且会在列表和 GUI 中隐藏该条目）：

```cmake
set(MY_CACHE_VARIABLE "VALUE" CACHE INTERNAL "")
```

由于 `BOOL` 是非常常见的变量类型，你可以用下面的简写方式更简洁地设置它：

```cmake
option(MY_OPTION "This is settable from the command line" OFF)
```

对于 `BOOL` 数据类型，`ON` 和 `OFF` 有几种不同的写法。

可以查看 [cmake-variables] 获取 CMake 中已知变量的列表。

## 环境变量

你也可以通过 `set(ENV{variable_name} value)` 和 `$ENV{variable_name}` 来设置和获取环境变量，不过一般来说，最好尽量避免使用它们。

## 缓存

缓存实际上就是一个名为 `CMakeCache.txt` 的文本文件，当你运行 CMake 时会在构建目录中创建它。CMake 就是通过它来记住你设置过的所有内容，因此你不必在每次重新运行 CMake 时都把所有选项重新输入一遍。

## 属性

CMake 存储信息的另一种方式是使用属性（property）。它类似于变量，但附加在某个特定对象上，比如目录或目标。全局属性可以作为一个实用的非缓存全局变量。许多目标属性都是由带有 `CMAKE_` 前缀的同名变量初始化的。例如，设置了 `CMAKE_CXX_STANDARD` 后，意味着之后新创建的所有目标在创建时其 `CXX_STANDARD` 属性都会被设为该值。设置属性有两种方式：

```cmake
set_property(TARGET TargetName
             PROPERTY CXX_STANDARD 11)

set_target_properties(TargetName PROPERTIES
                      CXX_STANDARD 11)
```

第一种形式更加通用，可以一次性设置多个目标、文件或测试，并且提供了一些实用的选项。第二种形式则是为单个目标同时设置多个属性的快捷方式。你也可以用类似的方式获取属性：

```cmake
get_property(ResultVariable TARGET TargetName PROPERTY CXX_STANDARD)
```

可以查看 [cmake-properties] 获取所有已知属性的列表。在某些情况下，你也可以自定义属性[^2]。

[cmake-properties]: https://cmake.org/cmake/help/latest/manual/cmake-properties.7.html
[cmake-variables]: https://cmake.org/cmake/help/latest/manual/cmake-variables.7.html

[^1]: `if` 语句有些奇特，它既可以接收包裹了 `${}` 语法的变量，也可以直接接收不带该语法的变量名；这是历史原因造成的：`if` 的出现早于 `${}` 语法。

[^2]: 例如，接口（Interface）目标可能会对允许的自定义属性有所限制。
