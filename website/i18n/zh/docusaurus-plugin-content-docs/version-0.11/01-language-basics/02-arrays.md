# 数组

数组用 `[N]T` 表示，其中 `N` 是数组中元素的数量，`T` 是这些元素的类型（即数组的子类型）。

对于数组字面量，`N` 可以替换为 `_` 以自动推断数组的大小。

```zig
const a = [5]u8{ 'h', 'e', 'l', 'l', 'o' };
const b = [_]u8{ 'w', 'o', 'r', 'l', 'd' };
```

要获取数组的大小，访问数组的 `len` 字段即可。

```zig
const array = [_]u8{ 'h', 'e', 'l', 'l', 'o' };
const length = array.len; // 5
```
