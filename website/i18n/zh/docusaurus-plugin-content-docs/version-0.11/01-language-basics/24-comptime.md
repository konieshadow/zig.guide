# Comptime

可以使用 [`comptime`](https://ziglang.org/documentation/master/#comptime) 关键字使块中的代码强制在编译时执行。
在此示例中，变量 x 和 y 使等效的。

```zig
test "comptime blocks" {
    var x = comptime fibonacci(10);
    _ = x;

    var y = comptime blk: {
        break :blk fibonacci(10);
    };
    _ = y;
}
```

整数字面量的类型是 `comptime_int`，它们的特殊之处在于它们没有大小（它们不能在运行时使用），并且它们具有任意精度。
 `comptime_int` 值可转换为任何可以容纳它们的整数类型。它们还可以转换为浮点数。字符字面量就是这种类型。

```zig
test "comptime_int" {
    const a = 12;
    const b = a + 10;

    const c: u4 = a;
    _ = c;
    const d: f32 = b;
    _ = d;
}
```

`comptime_float` 也是存在的，其内部为 `f128`。它们不能转换为整数，即使它们包含整数值。

Zig 中的类型是类型为 `type` 的值。这些值在编译时可用。我们之前通过检查 [`@TypeOf`](https://ziglang.org/documentation/master/#TypeOf) 并与其他类型进行比较时遇到过它们，但我们还可以做得更多。

```zig
test "branching on types" {
    const a = 5;
    const b: if (a < 10) f32 else i32 = 5;
    _ = b;
}
```

Zig 中的函数参数可以标记为 [`comptime`](https://ziglang.org/documentation/master/#comptime)。这意味着传递给该函数参数的值必须在编译时已知。让我们创建一个返回类型的函数。请注意该函数使用 PascalCase 命名，因为它返回一个类型。

```zig
fn Matrix(
    comptime T: type,
    comptime width: comptime_int,
    comptime height: comptime_int,
) type {
    return [height][width]T;
}

test "returning a type" {
    try expect(Matrix(f32, 4, 4) == [4][4]f32);
}
```

我们可以使用内置的 [`@typeInfo`](https://ziglang.org/documentation/master/#typeInfo) 来对类型使用反射，它接受一个类型并返回一个标记联合类型。这个标记联合类型可以在 [`std.builtin.TypeInfo`](https://ziglang.org/documentation/master/std/#std;builtin.TypeInfo) 中找到。

```zig
fn addSmallInts(comptime T: type, a: T, b: T) T {
    return switch (@typeInfo(T)) {
        .ComptimeInt => a + b,
        .Int => |info| if (info.bits <= 16)
            a + b
        else
            @compileError("ints too large"),
        else => @compileError("only ints accepted"),
    };
}

test "typeinfo switch" {
    const x = addSmallInts(u16, 20, 30);
    try expect(@TypeOf(x) == u16);
    try expect(x == 50);
}
```

我们可以使用 [`@Type`](https://ziglang.org/documentation/master/#Type) 函数从 [`@typeInfo`](https://ziglang.org/documentation/master/#typeInfo) 创建类型。
[`@Type`](https://ziglang.org/documentation/master/#Type) 已经为大多数类型做了实现，但对于枚举、联合类型、函数和结构体特意未实现。

这里通过 `.{}` 语法使用了匿名结构体，因为 `T{}` 中的 `T` 可以被推断出来。稍后会详细介绍匿名结构体。
在此示例中，如果未设置 `Int` 标记，我们将收到编译错误。

```zig
fn GetBiggerInt(comptime T: type) type {
    return @Type(.{
        .Int = .{
            .bits = @typeInfo(T).Int.bits + 1,
            .signedness = @typeInfo(T).Int.signedness,
        },
    });
}

test "@Type" {
    try expect(GetBiggerInt(u8) == u9);
    try expect(GetBiggerInt(i31) == i32);
}
```

返回结构体类型是在 Zig 中创建通用数据结构的方式。这里需要使用 [`@This`](https://ziglang.org/documentation/master/#This)，它获取最里面的结构体、联合类型或枚举的类型。这里还使用了 [`std.mem.eql`](https://ziglang.org/documentation/master/std/#std;mem.eql) 来比较两个切片。

```zig
fn Vec(
    comptime count: comptime_int,
    comptime T: type,
) type {
    return struct {
        data: [count]T,
        const Self = @This();

        fn abs(self: Self) Self {
            var tmp = Self{ .data = undefined };
            for (self.data, 0..) |elem, i| {
                tmp.data[i] = if (elem < 0)
                    -elem
                else
                    elem;
            }
            return tmp;
        }

        fn init(data: [count]T) Self {
            return Self{ .data = data };
        }
    };
}

const eql = @import("std").mem.eql;

test "generic vector" {
    const x = Vec(3, f32).init([_]f32{ 10, -10, 5 });
    const y = x.abs();
    try expect(eql(f32, &y.data, &[_]f32{ 10, 10, 5 }));
}
```

还可以通过使用 `anytype` 来代替函数的参数类型。然后可以在参数上使用 [`@TypeOf`](https://ziglang.org/documentation/master/#TypeOf)。

```zig
fn plusOne(x: anytype) @TypeOf(x) {
    return x + 1;
}

test "inferred function parameter" {
    try expect(plusOne(@as(u32, 1)) == 2);
}
```

Comptime 还引入了运算符 `++` 和 `**` 用于连接和复制数组与切片。这些运算符在运行时不起作用。

```zig
test "++" {
    const x: [4]u8 = undefined;
    const y = x[0..];

    const a: [6]u8 = undefined;
    const b = a[0..];

    const new = y ++ b;
    try expect(new.len == 10);
}

test "**" {
    const pattern = [_]u8{ 0xCC, 0xAA };
    const memory = pattern ** 3;
    try expect(eql(u8, &memory, &[_]u8{ 0xCC, 0xAA, 0xCC, 0xAA, 0xCC, 0xAA }));
}
```
