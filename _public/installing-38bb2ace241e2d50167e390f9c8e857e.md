# 安装

进行安装时（比如执行 `make install`），安装命令会将文件或目标“安装”到安装树中。不要硬编码 `lib` 这样的目标路径（例如在使用 `lib64` 的发行版上这会出错），而应使用标准的 [GNUInstallDirs](https://cmake.org/cmake/help/latest/module/GNUInstallDirs.html) 模块，它提供了类似 `CMAKE_INSTALL_LIBDIR` 这样带有适配当前平台默认值的缓存变量。基础的目标安装命令如下所示：

```cmake
include(GNUInstallDirs)
install(TARGETS MyLib
        EXPORT MyLibTargets
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
        )
```

只有当你需要安装库、静态库或程序时，才需要指定这些不同的目标路径；从 CMake 3.14 开始，你可以完全省略它们，CMake 会自动采用这些合理的默认值。`INCLUDES` 目标路径比较特殊，因为安装目标本身并不会安装头文件，它只会在导出的目标上设置包含目录（这通常已经由 `target_include_directories` 设置过了，所以如果你希望保持 CMake 文件整洁，请检查 `MyLibTargets` 文件，确保同一个包含目录没有被包含两次）。

让 CMake 能够获取版本信息通常是个好主意，这样在使用 `find_package` 时就可以指定版本号。这里使用的是你在 `project()` 命令中设置的项目版本：

```cmake
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    MyLibConfigVersion.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
    )
```

（你也可以完全省略 `VERSION` 参数，它默认就是项目版本。）

接下来有两个选择。你需要创建 `MyLibConfig.cmake`，既可以将目标直接导出到该文件中，也可以手动编写该文件并在其中包含目标文件。如果你有任何依赖项（哪怕只是 OpenMP），就需要采用后一种方式，因此下面我将演示这种方法。

首先，创建一个安装目标文件（类似于在构建目录中创建的文件）：

```cmake
install(EXPORT MyLibTargets
        FILE MyLibTargets.cmake
        NAMESPACE MyLib::
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/MyLib
        )
```

这会将你导出的目标写入该文件中。如果没有依赖项，这里只需用 `MyLibConfig.cmake` 代替 `MyLibTargets.cmake` 即可。接着，在源码树的某处编写一个自定义的 `MyLibConfig.cmake` 文件。如果想要捕获配置阶段的变量，可以使用 `.in` 文件，并采用 `@var@` 语法。其内容如下所示：

```cmake
include(CMakeFindDependencyMacro)

# 捕获配置阶段的变量值（可选）
set(my-config-var @my-config-var@)

# 语法与 find_package 相同
find_dependency(MYDEP REQUIRED)

# 其他额外配置

# 引入目标文件
include("${CMAKE_CURRENT_LIST_DIR}/MyLibTargets.cmake")
```

现在，可以配置该文件（如果使用了 `.in` 文件），然后安装生成的文件。既然我们已经创建了 `ConfigVersion` 文件，这里也是安装它的好地方。

```cmake
configure_file(MyLibConfig.cmake.in MyLibConfig.cmake @ONLY)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/MyLibConfig.cmake"
              "${CMAKE_CURRENT_BINARY_DIR}/MyLibConfigVersion.cmake"
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/MyLib
        )
```

就是这样！现在，当你安装好包之后，`${CMAKE_INSTALL_LIBDIR}/cmake/MyLib`（通常是 `lib/cmake/MyLib`）中就会有 CMake 要搜索的文件（具体来说是 `MyLibConfig.cmake` 和 `MyLibConfigVersion.cmake`），并且该配置文件所使用的目标文件也应该在那里。

当 CMake 搜索包时，它会在当前的安装前缀以及若干标准位置中进行查找。你也可以手动为 CMake 指定包的位置：将安装前缀添加到 `CMAKE_PREFIX_PATH`，或者将 `MyLib_DIR` 设置为包含该配置文件的目录（或者在 CMake 3.12+ 中设置 `MyLib_ROOT`）。如果未找到配置文件，CMake 会向用户输出友好的帮助信息。

上文提到的 [CMakePackageConfigHelpers](https://cmake.org/cmake/help/latest/module/CMakePackageConfigHelpers.html) 模块还提供了额外的工具，可以帮助编写更具可重定位性的 `Config.cmake` 文件。请参阅 CMake 文档中关于 [configure_package_config_file](https://cmake.org/cmake/help/latest/module/CMakePackageConfigHelpers.html#command:configure_package_config_file)（用于替代 `configure_file`）和 `@PACKAGE_INIT@` 替换字符串的说明，以获得：

- 一组自动定义的 `PACKAGE_<var>` 变量（对应 `<var>` 的相对路径版本），以及
- 用于替代 `set()` 的 `set_and_check()`，可自动检查路径是否存在。
