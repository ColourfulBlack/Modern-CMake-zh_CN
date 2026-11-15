# 用 CMake 进行编程

## 控制流程

CMake 有一个 [`if`](https://cmake.org/cmake/help/latest/command/if.html) 语句，尽管经过多年的迭代它已经变得相当复杂。有一系列全大写的关键字可以在 `if` 语句中使用，并且你通常既可以直接通过名称引用变量，也可以使用 `${}` 语法（从历史渊源来看，`if` 语句比变量展开出现得更早）。下面是一个 `if` 语句的例子：

```cmake
if(variable)
    # 如果 variable 为 `ON`、`YES`、`TRUE`、`Y` 或非零数字
else()
    # 如果 variable 为 `0`、`OFF`、`NO`、`FALSE`、`N`、`IGNORE`、`NOTFOUND`、`""`，或以 `-NOTFOUND` 结尾
endif()
# 如果 variable 展开后不属于上述任何一种情况，CMake 会将其展开后再重试一次
```

如果你显式地放入变量展开（如 `${variable}`），由于可能会发生“对展开结果的再次展开”，这可能会让人有些困惑。因此 CMake 3.1+ 引入了一项策略（[CMP0054](https://cmake.org/cmake/help/latest/policy/CMP0054.html)），防止被引号括起来的展开结果被再次展开。所以，只要 CMake 的最低版本为 3.1+，你就可以这样写：

```cmake
if("${variable}")
    # 如果 variable 不是假值（false-like），则为真
else()
    # 注意未定义的变量会展开为 `""`，因此为假
endif()
```

还有各种各样的关键字可供使用，例如：

- 一元运算符：`NOT`、`TARGET`、`EXISTS`（文件）、`DEFINED` 等。
- 二元运算符：`STREQUAL`、`AND`、`OR`、`MATCHES`（正则表达式）、`VERSION_LESS`、`VERSION_LESS_EQUAL`（CMake 3.7+）等。
- 可以使用圆括号进行分组

## [generator-expressions](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html)

[generator-expressions](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html)（生成器表达式）非常强大，不过稍微有些奇特和专精。大多数 CMake 命令都在配置阶段执行，包括上面看到的 `if` 语句。但是如果你需要让逻辑在构建阶段甚至安装阶段才执行呢？生成器表达式正是为此而引入的[^1]。它们会在目标属性中被求值（evaluate）。

最简单的生成器表达式是信息表达式（informational expressions），其形式为 `$<KEYWORD>`；它会求值为与当前配置相关的一段信息。另一种形式是 `$<KEYWORD:value>`，其中 `KEYWORD` 是控制求值过程的关键字，而 `value` 则是待求值的项（这里同样允许使用信息表达式关键字）。如果 `KEYWORD` 是一个求值为 `0` 或 `1` 的生成器表达式或变量，那么当其为 `1` 时会替换为 `value`，为 `0` 时则不会。你可以嵌套生成器表达式，也可以借助变量来让嵌套表达式的阅读不那么痛苦。部分表达式允许传入多个值，各值之间用逗号分隔[^2]。

例如，如果你只想为 `Debug` 配置添加一个编译标志（flag），可以这样做：

```
target_compile_options(MyTarget PRIVATE "$<$<CONFIG:Debug>:--my-flag>")
```

:::{note}

译者注：这里可能有点令人困惑，其实它包含了两种生成器表达式，分别是配置表达式（configuration expression）和条件表达式（conditional expression）。前者的形式是 `$<CONFIG:cfgs>`，如果当前配置匹配 `cfgs` 列表中的任意一个值，该表达式求值为 `1`，否则为 `0`。后者的形式是 `$<condition:true_string>`，如果 `condition` 值为 `1`，则表达式求值为 `true_string`，否则为空字符串。因此这里表达的含义是：如果当前是 `Debug` 配置，就添加 `--my-flag`。可参见[官方文档](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#genex:condition)。

:::

相比使用专门的 `*_DEBUG` 变量，这是一种更新、更优雅的做法，并且可以推广到生成器表达式所支持的所有场景中。需要注意的是，你永远、绝对不要在配置阶段使用表示“当前配置”的值，因为像 IDE 这样的多配置生成器在配置阶段根本没有所谓的“当前”配置，只有在构建阶段通过生成器表达式以及自定义的 `*_<CONFIG>` 变量才能获取。

生成器表达式的其他常见用途：

- 将某项设置限制为仅对特定语言（例如 `CXX`）生效，以避免与 `CUDA` 等语言混淆，或者将其封装起来以便根据目标语言展现不同行为。
- 访问与配置相关的属性，例如目标文件的位置。
- 为构建目录和安装目录指定不同的路径。

最后一种用法非常普遍。你几乎会在所有支持安装的软件包中看到类似下面的代码：

```cmake
target_include_directories(
    MyTarget
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)
```

:::{note}

译者注：这表示直接构建（BUILD）时目标包含的头文件目录为 `${CMAKE_CURRENT_SOURCE_DIR}/include`，而安装（INSTALL）后目标包含的头文件目录为相对路径 `include`（同时需要配合 `install` 安装对应的头文件）。

:::

## 宏定义与函数

你可以轻松地定义自己的 CMake [`function`](https://cmake.org/cmake/help/latest/command/function.html) 或 [`macro`](https://cmake.org/cmake/help/latest/command/macro.html)。函数和宏之间唯一的区别在于作用域；宏没有独立的作用域。因此，如果你在函数中设置了一个变量并希望它在外部可见，就需要使用 `PARENT_SCOPE`。也正因如此，嵌套函数会有些棘手，因为你必须在每一层函数中显式地将希望对外可见的变量设为 `PARENT_SCOPE`。但好处在于，函数不会像宏那样对外“泄漏”其内部的所有变量。在接下来的例子中，我都将使用函数。

下面是一个简单函数的例子：

```cmake
function(SIMPLE REQUIRED_ARG)
    message(STATUS "Simple arguments: ${REQUIRED_ARG}, followed by ${ARGN}")
    set(${REQUIRED_ARG} "From SIMPLE" PARENT_SCOPE)
endfunction()

simple(This Foo Bar)
message("Output: ${This}")
```

输出如下：

```
-- Simple arguments: This, followed by Foo;Bar
Output: From SIMPLE
```

如果你需要位置参数，需要将它们显式列出，而所有其余参数都会被收集到 `ARGN` 中（`ARGV` 则保存了所有参数，包括你显式列出的那些）。由于 CMake 函数没有返回值，你必须通过设置变量的方式来变通实现。在上面的例子中，你可以显式传入一个要设置的变量名。

## 参数

CMake 拥有一套命名参数系统，你已经在大多数内置 CMake 函数中见过了。你可以通过 [`cmake_parse_arguments`](https://cmake.org/cmake/help/latest/command/cmake_parse_arguments.html) 函数来使用它。下面是它的用法示例：

```cmake
function(COMPLEX)
    cmake_parse_arguments(
        COMPLEX_PREFIX
        "SINGLE;ANOTHER"
        "ONE_VALUE;ALSO_ONE_VALUE"
        "MULTI_VALUES"
        ${ARGN}
    )
endfunction()

complex(SINGLE ONE_VALUE value MULTI_VALUES some other values)
```

在该调用之后，你会在函数内部得到以下变量：

```
COMPLEX_PREFIX_SINGLE = TRUE
COMPLEX_PREFIX_ANOTHER = FALSE
COMPLEX_PREFIX_ONE_VALUE = "value"
COMPLEX_PREFIX_ALSO_ONE_VALUE = <UNDEFINED>
COMPLEX_PREFIX_MULTI_VALUES = "some;other;values"
```

如果你查看官方文档页面，会看到另一种略有不同的写法：使用 `set` 来避免在列表中显式书写分号；你可以根据个人喜好自由选择哪种结构。你还可以将它与上面提到的位置参数混用；任何剩余的未解析参数（也就是可选的位置参数）都会保存在 `COMPLEX_PREFIX_UNPARSED_ARGUMENTS` 中。

[^1]: 它们的表现就像是真正在构建或安装阶段被求值一样，不过实际上它们是针对每个构建配置分别求值的。

[^2]: CMake 官方文档将表达式分为信息表达式（Informational）、逻辑表达式（Logical）和输出表达式（Output）。
