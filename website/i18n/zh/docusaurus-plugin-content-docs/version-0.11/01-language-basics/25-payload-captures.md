# 负载捕获

负载捕获使用语法 `|value|` 并出现在许多地方，其中一些我们已经见过。无论它们出现在哪里，它们都被用来“捕获”某物的值。

使用 if 语句和可选类型。

```zig
test "optional-if" {
    var maybe_num: ?usize = 10;
    if (maybe_num) |n| {
        try expect(@TypeOf(n) == usize);
        try expect(n == 10);
    } else {
        unreachable;
    }
}
```

使用 if 语句和错误联合类型。这里需要带有错误捕获的 else。

```zig
test "error union if" {
    var ent_num: error{UnknownEntity}!u32 = 5;
    if (ent_num) |entity| {
        try expect(@TypeOf(entity) == u32);
        try expect(entity == 5);
    } else |err| {
        _ = err catch {};
        unreachable;
    }
}
```

使用 while 循环和可选类型。它可以有一个 else 块。

```zig
test "while optional" {
    var i: ?u32 = 10;
    while (i) |num| : (i.? -= 1) {
        try expect(@TypeOf(num) == u32);
        if (num == 1) {
            i = null;
            break;
        }
    }
    try expect(i == null);
}
```

使用 while 循环和错误联合类型。这里需要带有错误捕获的 else。

```zig
var numbers_left2: u32 = undefined;

fn eventuallyErrorSequence() !u32 {
    return if (numbers_left2 == 0) error.ReachedZero else blk: {
        numbers_left2 -= 1;
        break :blk numbers_left2;
    };
}

test "while error union capture" {
    var sum: u32 = 0;
    numbers_left2 = 3;
    while (eventuallyErrorSequence()) |value| {
        sum += value;
    } else |err| {
        try expect(err == error.ReachedZero);
    }
}
```

For 循环。

```zig
test "for capture" {
    const x = [_]i8{ 1, 5, 120, -5 };
    for (x) |v| try expect(@TypeOf(v) == i8);
}
```

标记联合体上的 switch cases。

```zig
const Info = union(enum) {
    a: u32,
    b: []const u8,
    c,
    d: u32,
};

test "switch capture" {
    var b = Info{ .a = 10 };
    const x = switch (b) {
        .b => |str| blk: {
            try expect(@TypeOf(str) == []const u8);
            break :blk 1;
        },
        .c => 2,
        //if these are of the same type, they
        //may be inside the same capture group
        .a, .d => |num| blk: {
            try expect(@TypeOf(num) == u32);
            break :blk num * 2;
        },
    };
    try expect(x == 20);
}
```

正如我们在上面的联合类型和可选类型部分中看到的，使用 `|val|` 语法捕获的值是不可变的（类似于函数参数），但我们可以使用指针捕获来修改原始值。
这将值捕获为本身仍然不可变的指针，但因为该值现在是一个指针，所以我们可以通过解引用来修改原始值：

```zig
test "for with pointer capture" {
    var data = [_]u8{ 1, 2, 3 };
    for (&data) |*byte| byte.* += 1;
    try expect(eql(u8, &data, &[_]u8{ 2, 3, 4 }));
}
```
