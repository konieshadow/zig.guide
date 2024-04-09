# 可选类型

可选类型使用语法 `?T` 存储 [`null`](https://ziglang.org/documentation/master/#null) 或 `T` 的一个值。

```zig
test "optional" {
    var found_index: ?usize = null;
    const data = [_]i32{ 1, 2, 3, 4, 5, 6, 7, 8, 12 };
    for (data, 0..) |v, i| {
        if (v == 10) found_index = i;
    }
    try expect(found_index == null);
}
```

可选类型支持 `orelse` 表达式，该表达式在值为 [`null`](https://ziglang.org/documentation/master/#null) 时起作用。这会将可选类型解包为其子类型。

```zig
test "orelse" {
    var a: ?f32 = null;
    var b = a orelse 0;
    try expect(b == 0);
    try expect(@TypeOf(b) == f32);
}
```

`.?` 是 `orelse unreachable` 的简写。当您知道可选类型不可能为空，并且使用它来展开 [`null`](https://ziglang.org/documentation/master/#null) 是可检测的非法行为时，可以使用此方法。

```zig
test "orelse unreachable" {
    const a: ?f32 = 5;
    const b = a orelse unreachable;
    const c = a.?;
    try expect(b == c);
    try expect(@TypeOf(c) == f32);
}
```

负载捕获在很多地方都适用于可选类型，这意味着如果它非空，我们可以“捕获”它的非空值。

这里我们使用 `if` 对可选类型进行负载捕获；a 和 b 在这里是等价的。`if (b) |value|` 捕获 `b` 的值（当 `b` 非空时），并使其可作为 `value` 被使用。
与联合类型中的示例一样，捕获的值是不可变的，但我们仍然可以使用指针捕获来修改存储在 `b` 中的值。

```zig
test "if optional payload capture" {
    const a: ?i32 = 5;
    if (a != null) {
        const value = a.?;
        _ = value;
    }

    var b: ?i32 = 5;
    if (b) |*value| {
        value.* += 1;
    }
    try expect(b.? == 6);
}
```

使用 `while`：

```zig
var numbers_left: u32 = 4;
fn eventuallyNullSequence() ?u32 {
    if (numbers_left == 0) return null;
    numbers_left -= 1;
    return numbers_left;
}

test "while null capture" {
    var sum: u32 = 0;
    while (eventuallyNullSequence()) |value| {
        sum += value;
    }
    try expect(sum == 6); // 3 + 2 + 1
}
```

与非可选类型相比，可选类型指针和可选类型切片不会占用任何额外的内存。
这是因为它们在内部使用指针 0 值来表示 `null`。

这就是 Zig 中空指针的工作方式——在解引用之前必须将它们解包为非可选类型，这样可以防止意外发生空指针解引用。
