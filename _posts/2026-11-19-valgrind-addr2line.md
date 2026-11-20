---
layout: post
title:  Valgrind 泄漏地址符号还原指南（maps + addr2line）
date:   2026-11-19 10:44:19
categories: gdb
tags: addr2line
---

* content
{:toc}

> 适用场景：valgrind 报告的泄漏调用栈全是 `???`，无法定位到源码。
> 本文记录一次完整的实战过程，以 trcpfileserver 项目 26D3 分支的 valgrind 报告为例。

---

## 一、为什么会出现 `???`

valgrind 报告泄漏时（进程退出阶段），如果泄漏发生地所在的动态库已经被 `dlclose` 卸载，
或缺少调试符号，调用栈中的函数名就会显示为 `???`，例如：

```
==32956== 72 bytes in 1 blocks are definitely lost in loss record 9 of 23
==32956==    at 0x4C2C291: realloc (vg_replace_malloc.c:836)
==32956==    by 0x8A697C4: ???        ← 这里只有裸地址
==32956==    by 0x8A6A700: ???
==32956==    by 0x8368A7D: ???
```

注意：这**不是** demangle 的问题（`--demangle` 只影响"符号是乱码"的情况），
`???` 意味着 valgrind 在报告时刻根本找不到该地址对应的符号。

---

## 二、方法概览

| 方法 | 适用场景 | 优缺点 |
|---|---|---|
| **方法 A：`--keep-debuginfo=yes`** | valgrind ≥ 3.13 | ✅ 最简单，一步到位；❌ 需重跑 |
| **方法 B：抓 maps + addr2line** | 无法重跑 / valgrind 版本老 | ✅ 通用；❌ 步骤多，时机要求严格 |

**优先尝试方法 A**，一条参数解决：

```bash
valgrind --leak-check=full \
         --show-leak-kinds=definite,indirect \
         --keep-debuginfo=yes \
         --track-origins=yes \
         ./launch
```

`--keep-debuginfo=yes` 的作用：模块被 dlclose 后仍保留其符号信息，
报告中的 `???` 会直接变成 `函数名 (文件:行号)`。

以下重点记录方法 B 的完整操作流程。

---

## 三、方法 B 完整流程

### 第 1 步：在正确的时机抓取进程内存映射

**时机要求（最容易踩的坑）**：

1. 必须在**进程还活着**的时候抓——泄漏地址对应的 so 若已卸载，maps 里就查不到了；
2. 必须抓 **valgrind client 进程**的 pid（即报告里 `==32956==` 的那个数字），
   不是外层 shell 脚本的 pid。

```bash
# 启动后（进程运行中、库还加载着）执行：
cat /proc/32956/maps > /tmp/maps_running.txt
```

判断抓没抓对：maps 内容里应能看到 valgrind 的预加载库，例如：

```
/usr/libexec/valgrind/vgpreload_memcheck-amd64-linux.so
```

看到它，说明抓的是 client 进程；看不到，说明抓错 pid 了。

### 第 2 步：在 maps 中定位地址所属的 so

maps 每行的格式：

```
起始地址-结束地址 权限  文件内偏移  设备  inode      映射文件路径
08a50000-08cf6000 r-xp 002a6000 fd:02 5374857    /path/lib3rds_x64/libsqlite3mc.so
```

在 `maps_running.txt` 中查找泄漏地址落在哪个区间，**优先认 `r-xp`（可执行代码段）行**。
例：要找 `0x8A6970B`：

```
08a50000-08cf6000 r-xp ... libsqlite3mc.so
   ↑ 起点 ≤ 0x8A6970B ≤ 终点 ↑
```

→ 该地址属于 **libsqlite3mc.so**，加载基址 = `0x08a50000`。

> 技巧：泄漏调用栈里相邻的几个地址往往在同一个 so。
> 本例中 `0x8A69xxx / 0x8B3xxxx / 0x8CBxxxx` 一簇都在 libsqlite3mc.so，
> 而 `0x836xxxx` 一簇在另一行 `0834a000-083d1000 r-xp ... libfts_ToolSet.so`。

快速查找脚本：

```bash
ADDR=0x8A6970B
awk -v a=$((ADDR)) '$1 !~ /---p/ { split($1,r,/-/); s=strtonum("0x" r[1]); e=strtonum("0x" r[2]); if (a>=s && a<e) print }' /tmp/maps_running.txt
```

### 第 3 步：计算文件内偏移

```
文件内偏移 = 泄漏地址 - 加载基址
```

示例：

| 泄漏地址 | 基址 | 偏移 |
|---|---|---|
| 0x8A6970B | 0x8A50000 (libsqlite3mc.so) | 0x1970B |
| 0x8A6A700 | 0x8A50000 | 0x1A700 |
| 0x8B345EB | 0x8A50000 | 0xE45EB |
| 0x8CB8C9A | 0x8A50000 | 0x268C9A |
| 0x8368A7D | 0x834A000 (libfts_ToolSet.so) | 0x1EA7D |

> 注意：如果泄漏地址的映射行不是第一个 `r-xp` 段（比如落在了后续的 `---p`、
> `r--p` 段之间），应取该 so **第一个映射段的起始地址**作基址，
> 并确认该 so 是非 PIE 随机加载或按实际映射行计算。本例各 so 均为单段连续映射，直接相减即可。

### 第 4 步：用 addr2line 还原符号

```bash
cd build/linux/x86_64/releasedbg

# sqlite 内部分配点
addr2line -e lib3rds_x64/libsqlite3mc.so -f -C 0x1970B 0x1A700 0xE45EB 0x268C9A

# 业务侧调用点
addr2line -e libs_x64/libfts_ToolSet.so -f -C 0x1EA7D 0x177A0 0x47691
```

参数说明：

- `-e`：指定 so 文件
- `-f`：同时输出函数名
- `-C`：C++ 符号 demangle（还原 `ns::func` 形式）
- 可一次跟多个地址

### 第 5 步：得到结果

```
sqlite3MemMalloc
.../sqlite3patched.c:27012          ← sqlite 内部：内存分配器初始化
sqlite3mc_initialize
.../sqlite3mc.c:780                 ← sqlite 全局一次性初始化

mod::BackupDB::doCreateDBTables(int)
.../fts_ToolSet/BackupDB.cpp:1546   ← 业务侧：开库触发 sqlite_initialize
utils::db_utility::openDB(char const*, CppSQLite3DB*, bool)
.../utils/db_utility.h:79
```

至此 `???` 完全还原，可以回到源码定位泄漏责任方。

---

## 四、常见坑总结

1. **泄漏地址不属于任何 r-xp 段**：可能落在 `rw-p`（数据段/匿名映射），
   说明该地址是数据指针而非代码地址（如 leak record 里 `at` 那一行的 malloc 地址本身
   就是堆地址）。只有 `by` 帧里的调用返回地址才需要做符号还原。
2. **so 被 strip**：`file libXXX.so` 显示 `stripped` 时 addr2line 无效，
   需要用带调试信息的构建产物（releasedbg 版本）。
2. **多段映射的 so**：基址应取该 so 首个映射段的起始地址。

---

## 五、一句话流程图

```
valgrind 报告 ??? 地址
        │
        ▼
抓 /proc/<valgrind_client_pid>/maps   （进程运行中！）
        │
        ▼
找到地址落在的 r-xp 行 → 得到 so 路径 + 加载基址
        │
        ▼
偏移 = 泄漏地址 - 基址
        │
        ▼
addr2line -e <so> -f -C <偏移>   →   函数名 (文件:行号)
```
