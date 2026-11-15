# 使用 CMake 下载项目

## 在构建时（build time）下载

在 CMake 3.11 之前，下载包的主要方式是在构建时进行的。这会带来几个问题；其中最重要的问题是 `add_subdirectory` 无法作用于一个尚不存在的目录！为此提供的内置工具 ExternalProject 必须通过自己执行构建来绕过这个问题（不过，它也因此能够构建非 CMake 的包）。[^1]

[^1]: 注意，ExternalData 是用于处理非软件包数据的工具。

## 在配置时（configure time）下载

如果你更喜欢在配置时下载，现代的做法是使用内置的 [FetchContent](fetch.md) 模块（CMake 3.11+ 引入，并在 3.14 中得到了大幅改进）。如果你受限于必须支持非常老旧的 CMake 版本，[Crascit/DownloadProject](https://github.com/Crascit/DownloadProject) 仓库曾是一个开箱即用（drop-in）的解决方案，但如今它已被 FetchContent 取代。不过，由于子模块（submodule）非常好用，我已经停用了大多数针对诸如 GoogleTest 之类项目的自动下载，而是将它们迁移到了子模块中。自动下载在没有互联网访问的环境下更难模拟，而且它们通常实现在构建目录中——如果你有多个构建目录，这既浪费时间又浪费空间。
