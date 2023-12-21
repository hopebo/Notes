# yum 安装 rpm 包

## 安装 gcc/g++
`yum search gcc`列表中会出现`gcc-10-repo.noarch : Repo file to install GCC-10.2.1 with GLIBC-2.32`的记录，代表的是一个 rpm 包源 repository 的配置文件，noarch 后缀表示不区分架构类型，对所有 CPU 都可用。

继续使用`yum install gcc-10-repo.noarch`后会在`/etc/yum.repos.d`路径下增加对应的源地址。

之后就可以通过`--repoid gcc-10`从新的源安装 rpm 包：
```
$ yum search gcc --repoid gcc-10

========================================================================================================= Name Exactly Matched: gcc ==========================================================================================================
gcc.x86_64 : Various compilers (C, C++, Objective-C, ...)
======================================================================================================== Name & Summary Matched: gcc =========================================================================================================
gcc-c++.x86_64 : C++ support for GCC
devtoolset-7-gcc.x86_64 : GCC version 7
gcc-gdb-plugin.x86_64 : GCC plugin for GDB
gcc-objc.x86_64 : Objective-C support for GCC
gcc-objc++.x86_64 : Objective-C++ support for GCC
libgcc.x86_64 : GCC version 10 shared support library
gcc-debuginfo.x86_64 : Debug information for package gcc
devtoolset-7-gcc-c++.x86_64 : C++ support for GCC version 7
gcc-gnat.x86_64 : Ada 83, 95, 2005 and 2012 support for GCC
gcc-plugin-devel.x86_64 : Support for compiling GCC plugins
libgccjit.x86_64 : Library for embedding GCC inside programs and libraries
libgccjit-devel.x86_64 : Support for embedding GCC inside programs and libraries
alios7u-2_32-gcc-10-repo.noarch : Repo file to install GCC-10.2.1 with GLIBC-2.32

$ yum install gcc-c++.x86_64 --repoid gcc-10
```

就可以将 10.2.1 版本的 gcc 和对应库安装下来了。

## 本地安装 rpm 包

有的高版本包在当前系统版本下的 yum 无法默认安装时，可以在[rpmfind](https://rpmfind.net/linux/rpm2html/search.php)上搜索下载后手动安装。

```
sudo rpm -i sample_file.rpm

或者

sudo yum localinstall sample_file.rpm
```

遇到本地存在低版本包时，不会自动替换，需要加上参数`--allowerasing`。

## yum 源不提供 rpm 包

获取源代码后`./configure && make && make install`。

