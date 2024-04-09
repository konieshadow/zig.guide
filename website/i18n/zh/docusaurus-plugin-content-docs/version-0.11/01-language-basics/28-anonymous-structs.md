# 匿名结构体

结构体字面量中可以省略结构体类型。这些字面量可转换为其他结构体类型。

```zig
test "anonymous struct literal" {
    const Point = struct { x: i32, y: i32 };

    var pt: Point = .{
        .x = 13,
        .y = 67,
    };
    try expect(pt.x == 13);
    try expect(pt.y == 67);
}
```

匿名结构体可以是完全匿名的，即不转换为其他的结构体类型。

```zig
test "fully anonymous struct" {
    try dump(.{
        .int = @as(u32, 1234),
        .float = @as(f64, 12.34),
        .b = true,
        .s = "hi",
    });
}

fn dump(args: anytype) !void {
    try expect(args.int == 1234);
    try expect(args.float == 12.34);
    try expect(args.b);
    try expect(args.s[0] == 'h');
    try expect(args.s[1] == 'i');
}
```

<!-- TODO: mention tuple slicing when it's implemented -->

可以创建没有字段名称的匿名结构体，并将其称为**元组**（**tuples**）。
它们具有数组的许多特性；元组可以迭代、索引、可以与 `++` 和 `**` 运算符一起使用，并且有一个 len 字段。
在内部，它们具有从 `"0"` 开始编号的字段名称，这些字段可以通过特殊语法 `@"0"` 进行访问，该语法有转义的作用——在 `@""` 内的东西总是被识别为标识符。

必须使用内联（`inline`）循环来迭代此处的元组，因为每个元组字段的类型可能不同。

```zig
test "tuple" {
    const values = .{
        @as(u32, 1234),
        @as(f64, 12.34),
        true,
        "hi",
    } ++ .{false} ** 2;
    try expect(values[0] == 1234);
    try expect(values[4] == false);
    inline for (values, 0..) |v, i| {
        if (i != 2) continue;
        try expect(v);
    }
    try expect(values.len == 6);
    try expect(values.@"3"[0] == 'h');
}
```
