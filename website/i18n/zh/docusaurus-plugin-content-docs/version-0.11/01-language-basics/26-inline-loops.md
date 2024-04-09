# 内联循环

内联（`inline`）循环会被展开，并允许发生一些仅在编译时起作用的事情。这里我们使用 [`for`](https://ziglang.org/documentation/master/#inline-for)，但是 [`while`](https://ziglang.org/documentation/master/#inline-while) 的工作原理类似。

```zig
test "inline for" {
    const types = [_]type{ i32, f32, u8, bool };
    var sum: usize = 0;
    inline for (types) |T| sum += @sizeOf(T);
    try expect(sum == 10);
}
```

处于性能原因，不建议使用这些方法，除非您已经测试过显式展开的速度更快；编译器在这里往往会比您做出更好的决定。
