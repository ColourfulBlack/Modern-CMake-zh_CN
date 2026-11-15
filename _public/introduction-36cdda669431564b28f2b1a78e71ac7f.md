# 现代 CMake 简介

人们喜欢对构建系统抱有敌意。
看看 CppCon17 上的演讲就知道了，开发者们把构建系统的现状当成头号笑料来调侃。
这引出了一个问题：为什么？
诚然，在构建项目时确实不乏各种各样的问题。
但我认为，对于其中的不少问题，我们已经有了一个非常好的解决方案。
那就是 CMake。但不是 CMake 2.8——那甚至是在 C++11 问世之前发布的！
也不是网上流传的那些糟糕的 CMake 示例（甚至包括 KitWare 自己的教程列表里发布的一些例子）。
我指的是现代 CMake（Modern CMake）：CMake 3.15+，甚至是 CMake 4.4+！
它简洁、强大、优雅，因此你可以把大部分时间花在编写代码上，而不是往一个不可读、不可维护的 Make（或 CMake 2）文件里堆砌代码行。

:::{attention}

本书是一份持续维护的文档。对于英文原版，你可以在 [GitLab](https://gitlab.com/CLIUtils/modern-cmake) 上提 Issue 或合并请求，也可以[下载英文版 PDF](https://CLIUtils.gitlab.io/modern-cmake/modern-cmake.pdf)；对于简体中文版，欢迎在 [GitHub](https://github.com/Modern-CMake-CN/Modern-CMake-zh_CN) 上反馈问题或参与贡献，也可以点击获取中文版 [PDF 下载](https://modern-cmake-cn.github.io/Modern-CMake-zh_CN/modern-cmake.pdf)。此外，也请务必查看一下 [HSF CMake Training][]（另一份优秀的 CMake 教程）！

:::

简而言之，如果你正在考虑使用 Modern CMake，以下是你心中最可能存在的疑问：

## 为什么我需要一个好的构建系统？

以下情况是否有适用于你的？

- 你想避免将路径硬编码
- 你需要在不止一台电脑上构建软件包
- 你想使用 CI（持续集成）
- 你需要支持不同的操作系统（哪怕只是不同风味的 Unix）
- 你想支持多个编译器
- 你想使用 IDE，但也许并非所有时候都用
- 你想从逻辑上描述程序的组织结构，而不是堆砌编译标志和命令
- 你想使用第三方库
- 你想使用诸如 Clang-Tidy 之类的工具来辅助编码
- 你想使用调试器

如果是这样，你会从类似 CMake 的构建系统中受益。

## 为什么答案一定是 CMake？

构建系统是一个热门话题。当然，市面上有很多选择。但是，即便某个构建系统真的非常优秀，或者复用了某种大家熟悉的语法，也依然无法企及 CMake。
为什么？
因为生态支持。
每个 IDE 都支持 CMake（或者说 CMake 支持那个 IDE）。
使用 CMake 的软件包数量超过了其他任何构建系统。
因此，如果你在代码中使用一个设计为可被引入的库，你面临两个选择：要么自己编写构建系统，要么使用该库提供的构建系统之一——而其中几乎总会包含 CMake。
如果你引入了多个项目，CMake 很快就会成为它们共同支持的交集。
而且，如果你需要使用系统中预装的库，它带有 Find CMake 脚本或 Config CMake 脚本的概率也极高。

## 我难道不应该支持尽可能老的版本吗？

简短的回答：不应该。

如果你将 CMake 的最低版本设置得太低，CMake 会产生警告甚至报错。截至 CMake 4.4，你能设置的最老版本是 3.5，且任何低于 3.10 的设置都会触发警告。虽然用户在紧急情况下可以通过将 `CMAKE_POLICY_VERSION_MINIMUM` 设置为有效版本来绕过这一限制，但你并不希望用户不得不这么做。

你**至少**应该使用在你的编译器发布之后推出的 CMake 版本，因为它需要知道该版本编译器的标志等信息。而且，由于 CMake 会自动按照你在 CMake 文件中指定的最低要求版本来兼容旧行为，因此即便在系统范围内安装新版本的 CMake 也是相当安全的。你**至少**应该在本地安装新版本。这非常简单（在许多情况下只需 1–2 行命令），你会发现花 5 分钟就能为你省下数百行代码和数小时编写 `CMakeLists.txt` 的时间，而且从长远来看也更容易维护。

合适的最低版本快速总结（书中会有更详细的介绍）：

- 3.24：包查找系统非常出色，适合作为支持包作者的良好最低版本。
- 3.18：对 Python 和 CUDA 有相当好的支持，大多数系统至少都具备此版本。
- 3.15：这是大多数项目应当设定的下限，Ubuntu 20.04+ 已自带 3.16+。
- 3.10：避免触发 CMake 警告的最低版本，Ubuntu 18.04+。
- 3.5：仅当你**确实**别无选择时使用。
- 只要你同时设置了最高版本，甚至可以将最低版本设得比这更低。但请不要这么做。

请始终将最高版本设置为你测试过的最高版本（例如 4.4）。这样一来，只要 CMake 尚未将该**最高**版本移出其支持窗口，你的配置就能继续正常工作，而不会受限于最低版本何时被淘汰！

本书试图解决网上泛滥的糟糕示例与所谓“最佳实践”所带来的问题。

## 其他资料

本书原作者的其他资料：

- [HSF CMake Training][]
- [Interactive Modern CMake talks](https://gitlab.com/CLIUtils/modern-cmake-interactive-talk)

网上还有一些其他地方可以找到优质资料，以下是其中的一部分：

- [The official help](https://cmake.org/cmake/help/latest/)：非常棒的官方文档。结构清晰，搜索功能出色，而且可以在顶部切换版本。它只是缺少一份优秀的“最佳实践教程”，而这正是本书试图填补的空白。
- [Effective Modern CMake](https://gist.github.com/mbinna/c61dbb39bca0e4fb7d1f73b0d66a4fd1)：一份很棒的 Do's and Don'ts 清单。
- [Embracing Modern CMake](https://steveire.wordpress.com/2017/11/05/embracing-modern-cmake/)：一篇很好地阐述了这一术语内涵的文章。
- [It's time to do CMake Right](https://pabloariasal.github.io/2018/02/19/its-time-to-do-cmake-right/)：一套很不错的现代 CMake 项目最佳实践。
- [The Ultimate Guide to Modern CMake](https://rix0r.nl/blog/2015/08/13/cmake-guide/)：一篇初衷类似但略显过时的文章。
- [More Modern CMake](https://youtu.be/y7ndUhdQuU8)：来自 Meeting C++ 2018 的精彩演讲，推荐使用 CMake 3.12+。该演讲将 CMake 3.0+ 称为“Modern CMake”，将 CMake 3.12+ 称为“More Modern CMake”。
- [Oh No! More Modern CMake](https://www.youtube.com/watch?v=y9kSr5enrSk)：More Modern CMake 的续篇。
- [toeb/moderncmake](https://github.com/toeb/moderncmake)：关于 CMake 3.5+ 的优秀演示文稿与示例，涵盖了从基础语法到项目组织的内容。

## 致谢

Modern CMake 最初由 [Henry Schreiner](https://iscinumpy.gitlab.io) 编写。其他贡献者可以在 [GitLab 贡献者列表](https://gitlab.com/CLIUtils/modern-cmake/-/network/master)中找到。

[hsf cmake training]: https://hsf-training.github.io/hsf-training-cmake-webpage/01-intro/index.html
