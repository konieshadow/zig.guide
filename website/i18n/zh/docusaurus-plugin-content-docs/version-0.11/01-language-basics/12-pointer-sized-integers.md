# 指针大小的整数

`usize` 和 `isize` 以无符号和有符号整数形式给出，其大小与指针相同。

```zig
test "usize" {
    try expect(@sizeOf(usize) == @sizeOf(*u8));
    try expect(@sizeOf(isize) == @sizeOf(*u8));
}
```
