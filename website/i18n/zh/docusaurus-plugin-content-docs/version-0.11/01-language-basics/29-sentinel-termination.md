# 哨兵终止

数组、切片和许多指针可以以其子类型的值终止。这称为哨兵终止（sentinel termination)。
它们遵循语法 `[N:t]T`、`[:t]T` 和 `[*:t]T`，其中 `t` 是子类型 `T` 的值。

哨兵终止数组的示例。内置的 [`@ptrCast`](https://ziglang.org/documentation/master/#ptrCast) 用于执行不安全的类型转换。
这表明数组的最后一个元素后面跟着一个值为 0 的字节。

```zig
test "sentinel termination" {
    const terminated = [3:0]u8{ 3, 2, 1 };
    try expect(terminated.len == 3);
    try expect(@as(*const [4]u8, @ptrCast(&terminated))[3] == 0);
}
```

字符串字面量的类型为 `*const [N:0]u8`，其中 N 是字符串的长度。这允许字符串字面量转换到哨兵终止的切片，并且哨兵终止可用于许多指针。
注意：字符串字面量是 UTF-8 编码的。

```zig
test "string literal" {
    try expect(@TypeOf("hello") == *const [5:0]u8);
}
```

`[*:0]u8` 和 `[*:0]const u8` 完美地模拟了 C 的字符串。

```zig
test "C string" {
    const c_string: [*:0]const u8 = "hello";
    var array: [5]u8 = undefined;

    var i: usize = 0;
    while (c_string[i] != 0) : (i += 1) {
        array[i] = c_string[i];
    }
}
```

哨兵终止类型可转换为其非哨兵终止的对应类型。

```zig
test "coercion" {
    var a: [*:0]u8 = undefined;
    const b: [*]u8 = a;
    _ = b;

    var c: [5:0]u8 = undefined;
    const d: [5]u8 = c;
    _ = d;

    var e: [:0]f32 = undefined;
    const f: []f32 = e;
    _ = f;
}
```

可使用语法 `x[n..m:t]` 创建哨兵终止切片，其中 `t` 是终止符的值。这样做表明程序员断言内存在应有的位置终止——断言失败是可检测的非法行为。

```zig
test "sentinel terminated slicing" {
    var x = [_:0]u8{255} ** 3;
    const y = x[0..3 :0];
    _ = y;
}
```
