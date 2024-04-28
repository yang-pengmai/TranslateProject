---
status: translating
title: "GCC plugin infrastructure"
author: Linux Kernel Community
collector: tttturtle-russ
collected_date: 20240425
translator: yang-pengmai
link: https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/Documentation/kbuild/gcc-plugins.rst
---

# GCC 插件架构基础

## 介绍

GCC 插件是为编译器提供额外功能的可加载模块[^1]。 它们对于运行时插装和静态分析非常有用。
我们可以在编译过程中通过回调[^2]，GIMPLE[^3]，IPA[^4]和 RTL 传递[^5]来分析、修改和添加更多的代码。


内核的GCC插件基础结构支持构建树外模块、交叉编译和在单独的目录中构建。
插件源文件必须由 C++ 编译器编译。


目前GCC插件基础结构只支持一些架构。选择 \"select HAVE_GCC_PLUGINS\" 来查找支持 GCC 插件的架构。

这个架构基础是从 grsecurity[^6] 和 PaX[^7] 移植过来的。

\--

## 目的

GCC 插件的设计目的是提供一个场所，用于试验 GCC 或 Clang 上游没有的潜在编译器功能。
一旦它们的实用性得到验证，目标就是将这些功能添加到 GCC（和 Clang）的上游，然后在
所有支持的 GCC 版本都支持这些功能后，再将它们从内核中移除。

具体来说，新插件应该只实现上游编译器（GCC 和 Clang）不支持的功能。

当 Clang 中存在而 GCC 中不存在某项功能时，应努力将该功能上传到上游 GCC（而不仅仅
是作为内核专用的 GCC 插件），以使整个生态都能从中受益。

类似的，如果 GCC 插件提供的功能在 Clang 中不存在，但该功能被证明是有用的，
也应努力将该功能上传到 GCC（和 Clang）。

在上游 GCC 提供了某项功能后，该插件将无法在相应的 GCC 版本（以及更高版本）下
编译。一旦所有内核支持的 GCC 版本都提供了该功能，该插件将从内核中移除。

## 文档

**\$(src)/scripts/gcc-plugins**

> 这是 GCC 插件的目录。

**\$(src)/scripts/gcc-plugins/gcc-common.h**

> 这是 GCC 插件的兼容性头文件。
> 应始终包含它，而不是单独的 GCC 头文件。

**\$(src)/scripts/gcc-plugins/gcc-generate-gimple-pass.h,
\$(src)/scripts/gcc-plugins/gcc-generate-ipa-pass.h,
\$(src)/scripts/gcc-plugins/gcc-generate-simple_ipa-pass.h,
\$(src)/scripts/gcc-plugins/gcc-generate-rtl-pass.h**

> 这些标头可以自动生成 GIMPLE、SIMPLE_IPA、IPA 和 RTL 通程的注册结构
> 与手动创建结构相比，它们更受欢迎。

## 用法

You must install the gcc plugin headers for your gcc version, e.g., on
Ubuntu for gcc-10:

    apt-get install gcc-10-plugin-dev

Or on Fedora:

    dnf install gcc-plugin-devel libmpc-devel

Or on Fedora when using cross-compilers that include plugins:

    dnf install libmpc-devel

Enable the GCC plugin infrastructure and some plugin(s) you want to use
in the kernel config:

    CONFIG_GCC_PLUGINS=y
    CONFIG_GCC_PLUGIN_LATENT_ENTROPY=y
    ...

Run gcc (native or cross-compiler) to ensure plugin headers are
detected:

    gcc -print-file-name=plugin
    CROSS_COMPILE=arm-linux-gnu- ${CROSS_COMPILE}gcc -print-file-name=plugin

The word \"plugin\" means they are not detected:

    plugin

A full path means they are detected:

    /usr/lib/gcc/x86_64-redhat-linux/12/plugin

To compile the minimum tool set including the plugin(s):

    make scripts

or just run the kernel make and compile the whole kernel with the
cyclomatic complexity GCC plugin.

## 4. How to add a new GCC plugin

The GCC plugins are in scripts/gcc-plugins/. You need to put plugin
source files right under scripts/gcc-plugins/. Creating subdirectories
is not supported. It must be added to scripts/gcc-plugins/Makefile,
scripts/Makefile.gcc-plugins and a relevant Kconfig file.

[^1]: <https://gcc.gnu.org/onlinedocs/gccint/Plugins.html>

[^2]: <https://gcc.gnu.org/onlinedocs/gccint/Plugin-API.html#Plugin-API>

[^3]: <https://gcc.gnu.org/onlinedocs/gccint/GIMPLE.html>

[^4]: <https://gcc.gnu.org/onlinedocs/gccint/IPA.html>

[^5]: <https://gcc.gnu.org/onlinedocs/gccint/RTL.html>

[^6]: <https://grsecurity.net/>

[^7]: <https://pax.grsecurity.net/>
