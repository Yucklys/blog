---
title: "Rust命令行界面开发入门"
date: 2020-05-06T21:06:59+08:00
draft: true
categories:
- Tutorial
tags:
- Rust
---

虽然 Rust 的 GUI 相关的库尚未完善，但是 TUI 开发的库已经相当成熟，并且也有许多应用了。**TUI**(Text-based User Interface)，即基于文本的用户界面，是指以文本符号为构成元素的界面UI，典型代表有[ranger](https://github.com/ranger/ranger)和[htop](https://hisham.hm/htop/)。

本文将会实现一个基于[tui](https://docs.rs/tui/0.9.1/tui/index.html)库的文件管理器，大约包括**界面初始化**、**绘制组件**、**按键绑定**以及一些辅助用的代码片段四部分。

<!--more-->

# 简介

在正式开始之前需要先介绍一下本文所需的库：

```toml
[dependencies]
# tui = "0.9"
# termion = "1.5"
crossterm = "0.17"
tui = { version = "0.9", default-features = false, features = ['crossterm'] }
```

tui默认使用的后端为termion，如果需要支持windows的话需要换成crossterm，并禁用tui的`default-features`。本文会以crossterm为后端讲解，不过也会将termion后端的版本放到gist中供参考。

# 界面初始化

第一步要做的就是实例化一个后端，这里用的是`CrosstermBackend`，如果你喜欢termion的话也可以换成`TermionBackend`。

```rust
use tui::backend::CrosstermBackend;
use crossterm::{execute, terminal};
use std::io::{self, Write};

fn get_backend() -> crossterm::Result<CrosstermBackend<io::Stdout>> {
    terminal::enable_raw_mode()?;
    let mut stdout = io::stdout();
    execute!(stdout, terminal::EnterAlternateScreen)?;

    let backend = CrosstermBackend::new(stdout);

    Ok(backend)
}
```

`terminal::enable_raw_mode`以及`terminal::EnterAlternateScreen`两句分别启动了终端的raw mode以及alternate screen，具体作用可参考crossterm的文档，目前只需要知道这两句能够进入一个干净的终端界面，避免之后的UI因之前的输出而变形。

```rust
use tui::Terminal;

fn main() -> crossterm::Result<()> {
	let mut terminal = Terminal::new(get_backend()?)?;
	terminal.hide_cursor()?;
    
    Ok(())
}
```

获得后端之后就可以创建`Terminal`对象，通过`Terminal`对象我们就可以开始UI绘制了。我们也可以进行一些基本的终端控制，例如示例中`hide_cursor`方法能隐藏终端光标，提高界面美观度。

```rust
use tui::{Terminal, widgets::{Block, Borders}};

fn main() -> crossterm::Result<()> {
	let mut terminal = Terminal::new(get_backend()?)?;
	terminal.hide_cursor()?;
    
    terminal.draw(|mut f| {
            let block = Block::default().title("Hello world").borders(Borders::ALL);
            f.render_widget(block, f.size());
    })?;
    
    Ok(())
}

```

这里的`mut f`即`mut Frame`。如果一切正常的话，这段代码会画出一个标题为"Hello world"的方框，那么恭喜你，你已经掌握了绘制组件的方法了！接下来我们会继续完善这个程序，让它更像一个合格的命令行程序。

`terminal.draw`这个方法只会打印一次输出，为了获得一个持续的UI界面我们需要把它放入一个循环中。

```rust
// ...
loop {
	terminal.draw(|mut f| {
        let block = Block::default().title("Hello world").borders(Borders::ALL);
        f.render_widget(block, f.size());
	})?;
    
    // keyboard interrupt
}
// ...
```

因为是loop循环，我们还需要一个跳出循环的条件。如果现在运行程序的话会发现，虽然能够持续地绘制界面，但是无法处理任何输入了。我们将在下一部分加入键盘输入的处理以处理这个问题。