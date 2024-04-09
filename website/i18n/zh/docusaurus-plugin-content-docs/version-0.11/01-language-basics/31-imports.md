---
pagination_next: standard-library/allocators
---

# 导入

内置函数 [`@import`](https://ziglang.org/documentation/master/#import) 接受一个文件，并根据该文件为您提供一个结构体类型。
所有标记为 `pub`（公开）的声明都将最终在此结构体类型中，可供使用。

`@import("std")` 是编译器中的一个特殊情况，它使您可以访问标准库。其他 [`@import`](https://ziglang.org/documentation/master/#import) 将接受文件路径或包名称（稍后的章节将详细介绍包）。

我们将在后面的章节中探索更多关于标准库的内容。
