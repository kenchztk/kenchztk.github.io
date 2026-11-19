---
layout: post
title: cross_compile  
date:   2023-09-19 09:17:17
categories: c/c++
tags: cmake
---

* content
{:toc}

autotools

``` shell
./configure \
CC=/usr/gcc-linaro-5.4.1-2017.05-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-gcc \
LD=/usr/gcc-linaro-5.4.1-2017.05-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-ld \
AR=/usr/gcc-linaro-5.4.1-2017.05-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-ar \
--build=x86_64-linux-gnu \
--host=aarch64-linux-gnu \
--prefix=$(pwd)/build \
CFLAGS="-O2 -fPIC"

make -j2 CC=/usr/gcc-linaro-5.4.1-2017.05-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-gcc \
LD=/usr/gcc-linaro-5.4.1-2017.05-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-ld \
AR=/usr/gcc-linaro-5.4.1-2017.05-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-ar

make install

```

xmake

``` shell
xmake f -p linux -a arm64-v8a \
--sdk=/usr/gcc-linaro-5.4.1-2017.05-x86_64_aarch64-linux-gnu/aarch64-linux-gnu/ \
--bin=/usr/gcc-linaro-5.4.1-2017.05-x86_64_aarch64-linux-gnu/bin
```


cmake

- arch64_toolchain.cmake

``` shell
set(CMAKE_SYSTEM_NAME Linux)#交叉编译的必设参数

set(CMAKE_C_COMPILER /usr/gcc-linaro-5.4.1-2017.05-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-gcc)
set(CMAKE_CXX_COMPILER /usr/gcc-linaro-5.4.1-2017.05-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-g++)

set(CMAKE_FIND_ROOT_PATH /usr/gcc-linaro-5.4.1-2017.05-x86_64_aarch64-linux-gnu)

set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)

```

```
cmake -S $module -B build -DCMAKE_TOOLCHAIN_FILE=.aarch64_toolchain.cmake
cmake --build build
rm -rf ./build
```

```
	-DCMAKE_C_COMPILER=$CMAKE_C_COMPILER \
	-DCMAKE_CXX_COMPILER=$CMAKE_CXX_COMPILER \
	-DCMAKE_AR=$CMAKE_AR \
	-DCMAKE_FIND_ROOT_PATH=$CMAKE_FIND_ROOT_PATH
```