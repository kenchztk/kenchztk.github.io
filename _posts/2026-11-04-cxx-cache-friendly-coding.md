---
layout: post
title:  C++ 高性能之CPU 缓存友好代码常见套路
date:   2026-11-04 09:45:39
categories: 高性能 
tags: c/c++
---

* content
{:toc}


# C++ 高性能之CPU 缓存友好代码常见套路

在 C++ 编程中，编写**CPU 缓存友好的代码**，可以显著提高程序的性能。这是因为现代处理器在访问内存时有多级缓存（L1, L2, L3 缓存），它们的访问速度比主存快得多。**缓存友好的代码**能够有效利用 CPU 缓存，减少缓存未命中（cache miss）的频率，从而加快程序的执行速度。

### 核心思想：**局部性原理**

1. **时间局部性**：程序中近期访问的数据很可能在短期内再次被访问。
2. **空间局部性**：程序倾向于访问与当前访问数据地址相邻的数据。

以下是一些常见的优化套路，可以让 C++ 代码更符合 CPU 缓存友好的原则。

---

### 1. **连续内存分配**

使用**连续内存**结构（如数组或 `std::vector`）可以有效利用空间局部性，帮助 CPU 在加载一个缓存行时同时加载多个相邻的数据元素。

#### 示例：数组遍历

```
#include <iostream>
#include <chrono>
int main() {
	const int length = 100000000;
	// 增大数组长度以增加工作量    
	const int cache_line_size = 64 / sizeof(int);
	// 假设缓存行大小为 64 字节
	// 动态分配数组并初始化    
	int* arr = new int[length]();
	// 记录开始时间    
	auto start = std::chrono::high_resolution_clock::now();
	// 通过步长为 4 的方式访问数组，确保 CPU 缓存友好   
	for (int j = 0; j < 100; ++j) {
		// 重复多次循环以增加运行时间       
		for (int i = 0; i < length; i += 4) {
			arr[i] += 10;
			// 增加操作以模拟更复杂的工作负载        
			arr[i + 1] += 10;
			arr[i + 2] += 10;
			arr[i + 3] += 10;
		}
	}
	// 记录结束时间    
	auto stop = std::chrono::high_resolution_clock::now();
	// 计算执行时间   
	auto duration = std::chrono::duration_cast<std::chrono::milliseconds>(stop - start);
	std::cout << "Duration: " << duration.count() << " ms" << std::endl;
	delete[] arr;
	// 释放内存   
	return 0;
}
```

- **缓存行大小**：假设缓存行大小为 64 字节。`cache_line_size` 通过除以 `sizeof(int)` 计算出一个缓存行可以容纳多少个 `int` 元素（这里为 16 个 `int`）。
- **步长为 4 的访问模式**：在 `for` 循环中，以步长为 4 访问相邻的 4 个数组元素（`arr[i]`、`arr[i + 1]`、`arr[i + 2]`、`arr[i + 3]`）。这种方式能够使每次访问的多个元素都在同一缓存行中，从而充分利用 CPU 缓存，避免频繁的缓存未命中。

---

### 2. **按行优先的矩阵遍历**

在处理二维数组或矩阵时，按行遍历（Row-major order）更符合 CPU 缓存的访问模式，因为数组在内存中是按行存储的。

#### 示例：按行遍历矩阵

``` c++
#include <iostream>
#include <vector>
int main() {
	const int N = 1000;
	std::vector<std::vector<int>> matrix(N, std::vector<int>(N, 1));
	int sum = 0;
	// 按行遍历，CPU 缓存友好    
	for (int i = 0; i < N; ++i) {
		for (int j = 0; j < N; ++j) {
			sum += matrix[i][j];
		}
	}
	std::cout << "Sum: " << sum << std::endl;
	return 0;
}
```

二维数组的按行遍历更符合内存布局（Row-major），相邻的元素被存储在连续的内存中，有效利用缓存。如果改为按列遍历（Column-major），缓存命中率将降低，性能会明显下降。

---

### 3. **避免链表，使用连续容器**

链表（`std::list`）在内存中存储不连续，容易导致缓存未命中。相比之下，使用像 `std::vector` 这样连续存储的容器可以显著提高缓存友好性。

#### 示例：链表 vs 向量

