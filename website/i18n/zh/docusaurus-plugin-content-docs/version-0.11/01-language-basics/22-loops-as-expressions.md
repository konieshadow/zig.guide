# 循环作为表达式

像 `return` 一样，`break` 接受一个值。这可用于从循环中产生一个值。
Zig 中的循环还有一个 `else` 分支，当循环未通过中断退出时，将从该分支产生值。

```zig
fn rangeHasNumber(begin: usize, end: usize, number: usize) bool {
    var i = begin;
    return while (i < end) : (i += 1) {
        if (i == number) {
            break true;
        }
    } else false;
}

test "while loop expression" {
    try expect(rangeHasNumber(0, 10, 3));
}
```
