# 不透明类型

不透明（[`opaque`](https://ziglang.org/documentation/master/#opaque)）类型在 Zig 中具有未知（尽管非零）的大小和对齐方式。
因此这些数据类型不能直接存储。它们用于通过指向我们没有相关信息的类型的指针来维护类型安全。

<!--fail_test-->

```zig
const Window = opaque {};
const Button = opaque {};

extern fn show_window(*Window) callconv(.C) void;

test "opaque" {
    var main_window: *Window = undefined;
    show_window(main_window);

    var ok_button: *Button = undefined;
    show_window(ok_button);
}
```

```
./test-c1.zig:653:17: error: expected type '*Window', found '*Button'
    show_window(ok_button);
                ^
./test-c1.zig:653:17: note: pointer type child 'Button' cannot cast into pointer type child 'Window'
    show_window(ok_button);
                ^
```

不透明类型可以在其定义中具有声明（与结构体、枚举和联合类型相同）。

<!--no_test-->

```zig
const Window = opaque {
    fn show(self: *Window) void {
        show_window(self);
    }
};

extern fn show_window(*Window) callconv(.C) void;

test "opaque with declarations" {
    var main_window: *Window = undefined;
    main_window.show();
}
```

不透明类型的典型用例是在与不公开完整类型信息的 C 代码互操作时维护类型安全。
