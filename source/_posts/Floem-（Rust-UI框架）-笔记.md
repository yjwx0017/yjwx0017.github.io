title: Floem （Rust UI框架） 笔记
categories: 日志
tags: [Rust,Floem]
date: 2024-04-20 03:30:00
---
[https://github.com/lapce/floem][1]

- 布局系统使用 Taffy 实现 Flexbox
- 响应式设计启发自 leptos_reactive

响应性“信号”允许您以最小的努力保持UI最新，同时保持非常高的性能。

``` rust
use floem::reactive::create_signal;
use floem::view::View;
use floem::views::{h_stack, label, v_stack, Decorators};
use floem::widgets::button;

fn app_view() -> impl View {
    // 创建一个响应式信号，默认值为0
    let (counter, set_counter) = create_signal(0);

    // 创建一个垂直布局
    v_stack((
        // 标签控件上的计数文本将自动更新，得益于响应式信号
        label(move || format!("Value: {}", counter.get())),
        // 创建一个水平布局
        h_stack((
            button(|| "Increment").on_click_stop(move |_| {
                // 按钮单击时更新计数
                set_counter.update(|value| *value += 1);
            }),
            button(|| "Decrement").on_click_stop(move |_| {
                // 按钮单击时更新计数
                set_counter.update(|value| *value -= 1);
            }),
        )),
    ))
}

fn main() {
    floem::launch(app_view);
}
```

<!--more-->

``` rust
use floem::{
    peniko::Color,
    view::View,
    views::{container, stack, Decorators},
    widgets::button,
};

fn main() {
    println!("Hello, world!");

    floem::launch(app_view_0);
    //floem::launch(app_view_1);
    //floem::launch(app_view_2);
}

fn app_view_0() -> impl View {
    button(|| "Hello")
        // 处理单击事件
        .on_click_stop(|_| println!("button clicked"))
        // 设置样式
        .style(|s| {
            s.absolute()          // 使用绝对定位
                .width(100)
                .height(50)
                .margin_left(50)  // x坐标
                .margin_top(50)   // y坐标
        })
}

fn app_view_1() -> impl View {
    container({
        button(|| "按钮")
            .on_click_stop(|_| {
                println!("button clicked!");
            })
            .style(|s| s.width(100).height(100))
    })
    .style(|s| s.background(Color::GRAY).height(200).width(200))
}

fn app_view_2() -> impl View {
    // 使用 stack 容器布局，还有 h_stack v_stack 
    stack((
        button(|| "Button1"),
        button(|| "Button2"),
    ))
}
```


  [1]: https://github.com/lapce/floem