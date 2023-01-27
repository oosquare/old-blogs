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
license: "本文以 CC BY-NC 4.0 许可证发布"
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

Brainfuck 是什么就不具体介绍了，可以看[这里](https://esolangs.org/wiki/brainfuck)。

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
$ cargo install --path . # The program will be installed to ~/.cargo/bin
$ brainfuck-interpreter ./examples/helloworld.bf
Hello World!
```

`./examples/helloworld.bf`：

```brainfuck
++++++++++[>+++++++>++++++++++>+++>+<<<<-]>++.>+.+++++++..+++.>++.<<+++++++++++++++.>.+++.------.--------.>+.>.
```

## 实现原理

程序大体上是按照如下流程进行：

1. 通过 clap 解析命令行参数，获得 brainfuck 程序代码以及解释器的配置
2. 把代码交给 `lexer` 处理，去除无关字符，把每个字符转换成枚举，同时把一些可以合并的字符合并（如 `+++--` 合并为 `+`），获得 `TokenList`
3. 把 `TokenList` 交给 `parser` 处理，检查语法正确性并构建 AST，程序中叫 `SyntaxTree`
4. 编译出字节码
5. 构建 `Interpreter` 实例，用于执行字节码，其中包含 `Memory`，`InStream/OutStream`，`Processor`，分别负责内存读写、IO、真正执行指令
6. 中间可能返回错误，因为大多数是外部原因造成的错误，所以错误都返回到 `main()` 中处理，其实也就只能打印错误了

## 实现细节

### 词法分析

这里定义了几个类型：

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum SingleToken {
    GreaterThan,
    LessThan,
    Add,
    Sub,
    Dot,
    Comma,
    LeftBracket,
    RightBracket,
}

type SingleTokenList = Vec<SingleToken>;

#[derive(Debug, Clone, Copy, Eq, PartialEq)]
pub struct Token {
    pub token: SingleToken,
    pub count: i32,
}

#[derive(Debug, PartialEq, Eq)]
pub struct TokenList(pub Vec<Token>);
```

`SingleToken` 是分割字符再转换得到的东西，然后对 `SingleTokenList` 进行压缩得到 `TokenList`。

压缩时可以把相邻的 `+` 与 `-` 都变成一个 `+` 与其重复次数，`<` 与 `>` 同理，后续的字节码就可以直接一步到位，而不是一个一个运行。这里的重复次数可以是负数，这样可以简化整个模型。

为了后续构建 AST 更方便，我选择不合并 `[` 与 `]`，因为构建 AST 采用递归，并且是用迭代器一个一个读字符，如果合并，要么在不移动迭代器的情况下将 `count` 减一，要么一下子进行多层递归，这些都很麻烦。

这里给出部分合并代码，每一部分都是类似的：

```rust
impl TokenList {
    /// Combine the same tokens (except `[` and ']') into a `Token`
    /// which contains the count of them.
    fn combine_same(tokens: SingleTokenList) -> TokenList {
        let mut res = vec![];
        let mut last = None::<SingleToken>;
        let mut now = None::<Token>;

        for token in tokens {
            if let None = last {
                now = Some(Token::new(token, 1));
            } else if let Some(last) = last {
                if last == token
                    && token != SingleToken::LeftBracket
                    && token != SingleToken::RightBracket
                {
                    now.as_mut().unwrap().count += 1;
                } else {
                    res.push(now.take().unwrap());
                    now = Some(Token::new(token, 1));
                }
            }

            last = Some(token);
        }

        if let Some(now) = now.take() {
            res.push(now);
        }

        TokenList(res)
    }

    /// Combine `SingleToken::Add` and `SingleToken::Sub`. When it comes to
    /// `(SingleToken::Sub, 1)`, we turn it into `(SingleToken::Add, -1)`.
    fn combine_add_sub(self) -> TokenList {
        // snip
    }

    /// Combine `SingleToken::LessThan` and `SingleToken::GreaterThan`. When it comes to
    /// `(SingleToken::LessThan, 1)`, we turn it into `(SingleToken::GreaterThan, -1)`.
    fn combine_less_greater(self) -> TokenList {
        // snip
    }
}

impl From<SingleTokenList> for TokenList {
    /// Combine the similar and adjacent tokens, such as `[Token::Add, Token::Add,
    /// Token::Sub]` to `[(Token::Add, 1)]`.
    fn from(tokens: SingleTokenList) -> TokenList {
        TokenList::combine_same(tokens)
            .combine_add_sub()
            .combine_less_greater()
    }
}
```

### 语法分析

```rust
#[derive(Debug, PartialEq, Eq)]
pub enum SyntaxTree {
    Add(i32),
    Seek(i32),
    Input,
    Output,
    Root(Vec<SyntaxTree>),
    Loop(Vec<SyntaxTree>),
}
```

首先定义了 `SyntaxTree` 枚举作为 AST 的结点类型，可以发现这里又没有 `count` 来标记重复次数了，原因是 `Add` 与 `Seek` 已经优化为无需重复的版本，而如 `Input` 和 `Output`，即使放入循环也没办法优化什么，再如 `Loop` 本身作为控制流，也不可以简单重复，所以 `count` 就没有存在的必要了。

然后定义了一个 `ParseError`，这里使用了 snafu crate 来方便定义错误：

```rust
#[derive(Snafu, Debug, PartialEq, Eq)]
pub enum ParseError {
    #[snafu(display("unpaired `[` was found, and expect another `]`"))]
    UnpairedLeftBracket,
    #[snafu(display("unpaired `]` was found, and expect another `[`"))]
    UnpairedRightBracket,
}
```

接着就是解析并生成 `SyntaxTree` 了：

```rust
impl SyntaxTree {
    pub(super) fn parse(tokens: TokenList) -> Result<SyntaxTree> {
        let mut current = tokens.0.into_iter();
        let mut left_bracket_count = 0;
        let block = SyntaxTree::parse_impl(&mut current, &mut left_bracket_count)?;
        Ok(SyntaxTree::Root(block))
    }

    fn parse_impl<I>(current: &mut I, left_bracket_count: &mut i32) -> Result<Vec<SyntaxTree>>
    where
        I: Iterator<Item = Token>,
    {
        let mut res: Vec<SyntaxTree> = vec![];

        loop {
            if let Some(Token { token, count }) = current.next() {
                match token {
                    SingleToken::Add => res.push(SyntaxTree::Add(count)),
                    SingleToken::GreaterThan => res.push(SyntaxTree::Seek(count)),
                    SingleToken::Comma => {
                        for _ in 0..count {
                            res.push(SyntaxTree::Input)
                        }
                    }
                    SingleToken::Dot => {
                        for _ in 0..count {
                            res.push(SyntaxTree::Output)
                        }
                    }
                    SingleToken::LeftBracket => {
                        *left_bracket_count += 1;
                        let block = SyntaxTree::parse_impl(current, left_bracket_count)?;
                        res.push(SyntaxTree::Loop(block))
                    }
                    SingleToken::RightBracket => {
                        *left_bracket_count -= 1;
                        ensure!(*left_bracket_count >= 0, UnpairedRightBracketSnafu);
                        break;
                    }
                    // Both `SingleToken::Sub` and `SingleToken::LessThan` have been
                    // converted to `SingleToken::Add` and `SingleToken::GreaterThan`.
                    SingleToken::Sub | SingleToken::LessThan => unreachable!(),
                }
            } else {
                if *left_bracket_count == 0 {
                    break;
                } else if *left_bracket_count > 0 {
                    return Err(ParseError::UnpairedLeftBracket);
                }
                // It's impossible to reach where `left_bracket_count < 0`, for it has
                // been already checked above.
            }
        }

        Ok(res)
    }
}
```

### 运行

接下来把 `SyntaxTree` 编译为 `InstructionList`，以下为类型定义：

```rust
#[derive(Debug, PartialEq, Eq)]
pub enum Instruction {
    Add(i32),
    Seek(isize),
    Input,
    Output,
    Jump(usize), // Jump to the target without conditions
    JumpIfZero(usize), // Jump if the value in the current cell is `0`
    Halt, // Indicate the end of execution
}

#[derive(Debug, PartialEq, Eq)]
pub struct InstructionList(pub Vec<Instruction>);
```

转换的具体处理就省略了，就是简单的前序遍历。

然后来看 `Memory`:

```rust
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
```

其中的 `cur` 是相对于 brainfuck 的，如果要访问实际的内存，需要进行转换。

为了实现简单，不管对于 brainfuck 来说内存是什么数据类型，底层均使用 `Vec<i32>` 进行存储，只是在操作是加上判断。

这里为了实现多种配置，使用了策略模式，比如 `AddrStrategy` 就是表示地址范围的特征，它有两个实现 `UnsignedAddrStrategy` 与 `SignedAddrStrategy`。

```rust
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

pub struct UnsignedAddrStrategy {
    len: usize,
}

impl UnsignedAddrStrategy {
    pub fn new(len: usize) -> Self {
        Self { len }
    }
}

impl AddrStrategy for UnsignedAddrStrategy {
    fn seek(&self, addr: isize, offset: isize) -> Result<isize> {
        let target = addr + offset;

        if 0 <= target && target < self.len as isize {
            Ok(target)
        } else {
            Err(MemoryError::OutOfBounds {
                now_position: addr,
                offset,
                range: self.range(),
            })
        }
    }

    fn calc(&self, addr: isize) -> usize {
        addr as usize
    }

    fn range(&self) -> AddrRange {
        AddrRange {
            left: 0,
            right: self.len as isize - 1,
        }
    }
}

pub struct SignedAddrStrategy {
    half_len: usize,
}

impl SignedAddrStrategy {
    pub fn new(half_len: usize) -> Self {
        Self { half_len }
    }
}

impl AddrStrategy for SignedAddrStrategy {
    fn seek(&self, addr: isize, offset: isize) -> Result<isize> {
        let target = addr + offset;

        if -(self.half_len as isize) <= target && target < self.half_len as isize {
            Ok(target)
        } else {
            Err(MemoryError::OutOfBounds {
                now_position: addr,
                offset,
                range: self.range(),
            })
        }
    }

    fn calc(&self, addr: isize) -> usize {
        addr as usize + self.half_len
    }

    fn range(&self) -> AddrRange {
        AddrRange {
            left: -(self.half_len as isize),
            right: self.half_len as isize - 1,
        }
    }
}
```

其他的策略可以看[源码](https://github.com/ctj12461/brainfuck-interpreter/blob/master/src/interpreter/memory/strategy/mod.rs)。

然后就是具体执行的 `Processor`，其模拟了一个 CPU 的执行：

```rust
#[derive(PartialEq, Eq, Clone, Copy)]
pub enum ProcessorState {
    Ready,
    Running,
    Halted,
    Failed,
}

pub struct Processor {
    counter: Counter,
    instructions: InstructionList,
    memory: Memory,
    in_stream: Box<dyn InStream>,
    out_stream: Box<dyn OutStream>,
    state: ProcessorState,
}

impl Processor {
    pub fn new(
        instructions: InstructionList,
        memory: Memory,
        in_stream: Box<dyn InStream>,
        out_stream: Box<dyn OutStream>,
    ) -> Self {
        Self {
            counter: Counter::new(),
            instructions,
            memory,
            in_stream,
            out_stream,
            state: ProcessorState::Ready,
        }
    }

    pub fn counter(&self) -> usize {
        self.counter.get()
    }

    pub fn memory(&self) -> &Memory {
        &self.memory
    }

    pub fn state(&self) -> ProcessorState {
        self.state
    }

    fn abort(&mut self) {
        self.state = ProcessorState::Failed;
    }

    fn tick(&mut self) {
        self.counter.tick();
        self.check_halted();
    }

    fn check_halted(&mut self) {
        if self.instructions.0[self.counter.get()] == Instruction::Halt {
            self.state = ProcessorState::Halted;
        }
    }

    pub fn step(&mut self) -> Result<()> {
        match self.state {
            ProcessorState::Halted => return Err(ProcessorError::AlreadyHalted),
            ProcessorState::Failed => return Err(ProcessorError::Failed),
            _ => {}
        }

        match self.instructions.0[self.counter.get()] {
            Instruction::Add(val) => {
                if let Err(e) = self.memory.add(val) {
                    self.abort();
                    Err(e.into())
                } else {
                    self.tick();
                    Ok(())
                }
            }
            Instruction::Seek(offset) => {
                if let Err(e) = self.memory.seek(offset) {
                    self.abort();
                    Err(e.into())
                } else {
                    self.tick();
                    Ok(())
                }
            }
            Instruction::Input => {
                self.memory.set(self.in_stream.read());
                self.tick();
                Ok(())
            }
            Instruction::Output => {
                self.out_stream.write(self.memory.get());
                self.tick();
                Ok(())
            }
            Instruction::Jump(target) => {
                self.counter.jump(target);
                self.check_halted();
                Ok(())
            }
            Instruction::JumpIfZero(target) => {
                if self.memory.get() == 0 {
                    self.counter.jump(target);
                    self.check_halted();
                } else {
                    self.tick();
                }

                Ok(())
            }
            Instruction::Halt => {
                unreachable!()
            }
        }
    }

    pub fn run(&mut self) -> Result<()> {
        match self.state {
            ProcessorState::Halted => return Err(ProcessorError::AlreadyHalted),
            ProcessorState::Failed => return Err(ProcessorError::Failed),
            _ => {}
        }

        while self.state == ProcessorState::Ready || self.state == ProcessorState::Running {
            self.step()?
        }

        Ok(())
    }
}
```

整体上就是比较简单的模拟。对于中间出现的错误，直接返回，并且把 `Processor` 的状态设置为 `Failed`，不再可用。

有了这些，就可以组装出一个 `Interpreter`，这个也不放代码了。

### 错误处理

利用 snafu 来化简错误定义，只要通过给错误添加 `#[derive(Snafu)]`、给错误的枚举项添加 `#[snafu(display("..."))]` 就可以自动实现 `Display`，这与 thiserror 类似。

错误采用层层包装的方式传播给上层，最后在 `main()` 里全部输出。

我个人认为使用每一个模块定义一个错误并包装下层错误的方式会更好，即使是在二进制 crate 中，这样返回错误的方式能够保留错误的发出路径，而使用 `Box<dyn Error>` 或者 anyhow crate 就只能保留最底层的错误消息，缺少了层次。

比如运行 `brainfuck-interpreter --overflow error ./examples/squares.bf`，最终的错误信息是：

```plain
error: an error occured when running the code
caused by: invalid memory operation occured
caused by: 127 + 1 will overflow
```

相比一行的 `error: 127 + 1 will overflow` 就更加清晰。

为了实现这样的功能，只要在输出本层级错误信息时加上 `\ncaused by: {source}` 即可，比如下面：

```rust
#[derive(Snafu, Debug, PartialEq, Eq)]
pub enum InterpreterError {
    #[snafu(display("couldn't parse the code\ncaused by: {source}"))]
    Parse { source: ParseError },
    #[snafu(display("an error occured when running the code\ncaused by: {source}"))]
    Runtime { source: ProcessorError },
    #[snafu(display("the program hasn't been loaded yet"))]
    Uninitialized,
}
```

输出最顶层错误时用 `eprintln!("error: {e}")` 即可。

### 其他

这个解释器其实还有很多优化空间，比如可以对一些特定的 brainfuck 代码做优化，如 `[-]` 就是把当前位清零，`[->+<]` 就是把当前单元清零，并把其中的值都加到地址加一的单元里。如果完善这些优化，执行速度应该会有巨大的提升。另外可以考虑做一个 REPL，如果实现了应该就是第一个实现 brainfuck REPL 的项目了。