``` c++
#include <iostream>#include <vector>#include <list>#include <chrono>
int main() {
	const int N = 1000000;
	std::vector<int> vec(N, 1);
	std::list<int> lst(N, 1);
	auto start = std::chrono::high_resolution_clock::now();
	int sum_vec = 0;
	for (const auto& v : vec) {
		sum_vec += v;
		// 连续内存访问，缓存友好
	}
	auto end = std::chrono::high_resolution_clock::now();
	std::cout << "Vector sum: " << sum_vec               << ", Time: " << std::chrono::duration_cast<std::chrono::microseconds>(end - start).count() << "us\n";
	start = std::chrono::high_resolution_clock::now();
	int sum_lst = 0;
	for (const auto& l : lst) {
		sum_lst += l;
		// 非连续内存访问，缓存不友好
	}
	end = std::chrono::high_resolution_clock::now();
	std::cout << "List sum: " << sum_lst               << ", Time: " << std::chrono::duration_cast<std::chrono::microseconds>(end - start).count() << "us\n";
	return 0;
}
```

由于 `std::vector` 使用连续内存，遍历时 CPU 可以高效地将一整块数据加载到缓存中。而链表的节点是分散存储的，访问时会导致频繁的缓存未命中，影响性能。

---

### 4. **结构体内数据排列优化**

如果结构体中的数据成员之间的大小差异很大，合理的内存布局可以减少内存对齐带来的浪费并提高缓存命中率。

#### 示例：优化结构体内存布局

```
#include <iostream>
struct BadStruct {
	char a;   // 1 byte    
    int b;    // 4 bytes    
    char c;   // 1 byte    
    // 会存在内存对齐问题，浪费空间
};


struct GoodStruct {
    char a;   // 1 byte    
    char c;   // 1 byte    
    int b;    // 4 bytes    
    // 内存对齐，减少空隙浪费
};

int main() {
    std::cout << "Size of BadStruct: " << sizeof(BadStruct) << std::endl;
    std::cout << "Size of GoodStruct: " << sizeof(GoodStruct) << std::endl;
    return 0;
}
```

\*\*`BadStruct` 中，由于 `char` 和 `int` 大小不一致，系统会为了对齐增加内存填充，而 `GoodStruct` 则通过调整成员的顺序，减少了对齐带来的内存浪费，也有利于更高效的缓存访问。

---

### 5. **使用 `restrict` 关键字（C++ 中的 `__restrict__`）**

在处理指针时，告诉编译器不同指针不会指向同一块内存，允许编译器进行更多的优化。

#### 示例：使用 `__restrict__`

``` c++
void copy_arrays(int* __restrict__ a, const int* __restrict__ b, size_t size) {
	for (size_t i = 0; i < size; ++i) {
		a[i] = b[i];
	}
}```

\*\*通过 `__restrict__` 告诉编译器指针 `a` 和 `b` 不会指向同一块内存区域，编译器可以对指针相关的操作进行更多优化，避免冗余的内存加载和存储操作。

---

### 6. **预取指令（Prefetching）**

手动预取可能即将访问的数据到 CPU 缓存中，尤其在数据访问模式明确的情况下，可以减少缓存未命中。

#### 示例：手动预取

``` c++
void sum_arrays(const int* a, const int* b, int* result, size_t size) {
	for (size_t i = 0; i < size; ++i) {
		_mm_prefetch((const char*)&a[i + 4], _MM_HINT_T0);
		// 预取即将访问的数据        r
        result[i] = a[i] + b[i];
	}
}
```

`_mm_prefetch` 可以提前将内存数据加载到缓存中，避免当实际需要访问时才触发缓存未命中。这里我们假设 `a[i + 4]` 是接下来会用到的数据。

---

### 编写缓存友好代码的套路

1. **使用连续内存结构**（如数组、`std::vector`），避免链表等非连续数据结构。
2. **合理的数据布局**，优化结构体的内存对齐，减少缓存行的填充。
3. **按行遍历矩阵**，确保数据访问顺序符合内存布局。
4. **减少指针混淆**，通过 `restrict` 或明确的内存访问模式，帮助编译器优化。
5. **预取指令**，在访问大量数据时提前加载到缓存中。
6. **小心动态内存分配**，尽量减少堆内存操作，避免额外的缓存负担。

---

> 原文链接：http://mp.weixin.qq.com/s?__biz=MzAwNjExNTU1MQ==&mid=2247483741&idx=1&sn=961adc71dd9d2e181a6f81addb29deb1&chksm=9a44e736b5a7d9cce918c6167aee86da163c0cfeacad0ead92a5c2b110ed8bda699e28f7b3b4&mpshare=1&scene=24&srcid=12264tiseJc503BjKg37bEFu&sharer_shareinfo=e682b09cded5b2626b86339ba505a798&sharer_shareinfo_first=e682b09cded5b2626b86339ba505a798#rd
> 收藏时间：2024-12-26 23:17:59 | 来源（wxid）：wxid_y9ar5ahmtw1c22 | 收藏项 ID：687

