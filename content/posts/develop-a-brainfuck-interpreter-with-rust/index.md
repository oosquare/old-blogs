---
title: "用 Rust 开发 Brainfuck 解释器"
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
- Brainfuck
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

Brainfuck 是什么就不具体介绍了，可以看[这里]()。

这个解释器实现总体上是比较简单的，但是相比其他的大多数解释器还是有比较多的不同之处，具体如下：

- 更多的配置选项
  - 内存的长度与地址范围：可配置为负数
  - 单个内存单元的数据类型
  - 数据溢出处理机制：wrap 或错误
  - 读到 `EOF` 的处理机制：返回 `0`、`EOF` 本身或不改变
- 优化指令
- 采用类似编译为字节码的机制

这个项目可以算是学习 Rust 的练手项目，尝试着用了如 clap 这样的 crate。接下来就介绍一些技术细节。

## 实现原理

程序大体上是按照如下流程进行：

1. 通过 clap 解析命令行参数，获得 Brainfuck 程序代码以及解释器的配置
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