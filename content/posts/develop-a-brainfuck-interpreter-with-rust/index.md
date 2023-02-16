---
title: "用 Rust 开发 brainfuck 解释器"
subtitle: ""
date: 2023-01-26T22:26:51+08:00
draft: false
author: "ctj12461"
authorLink: ""
authorEmail: ""
description: ""
keywords: ""
license: "本文以 CC BY-NC-SA 4.0 许可证发布"
comment: false
weight: 0

tags:
- Rust
- brainfuck
- CLI
categories:
- Tech

hiddenFromHomePage: false
hiddenFromSearch: false

summary: ""
resources:
- name: featured-image
  src: featured-image.jpg
- name: featured-image-preview
  src: featured-image-preview.jpg

toc:
  enable: true
math:
  enable: false
lightgallery: false
seo:
  images: []

repost:
  enable: false
  url: ""

# See details front matter: https://fixit.lruihao.cn/theme-documentation-content/#front-matter
---

项目地址：<https://github.com/ctj12461/brainfuck-interpreter>

Brainfuck 是什么就不具体介绍了，可以看[这里](https://esolangs.org/wiki/brainfuck)。以下简称 bf。

这个解释器实现总体上是比较简单的，但是相比其他的大多数解释器还是有比较多的不同之处，具体如下：

- 更多的配置选项
  - 内存的长度与地址范围：可配置为负数
  - 单个内存单元的数据类型
  - 数据溢出处理机制：wrap 或错误
  - 读到 `EOF` 的处理机制：返回 `0`、`EOF` 本身或不改变
- 优化指令
- 采用类似编译为字节码的机制

这个项目可以算是学习 Rust 的练手项目，尝试着用了如 clap 这样的 crate。接下来就介绍一些技术细节。

## 使用方法

具体说明在项目 [README](https://github.com/ctj12461/brainfuck-interpreter#readme).

编译、安装、执行全过程：

```bash
$ git clone https://github.com/ctj12461/brainfuck-interpreter.git
$ cd brainfuck-interpreter
$ cargo install --path ./crates/bf-exec # The program will be installed to ~/.cargo/bin
$ bf-exec ./examples/helloworld.bf
Hello World!
```

`./examples/helloworld.bf`：

```brainfuck
++++++++++[>+++++++>++++++++++>+++>+<<<<-]>++.>+.+++++++..+++.>++.<<+++++++++++++++.>.+++.------.--------.>+.>.
```

## 实现原理

目前，项目分为两个 crate：`common` 与 `bf-exec`，其中 `common` 实现了解释器的所有逻辑，而 `bf-exec` 是 `common` 的前端，负责处理输入和配置。

`common` 的模块树如下：

- `complier`：解析代码并转换为 `IR (Intermediate Representation，中间表示)`
  - `lexer`：词法分析
  - `parser`：语法分析并优化，生成 AST `SyntaxTree`
    - `syntax`：AST 生成
    - `optimizer`：AST 优化
  - `instruction`：根据 AST 生成 `IR`，同时也是最终执行的指令
- `execution`：`IR` 的执行与相关环境
  - `memory`：按照 bf 的内存模型实现的可配置内存
    - `strategy`：基于策略模式实现的可配置组件
    - `config`：构建 `Memory` 的配置
  - `stream`：bf 的 IO 实现
    - `config`：构建 `InStream`、`OutStream` 的配置
  - `context`：`Memory` 与 `InStream`、`OutStream` 的组合
  - `processor`：运行指令，并调用 `Context`

## 实现细节

### 代码优化

#### 同质代码合并

首先最简单的就是这个优化，它是指把相邻的 `+` 与 `-`、`<` 与 `>` 合并到一起并加上一个重复次数。

为了方便，把 `-` 当作重复次数是 `-1` 的 `+`，把 `<` 当作重复次数是 `-1` 的 `>`。

这个优化虽然简单，但是非常有效，绝大多数的 bf 程序中这四个的占比是很大的，考虑到有循环，一般是多项式级别的优化。

#### 清零优化

在 bf 中，`[-]` 可以理解为把当前指针指向的数据清零，还有一个 `[+]`，但是需要使用溢出。

我们在 `SyntaxTree` 上进行操作，`SyntaxTree` 定义如下：

```rust
// crates/common/src/compiler/parser/syntax/mod.rs

#[derive(Debug, PartialEq, Eq)]
pub enum SyntaxTree {
    Add { val: i32 }, // `+`, `-`
    Seek { offset: i32 }, // `<`, `>`
    Clear,
    AddUntilZero { target: Vec<AddUntilZeroArg> },
    Input, // `,`
    Output, // `.`
    Root { block: Vec<SyntaxTree> },
    Loop { block: Vec<SyntaxTree> }, // `[]` block
}
```

考虑到还有其他的优化，我们定义一个 `trait` 用来表示优化的行为，以及 `Optimizer`，负责遍历 `SyntaxTree`。

```rust
// crates/common/src/compiler/parser/optimizer/mod.rs

pub trait Rule {
    /// Return the optimized `SyntaxTree` or the original one.
    fn apply(&self, block: SyntaxTree) -> SyntaxTree;
}

pub struct Optimizer {
    rules: Vec<Box<dyn Rule>>,
}

impl Optimizer {
    pub fn optimize(&self, mut tree: SyntaxTree) -> SyntaxTree {
        for rule in &self.rules {
            tree = rule.apply(tree);
        }

        match tree {
            SyntaxTree::Root { block } => SyntaxTree::Root {
                block: block.into_iter().map(|tree| self.optimize(tree)).collect(),
            },
            SyntaxTree::Loop { block } => SyntaxTree::Loop {
                block: block.into_iter().map(|tree| self.optimize(tree)).collect(),
            },
            otherwise => otherwise,
        }
    }

    // ...
}
```

接下来实现也很简单：

```rust
// crates/common/src/compiler/parser/optimizer/mod.rs

pub struct ClearRule;

impl ClearRule {
    pub fn new() -> Self {
        Self
    }
}

impl Rule for ClearRule {
    fn apply(&self, block: SyntaxTree) -> SyntaxTree {
        match block {
            SyntaxTree::Loop { block } => {
                if block.len() == 1 && block[0] == (SyntaxTree::Add { val: -1 }) {
                    SyntaxTree::Clear
                } else {
                    SyntaxTree::Loop { block }
                }
            }
            otherwise => otherwise,
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn clear_rule() {
        let mut optimizer = Optimizer::new();
        optimizer.add_rule(Box::new(ClearRule::new()));

        let tree = SyntaxTree::Root {
            block: vec![
                SyntaxTree::Input,
                SyntaxTree::Loop {
                    block: vec![SyntaxTree::Add { val: -1 }],
                },
            ],
        };

        let tree = optimizer.optimize(tree);

        let expected = SyntaxTree::Root {
            block: vec![SyntaxTree::Input, SyntaxTree::Clear],
        };

        assert_eq!(tree, expected);
    }

    // ...
}
```

#### 计数器优化

在 bf 中，把一个单元作为计算器，然后在一个循环中递减它并做其他操作是很正常的事，如果循环中只有 `+`、`-`、`<`、`>`，那么就可以直接计算出这个循环结束后的结果。因为计数器最终被减为 0，且在循环中都是加法，所以我称这个优化叫做 `AddUntilZeroRule`。

比如最简单的 `[->+<]`，它代表递减计数器，并每次递增计数器右边第 1 个单元。

在比如 `[->++>+<<<->]`，它代表递减计数器，并每次给计数器右边第 1 个单元加 2、给右边第 2 个单元加 1、给左边第 1 个单元减 1。

从中可以发现一个规律：每个循环体都是一个 `-` 开头，用于递减计数器，然后移动指针，对某些单元进行加减，最后指针都会被移动到计数器的位置。由此就知道如何计算循环结束后的结果了。

为了简化问题，如果循环非开头位置出现了指针指向计数器并进行操作的情况，那么就放弃优化，比如 `[-->+<]`，计数器是每次循环减 2。选择放弃优化的原因很简单，因为这会使得循环结束的判断变复杂，并且没有人会这么写。

实现如下：

```rust
// crates/common/src/compiler/parser/syntax/mod.rs

#[derive(Debug, PartialEq, Eq)]
pub struct AddUntilZeroArg {
    pub offset: isize,
    pub times: i32,
}

impl AddUntilZeroArg {
    pub fn new(offset: isize, times: i32) -> Self {
        Self { offset, times }
    }
}

// crates/common/src/compiler/parser/optimizer/mod.rs

pub struct AddUntilZeroRule;

impl AddUntilZeroRule {
    pub fn new() -> Self {
        Self
    }
}

impl Rule for AddUntilZeroRule {
    fn apply(&self, block: SyntaxTree) -> SyntaxTree {
        let block = match block {
            SyntaxTree::Loop { block } => block,
            otherwise => return otherwise,
        };

        // Check whether the first character in code is `-`.
        match block.get(0) {
            Some(SyntaxTree::Add { val: -1 }) => (),
            _ => return SyntaxTree::Loop { block },
        }

        let mut current_offset = 0;
        let mut target = Vec::with_capacity(block.len() / 2);

        for statement in block.iter().skip(1) {
            match statement {
                SyntaxTree::Add { val } => {
                    // Optimization fails if the program tries to change the
                    // counter inside a loop.
                    if current_offset == 0 {
                        return SyntaxTree::Loop { block };
                    }

                    target.push(AddUntilZeroArg::new(current_offset, *val))
                }
                SyntaxTree::Seek { offset } => current_offset += *offset as isize,
                _ => return SyntaxTree::Loop { block },
            }
        }

        // Ensure the last behavior is moving the pointer back to the place
        // where it stayed when the loop started.
        if current_offset != 0 {
            SyntaxTree::Loop { block }
        } else {
            SyntaxTree::AddUntilZero { target }
        }
    }
}

#[cfg(test)]
mod tests {
    use crate::compiler::parser::syntax::AddUntilZeroArg;

    use super::*;

    #[test]
    fn add_until_zero_rule() {
        let mut optimizer = Optimizer::new();
        optimizer.add_rule(Box::new(AddUntilZeroRule::new()));

        let tree = SyntaxTree::Root {
            block: vec![
                SyntaxTree::Loop {
                    block: vec![
                        SyntaxTree::Add { val: -1 },
                        SyntaxTree::Seek { offset: 2 },
                        SyntaxTree::Add { val: -2 },
                        SyntaxTree::Seek { offset: -3 },
                        SyntaxTree::Add { val: 1 },
                        SyntaxTree::Seek { offset: 1 },
                    ],
                },
                SyntaxTree::Loop {
                    block: vec![
                        SyntaxTree::Add { val: -1 },
                        SyntaxTree::Seek { offset: 1 },
                        SyntaxTree::Output,
                        SyntaxTree::Add { val: 1 },
                        SyntaxTree::Seek { offset: -1 },
                    ],
                },
            ],
        };

        let tree = optimizer.optimize(tree);

        let expected = SyntaxTree::Root {
            block: vec![
                SyntaxTree::AddUntilZero {
                    target: vec![AddUntilZeroArg::new(2, -2), AddUntilZeroArg::new(-1, 1)],
                },
                SyntaxTree::Loop {
                    block: vec![
                        SyntaxTree::Add { val: -1 },
                        SyntaxTree::Seek { offset: 1 },
                        SyntaxTree::Output,
                        SyntaxTree::Add { val: 1 },
                        SyntaxTree::Seek { offset: -1 },
                    ],
                },
            ],
        };

        assert_eq!(tree, expected);
    }

    #[test]
    fn add_while_zero_rule_with_changing_the_counter_incorrectly() {
        let mut optimizer = Optimizer::new();
        optimizer.add_rule(Box::new(AddUntilZeroRule::new()));

        let tree = SyntaxTree::Root {
            block: vec![SyntaxTree::Loop {
                block: vec![
                    SyntaxTree::Add { val: -1 },
                    SyntaxTree::Seek { offset: 1 },
                    SyntaxTree::Add { val: 1 },
                    // Move the pointer to the counter and change it apart from
                    // the decrement in the front of the loop.
                    SyntaxTree::Seek { offset: -1 },
                    SyntaxTree::Add { val: -1 },
                ],
            }],
        };

        let tree = optimizer.optimize(tree);

        // Failed to optimize the code and nothing changed
        let expected = SyntaxTree::Root {
            block: vec![SyntaxTree::Loop {
                block: vec![
                    SyntaxTree::Add { val: -1 },
                    SyntaxTree::Seek { offset: 1 },
                    SyntaxTree::Add { val: 1 },
                    SyntaxTree::Seek { offset: -1 },
                    SyntaxTree::Add { val: -1 },
                ],
            }],
        };

        assert_eq!(tree, expected);
    }

    // ...
}
```

### 内存

为了实现内存可配置，我使用了策略模式，定义以下 4 个 `trait`：

- `AddrStrategy`：地址范围（目前支持无符号和有符号地址范围）以及与地址和指针有关的操作
- `CellStrategy`：针对不同的大小的单元的操作
- `EofStrategy`：EOF 的处理方式
- `OverflowStrategy`：溢出的处理方式

```rust
// crates/common/src/execution/memory/mod.rs

pub type Result<T> = std::result::Result<T, MemoryError>;

#[derive(Snafu, Debug, PartialEq, Eq)]
pub enum MemoryError {
    #[snafu(display("try to seek pointer from {} to {}, which is out of [{}, {}]",
    now_position, now_position + offset, range.left, range.right))]
    OutOfBounds {
        now_position: isize,
        offset: isize,
        range: AddrRange,
    },
    #[snafu(display("{before} + {add} will overflow"))]
    Overflow { before: i32, add: i32 },
}

// ...

// crates/common/src/execution/memory/strategy/mod.rs

#[derive(Debug, PartialEq, Eq, Clone, Copy)]
pub struct AddrRange {
    pub left: isize,
    pub right: isize,
}

pub trait AddrStrategy {
    /// Return the initial value the pointer should contain.
    fn initial(&self) -> isize {
        0
    }

    /// Calculate `addr + offset`. Return `None` when `addr + offset` is out of bounds.
    fn seek(&self, addr: isize, offset: isize) -> Result<isize>;

    /// Calculate the actual address.
    fn calc(&self, addr: isize) -> usize;

    /// Get the abstract address range.
    fn range(&self) -> AddrRange;
}

pub trait CellStrategy {
    fn is_overflowed(&self, num: i64) -> bool;

    fn wrap(&self, num: i64) -> i32;
}

pub trait OverflowStrategy {
    /// Calculate and check the value for the `add` operation.
    fn add(&self, cell_strategy: &dyn CellStrategy, before: i32, add: i32) -> Result<i32>;
}

pub trait EofStrategy {
    fn check(&self, input: i8) -> Option<i8>;
}

// ...
```

具体实现见 [GitHub](https://github.com/ctj12461/brainfuck-interpreter/blob/master/crates/common/src/execution/memory/strategy/mod.rs)。

然后就可以实现内存的操作了：

```rust
// crates/common/src/execution/memory/mod.rs

pub struct Memory {
    memory: Vec<i32>,
    cur: isize,
    addr_strategy: Box<dyn AddrStrategy>,
    cell_strategy: Box<dyn CellStrategy>,
    eof_strategy: Box<dyn EofStrategy>,
    overflow_strategy: Box<dyn OverflowStrategy>,
}

impl Memory {
    pub fn new(
        addr_strategy: Box<dyn AddrStrategy>,
        cell_strategy: Box<dyn CellStrategy>,
        eof_strategy: Box<dyn EofStrategy>,
        overflow_strategy: Box<dyn OverflowStrategy>,
    ) -> Self {
        let memory = vec![0; addr_strategy.range().len()];
        let cur = addr_strategy.initial();
        Self {
            memory,
            cur,
            addr_strategy,
            cell_strategy,
            eof_strategy,
            overflow_strategy,
        }
    }

    pub fn seek(&mut self, offset: isize) -> Result<()> {
        self.cur = self.addr_strategy.seek(self.cur, offset)?;
        Ok(())
    }

    pub fn position(&self) -> isize {
        self.cur
    }

    pub fn add(&mut self, add: i32) -> Result<()> {
        let addr = self.addr_strategy.calc(self.cur);
        let target = self.memory.get_mut(addr).unwrap();
        let strategy = self.cell_strategy.as_ref();
        let res = self.overflow_strategy.add(strategy, *target, add)?;
        *target = res;
        Ok(())
    }

    pub fn set(&mut self, val: i8) {
        let addr = self.addr_strategy.calc(self.cur);
        let target = self.memory.get_mut(addr).unwrap();

        if let Some(res) = self.eof_strategy.check(val) {
            *target = res as i32;
        }
    }

    pub fn get(&self) -> i32 {
        self.memory[self.addr_strategy.calc(self.cur)]
    }
}

// ...
```

注意，这里 `Memory` 中的 `cur` 存储的是抽象层次上的指针，最终访问底层数据结构时是需要用 `AddrStragety::calc` 来换算的。

其他的实现都比较简单，本文就不再赘述了。