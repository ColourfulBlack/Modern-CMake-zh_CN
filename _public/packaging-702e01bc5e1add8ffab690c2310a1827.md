# 打包

指示 CMake 构建包有两种方式：一种是使用 `CPackConfig.cmake` 文件，另一种是将 CPack 变量集成到 `CMakeLists.txt` 文件中。由于通常需要引入主构建中的变量（比如版本号），如果采用第一种方式，你就需要编写一个用于生成配置的模板文件。这里我将演示第二种集成式写法：

```cmake
# 打包支持
set(CPACK_PACKAGE_VENDOR "Vendor name")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Some summary")
set(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE")
set(CPACK_RESOURCE_FILE_README "${CMAKE_CURRENT_SOURCE_DIR}/README.md")
```

这些是制作二进制包时最常用的变量。二进制包使用 CMake 的安装机制，因此所有被安装的内容都会包含在包中。

你还可以制作源码包。你应该将 `CPACK_SOURCE_IGNORE_FILES` 设置为一组正则表达式，以确保不会打包进多余的文件（比如构建目录或 git 信息）；否则 `make package_source` 会把源码目录中的所有内容原封不动地全部打包进去。你还可以设置源码包生成器，以生成自己喜欢的文件类型：

```cmake
set(CPACK_SOURCE_GENERATOR "TGZ;ZIP")
set(CPACK_SOURCE_IGNORE_FILES
    /.git
    /dist
    /.*build.*
    /\\\\.DS_Store
)
```

注意，这种方式无法在 Windows 系统中正常运行，但是生成的源码包可以在 Windows 系统中正常使用。

最后，需要包含 CPack 模块：

```cmake
include(CPack)
```
