# 命名块

Zig 中的块是表达式，可以给定标签，用于生成值。在这里，我们使用一个名为 `blk` 的标签。块产生值，这意味着它们可以用来代替值。
空快 `{}` 的类型是 `void`。

```zig
test "labelled blocks" {
    const count = blk: {
        var sum: u32 = 0;
        var i: u32 = 0;
        while (i < 10) : (i += 1) sum += i;
        break :blk sum;
    };
    try expect(count == 45);
    try expect(@TypeOf(count) == u32);
}
```

这可以看作相当于 C 的 `i++`。

<!--no_test-->

```zig
blk: {
    const tmp = i;
    i += 1;
    break :blk tmp;
}
```
