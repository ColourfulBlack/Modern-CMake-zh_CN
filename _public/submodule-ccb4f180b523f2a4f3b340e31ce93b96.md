# Git 子模组（Submodule）

如果你想要添加一个 Git 仓库，它与你的项目仓库使用相同的托管服务（诸如 GitHub、GitLab、BitBucket 等），下面是正确地将一个子模组添加到 `extern` 目录中的 Git 命令：

```bash
$ git submodule add ../../owner/repo.git extern/repo
```

此处的关键是使用指向该仓库的相对路径；它可以保证你使用与父仓库相同的访问方式（ssh 或 https）访问子模组。这在大多数情况下都能工作得相当好。当你在一个子模组里的时候，你可以把它看作一个普通的仓库；而当你在父仓库里时，你可以用 `add` 来改变当前的提交指针。

但传统的缺点是，你的用户要么必须懂 git submodule 命令，这样他们才可以 `init` 和 `update` 仓库，要么必须在最开始克隆你的仓库的时候加上 `--recursive` 选项。针对这种情况，CMake 可以提供一种解决方案：

```cmake
find_package(Git QUIET)
if(GIT_FOUND AND EXISTS "${PROJECT_SOURCE_DIR}/.git")
# 按需更新子模组
    option(GIT_SUBMODULE "Check submodules during build" ON)
    if(GIT_SUBMODULE)
        message(STATUS "Submodule update")
        execute_process(COMMAND ${GIT_EXECUTABLE} submodule update --init --recursive
                        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
                        RESULT_VARIABLE GIT_SUBMOD_RESULT)
        if(NOT GIT_SUBMOD_RESULT EQUAL "0")
            message(FATAL_ERROR "git submodule update --init --recursive failed with ${GIT_SUBMOD_RESULT}, please checkout submodules")
        endif()
    endif()
endif()

if(NOT EXISTS "${PROJECT_SOURCE_DIR}/extern/repo/CMakeLists.txt")
    message(FATAL_ERROR "The submodules were not downloaded! GIT_SUBMODULE was turned off or failed. Please update submodules and try again.")
endif()
```

第一行使用 CMake 内置的 `FindGit.cmake` 检测是否安装了 Git。接着，如果项目源码处于 Git 检出目录中，则添加一个选项（默认值为 `ON`），以便开发者在需要时关闭这个功能。然后我们运行命令来获取所有仓库，如果该命令失败则报错终止，并输出一条友好的错误信息。最后，无论通过何种方式获取仓库，在继续之前我们都会验证这些仓库确实已经存在。你也可以使用 `OR` 来列举多个仓库。

现在，你的用户可以完全不必理会子模组的存在，而你依然能够保持良好的开发实践！唯一需要开发者注意的一点是：如果你正在子模组内部进行开发，重新运行 CMake 时会重置该子模组。只需要将新的提交添加到父仓库的暂存区（staging area），就可以避免这个问题。

然后你就可以引入对 CMake 有良好支持的项目了：

```cmake
add_subdirectory(extern/repo)
```

或者，如果这是一个纯头文件（header-only）项目，你可以自己创建一个接口库目标（interface library target）。又或者，如果该项目支持的话，你可以使用 `find_package`，通常需要将初始搜索目录准备为你所添加的那个目录（请查看相关文档或你所使用的 `Find*.cmake` 文件）。如果你将其追加到 `CMAKE_MODULE_PATH` 中，还可以引入一个 CMake 辅助文件目录，例如添加 `pybind11` 改进过的 `FindPython*.cmake` 文件。

## 小贴士：获取 Git 版本号

既然你在上面的代码片段中已经通过 `find_package(Git)` 找到了 Git，你还可以使用 `${GIT_EXECUTABLE}` 将当前的提交哈希（commit hash）捕获到一个变量中，例如将其嵌入到你的构建中：

```cmake
execute_process(COMMAND ${GIT_EXECUTABLE} rev-parse --short HEAD
                WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
                OUTPUT_VARIABLE PACKAGE_GIT_VERSION
                ERROR_QUIET
                OUTPUT_STRIP_TRAILING_WHITESPACE)
```
