---
layout: post
title:  asio 在 win 平台错误信息的不同编码行为
date:   2026-10-20 10:36:22
categories: 通信
tags: asio
---

* content
{:toc}

Asio 在 Windows 上有多个 error_category，它们的 message() 实现方式不同：

#### 1. asio::system_category（GBK，问题根源）
Asio 有自己的 asio::detail::system_category（不是 std::system_category），其 message() 在 Windows 桌面环境下直接调用 FormatMessageA：

``` cpp
// impl/error_code.ipp — Windows desktop path
DWORD length = ::FormatMessageA(FORMAT_MESSAGE_ALLOCATE_BUFFER
    | FORMAT_MESSAGE_FROM_SYSTEM | FORMAT_MESSAGE_IGNORE_INSERTS,
    0, value, MAKELANGID(LANG_NEUTRAL, SUBLANG_DEFAULT),
    (char*)&msg, 0, 0);
```
FormatMessageA 返回的字节编码是 CP_ACP（系统默认 ANSI 代码页），中文 Windows 上就是 GBK/CP936。

#### 2. netdb_category 和 addrinfo_category（GBK，继承自 system）
``` cpp
inline const asio::error_category& get_netdb_category()
{ return get_system_category(); }  // Windows path!

inline const asio::error_category& get_addrinfo_category()
{ return get_system_category(); }  // Windows path!
```
所以它们返回的也是 GBK。在 Linux 上它们有独立的硬编码英文实现，但在 Windows 上没有。

#### 3. misc_category（ASCII，安全）
返回硬编码英文字符串（"Already open", "End of file" 等），纯 ASCII，天然 UTF-8 兼容。

#### 4. std::system_category（也是 GBK）
如果你用 std::error_code 接收非 Asio 的系统错误（比如文件 IO），MSVC STL 的 std::system_category().message() 同样用 FormatMessageW 转成 CP_ACP，还是 GBK。


