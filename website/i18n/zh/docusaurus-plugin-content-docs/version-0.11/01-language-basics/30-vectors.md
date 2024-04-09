# 向量

Zig 为 SIMD 提供向量类型。这些不能与数学意义上的向量或 C++ 中的 std::vector（为此，请参阅第二章中的“数组”）混为一谈。
向量可以使用我们之前使用过的内置 [`@Type`](https://ziglang.org/documentation/master/#Type) 来创建，而 [`std.meta.Vector`](https://ziglang.org/documentation/master/std/#std;meta.Vector) 为此提供了一个简写。

向量只能由 bool、整数、浮点数和指针作为子类型。

具有相同子类型和长度的向量之间可以进行运算。这些操作对向量中的每个值执行。
[`std.meta.eql`](https://ziglang.org/documentation/master/std/#std;meta.eql) 在此处用于检查两个向量之间的相等性（对于其他类型比如结构体也很有用）。

```zig
const meta = @import("std").meta;

test "vector add" {
    const x: @Vector(4, f32) = .{ 1, -10, 20, -1 };
    const y: @Vector(4, f32) = .{ 2, 10, 0, 1 };
    const z = x + y;
    try expect(meta.eql(z, @Vector(4, f32){ 3, 0, 20, 0 }));
}
```

向量是可索引的。

```zig
test "vector indexing" {
    const x: @Vector(4, u8) = .{ 255, 0, 255, 0 };
    try expect(x[0] == 255);
}
```

内置函数 [`@splat`](https://ziglang.org/documentation/master/#splat) 可用于构造所有值都相同的向量。这里我们用它来将向量乘以标量。

```zig
test "vector * scalar" {
    const x: @Vector(3, f32) = .{ 12.5, 37.5, 2.5 };
    const y = x * @as(@Vector(3, f32), @splat(2));
    try expect(meta.eql(y, @Vector(3, f32){ 25, 75, 5 }));
}
```

向量没有像数组那样的 `len` 字段，但仍然可以循环。

```zig
test "vector looping" {
    const x = @Vector(4, u8){ 255, 0, 255, 0 };
    var sum = blk: {
        var tmp: u10 = 0;
        var i: u8 = 0;
        while (i < 4) : (i += 1) tmp += x[i];
        break :blk tmp;
    };
    try expect(sum == 510);
}
```

向量可转换为它们对应的数组。

```zig
const arr: [4]f32 = @Vector(4, f32){ 1, 2, 3, 4 };
```

值得注意的是，如果您没有做出正确的决定，使用显式向量可能会导致软件速度变慢——编译器的自动向量化本身就相当智能。
