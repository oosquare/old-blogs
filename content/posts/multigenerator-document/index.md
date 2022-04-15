---
title: "MultiGenerator 使用文档"
date: 2022-04-04T22:15:15+08:00
draft: false
author: ""
authorLink: ""
description: ""
keywords: ""
license: "本文以 CC BY-NC 4.0 许可证发布"
comment: false
weight: 0

tags:
  - C++
  - OI
  - 多线程
  - MultiGenerator
  - 项目
categories:
  - Document

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

# See details front matter: /theme-documentation-content/#front-matter
---

## 概述
`MultiGenerator` 是一个为 `OI` 而生的多线程并行数据生成库，基于 `C++ 17`，使用面向对象和泛型等 `Morden C++` 高级特性，只需要添加最少的额外代码，就可以获得最高的性能。以下是一个能够指定数据范围的 `A + B Problem` 数据生成器的示例代码：

```cpp
#include <random>
#include <MultiGenerator.hpp>

using MultiGenerator::DataConfig;
using MultiGenerator::GeneratingTask;
using MultiGenerator::SolutionTask;
using MultiGenerator::NormalTemplate;
using MultiGenerator::entry;
using MultiGenerator::testcase;

/** 指定数据生成器，仅需继承一个抽象类和实现一个成员函数 */
class AddGenerator : public GeneratingTask {
private:
    void generate(std::ostream &data, const DataConfig &config) override {
        /** DataConfig 为配置信息，可以用于储存数据范围等元信息 */
        auto minValue = std::stoi(config.get("minValue").value());
        auto maxValue = std::stoi(config.get("maxValue").value());

        std::random_device rd;
        std::mt19937 gen(rd());
        std::uniform_int_distribution<> dist(minValue, maxValue);
        /** 像 cout 一样输出生成结果 */
        data << dist(gen) << " " << dist(gen) << std::endl;
    }
};

/** 指定数据求解器，也仅需继承一个抽象类和实现一个成员函数 */
class AddSolution : public SolutionTask {
private:
    /** 假如你有标程，仅需要把程序用这个类包装起来，再把 main() 改为这个成员函数即可 */
    void solve(std::istream &dataIn, std::ostream &dataOut, const DataConfig &) override {
        int a, b;
        /** 像 cin 一样读入数据 */
        dataIn >> a >> b;
        /** 像 cout 一样输出答案 */
        dataOut << a + b << std::endl;
    }
};

int main() {
    constexpr int MAX_THREAD_COUNT = 8;
    constexpr int MAX_TESTCASE_COUNT = 20;
    constexpr char PROBLEM_NAME[] = "add";
    /** 创建一个题目生成模板，指定数据文件名为 add#.in/add#.out，# 是测试点编号，可以含子任务编号 */
    MultiGenerator::NormalTemplate temp(PROBLEM_NAME);

    for (int i = 0; i < MAX_TESTCASE_COUNT; ++i) {
        /** 添加测试点配置，并指定生成器和求解器 */
        temp.add<AddGenerator, AddSolution>(testcase(i, {
            entry("minValue", i * 1000000),
            entry("maxValue", (i + 1) * 1000000)
        }));
    }
    /** 开始根据指定的线程数生成数据 */
    temp.execute(MAX_THREAD_COUNT);
    return 0;
}
```

## 要求
- `C++ 17 Compiler`
- `C++` 基础知识，包括最基本的模板的使用（基本都可以满足）
- 能够认真阅读文档

## 安装
### 编译器支持
首先确保你有支持 `C++ 17` 的编译器，如果你已经有了，可以跳过这一步。

#### Linux
绝大多数的 `Linux` 发行版预装的 `GCC` 版本都比较低，仅能支持 `C++ 11`，建议使用包管理器进行安装更新版本的 `GCC`，至少为 `GCC 8`，建议 `GCC 11`，这里仅列举部分安装方法，具体请查阅发行版的包管理器文档。

**Debian/Ubuntu/Deepin**
```sh
$ sudo add-apt-repository ppa:ubuntu-toolchain-r/test
$ sudo apt-get update
$ sudo apt install gcc g++
```

**Arch Linux/Manjaro**
```sh
$ sudo pacman -S gcc
```

**CentOS/RHEL**
```sh
$ sudo yum -y install centos-release-scl
$ yum -y install devtoolset-11-gcc devtoolset-11-gcc-c++ devtoolset-11-binutils
echo "source /opt/rh/devtoolset-11/enable" >> /etc/profile
```

#### Windows
推荐使用 `TDM-GCC`，进入下载页面 <https://jmeubank.github.io/tdm-gcc/download/>，选择 `64+32-bit` 的安装包，安装即可。

#### macOS
一般 `macOS` 都已经自带 `LLVM` 环境和 `Clang`，如果没有还可以通过安装 `Xcode` 来安装 `g++`。

### 安装 MultiGenerator
`MultiGenerator` 是一个 `header-only` 库，所以无需任何编译即可使用，只需要复制 `https://github.com/ctj12461/MultiGenerator/tree/master/src` 下的所有文件到 `include` 路径即可。

#### Linux
```sh
$ git clone https://github.com/ctj12461/MultiGenerator.git
$ sudo cp -r src/* /usr/local/include
```

如果你有 `xmake`，也可以按照如下方式安装：

```sh
$ git clone https://github.com/ctj12461/MultiGenerator.git
$ cd MultiGenerator
$ sudo xmake install --root MultiGenerator
```

#### Windows/macOS
复制 `https://github.com/ctj12461/MultiGenerator/tree/master/src` 下的所有文件到编译器提供的 `include` 路径下，具体请在编译器安装路径下搜索或查看相关文档。一个简单的办法是找到 `iostream` 的位置，然后粘贴到相同的目录下即可。

## 快速入门
### 导入库
仅需要使用 `#include <MultiGenerator.hpp>` 即可导入本库，库中所有的类和函数全部定义在 `MultiGenerator` 命名空间下，可以通过 `using namespace MultiGenerator` 来更加方便地使用本库，但是更推荐的方法是仅对有需要的类或函数使用 `using` 声明。

以下是一个简单的例子：

```cpp
#include <MultiGenerator.hpp>

using MultiGenerator::DataConfig;
using MultiGenerator::GeneratingTask;
using MultiGenerator::SolutionTask;
using MultiGenerator::NormalTemplate;
using MultiGenerator::entry;
using MultiGenerator::testcase;

int main() {
    return 0;
}
```

事实上 `MultiGenerator` 对内部实现细节做了较多的封装，如果有使用 `IDE` 的智能提示功能，可能会发现有若干子命名空间，如 `MultiGenerator::Context`，`MultiGenerator::Interface` 等等，在绝大多数情况下，您都不需要使用这些内部的组件，只需使用定义在 `MultiGenerator` 下的部分，下文也仅会介绍这一部分。

以下是本项目的目录结构。

```plain
MultiGenerator/src
├── MultiGenerator
│   ├── Context
│   │   ├── Environment.hpp
│   │   └── Stream.hpp
│   ├── Executor
│   │   ├── Channel.hpp
│   │   ├── TaskExecutor.hpp
│   │   └── ThreadPool.hpp
│   ├── Interface
│   │   ├── Component.hpp
│   │   ├── Template.hpp
│   │   └── Utility.hpp
│   ├── Variable
│   │   ├── Argument.hpp
│   │   └── DataConfig.hpp
│   └── Workflow
│       ├── Callable.hpp
│       ├── Runner.hpp
│       ├── TaskGroup.hpp
│       └── Task.hpp
└── MultiGenerator.hpp
```

### 基本概念
#### Task
在 `MultiGenerator` 中，生成的过程可以被拆分为若干个部分，每个部分有不同的功能，比如根据参数生成数据，或者读入数据并输出正确答案。这样的每个部分被成为 `Task`。

`MultiGenerator` 预定义了 3 种 `Task`，它们是更加具体的 `Task`，并规定了相关功能的接口，以便使用：

- `GeneratingTask`：表示所有生成数据的 `Task` 的抽象类，可以继承该抽象类并实现接口函数来获得生成数据的功能。
- `SolutionTask`：表示所有根据给定数据求解答案的 `Task` 的抽象类，可以继承该抽象类并实现接口函数来获得求解答案的功能。一般使用您的标准程序即 `std` 来实现。
- `IntegratedGeneratingTask`：表示所有同时生成数据和求解答案的 `Task` 的抽象类，可以用于实现必须同时生成和求解的生成器，比如一些强制在线题目。

您只需实现这些抽象类的接口，并通过给定的流进行 `IO` 操作，无需考虑文件系统相关的问题，`MultiGenerator` 可以处理它们。

#### Template
`Template` 规定了一道题目的生成程序应该如何调用 `Task`，比如普通的题目，没有强制在线操作，此时只需要分别生成数据和求解答案，就可以使用 `NormalTemplate` 来管理这些 `GeneratingTask` 和 `SolutionTask`。

`MultiGenerator` 预定义了 2 种 `Template`：

- `NormalTemplate`：可以调用实现了 `GeneratingTask` 和 `SolutionTask` 的类，用于普通的数据生成，由于生成和求解是可分离的，所以可以更好地利用系统资源进行并行优化。
- `IntegratedTemplate`：可以调用实现了 `IntegratedGeneratingTask` 的类，可以用于需要强制在线的题目的数据生成。

`Template` 可以接受一个字符串作为题目的名称，所有生成的数据都会自动带上该名称。然后您可以向 `Template` 添加测试点信息，如要使用的生成器（实现了 `GeneratingTask`）和求解器（实现了 `SolutionTask`），测试点编号，测试点的数据规模配置等。随后 `Template` 会自动将这些参数传给 `Task`，实现数据生成的定制。

如果您需要对不同的测试点应用不同的生成器，比如在使用 `NormalTemplate` 时，需要构造具有特殊性质的数据，您可以定义多个生成器，只要它们实现了 `GeneratingTask`，就可以被 `Template` 调用，而无需做任何的特殊判断。

#### testcase
`testcase` 是一个函数，可以用于生成一个测试点的配置，这个测试点可以是一个子任务中的测试点。它还接受一个 `std::unordered_map<std::string, std::string>` 作为测试点配置，这里选用 `std::string` 作为键和值是因为这样可以最简单的实现配置，假如您需要同时传入 `int` 和 `double` 的值作为配置，`std::string` 可以很好地储存它们。后面会详细介绍如何使用该函数。

#### DataConfig
`DataConfig` 储存了测试点配置信息（不包括测试点编号，事实上很少情况会需要编号，因为 `MultiGenerator` 会在外部自动处理测试点编号，无需 `Task` 内部插手），`testcase` 函数所接受的 `std::unordered_map<std::string, std::string>` 也就是构造它的参数。

`DataConfig` 不会自动转换值到您所需要的类型，它只会返回一个 `std::string`，但是转换类型大多数情况下只需要使用 `std::stoi()` 或 `std::stof()` 完成，更加高级的也仅需要使用 `std::stringstream` 即可。

### 创建 Task
`MultiGenerator` 所提供的 `Task` 已经拥有大部分功能，如如何处理文件名，如何与文件系统交互，但它唯独不知道具体该如何生成数据，所以您只需要通过继承相关 `Task` 来实现接口，从而补上缺失的一部分功能。

#### GeneratingTask
前面提到 `GeneratingTask` 表示所有生成数据的 `Task`，所以我们可以这么写：

```cpp
class MyGenerator : public GeneratingTask {
/** 这里只可以使用 private 或 protected */
private:
    /** 实现这个接口 */
    void generate(std::ostream &data, const DataConfig &config) override {
        /** 通过 DataConfig 获取配置，get 内填上自定义的键 */
        int someValue = std::stoi(config.get("some key").value());
        /** 在这里实现生成数据 */
        int someResult = someFunction();
        /** 输出结果 */
        data << someResult << std::endl;
    }

    int sumeFunction() {
        return /* ... */;
    }
};
```

如果您不了解 `C++` 的面向对象特性，您可查阅相关资料，或者直接复制上面的模板，您只要保留上述 `generate(std::ostream &data, const DataConfig &config)` 成员函数即可。

#### SolutionTask
```cpp
class MySolution : public SolutionTask {
private:
    void solve(std::istream &dataIn, std::ostream &dataOut, const DataConfig &) override {
        int someValue;
        /** 像 std::cin 一样读入数据 */
        dataIn >> someValue;
        int someResult = someFunction();
        /** 像 std::cout 一样输出答案 */
        dataOut << someResult << std::endl;
    }

    int sumeFunction() {
        return /* ... */;
    }
};
```

还是同样的道理，您只需要保留 `void solve(std::istream &dataIn, std::ostream &dataOut, const DataConfig &config)` 成员函数，并在其中填上自己的东西即可。

理论上您可以直接将 `std` 的东西复制进 `MySolution` 这个类，并把 `main()` 函数改为这个成员函数，然后做好初始化工作，因为所有的 `Task` 都是要在堆上分配内存然后运行的，所以不能够保证所有数据都和全局变量一样被初始化为 `0`。对于数组，可以将其替换为 `std::array`，它会默认初始化所有的元素为 `0`，且在使用上和原生数组没有任何差别，包括性能开销。

#### IntegratedGeneratingTask
如果您需要同时生成和求解，那 `IntegratedGeneratingTask` 会是一个很好的选择，它同时提供了两个输出流，分别连接了数据的文件（`*.in`）和答案的文件（`*.out`）。

```cpp
class MyIntegratedGenerator : public IntegratedGeneratingTask {
private:
    void generate(std::ostream &dataIn, std::ostream &dataOut, const DataConfig &config) override {
        while (/* 条件 */) {
            auto someData = /* ... */;
            auto someAnswer = /* ... */;
            dataIn << someData << std::endl;
            dataOut << someAnswer << std::endl;
        }
    }
};
```

`IntegratedGeneratingTask` 一般用于为强制在线题目或一些复杂的数据结构题目生成数据。

### 使用 testcase 创建测试点配置
`testcase` 函数可以用于创建测试点的配置，其有两个重载：

```cpp
std::shared_ptr<Variable::Argument> testcase(int id, const std::unordered_map<std::string, std::string> &config);

std::shared_ptr<Variable::Argument> testcase(int subtaskId, int id, const std::unordered_map<std::string, std::string> &config);
```

这两个函数都返回 `Variable::Argument` 的智能指针，其储存着测试点的配置参数。从函数签名可以很容易地看出第一个是用于创建无子任务的测试点，而第二个是创建有子任务的测试点。

这两个函数都在最后接受一个 `std::unordered_map<std::string, std::string>` 作为测试点配置，使用初始化列表可以很方便地传入这个参数。您还可以使用 `entry` 函数创建一个键值对，使代码更加简单易读：

```cpp
template <typename Value>
std::pair<std::string, std::string> entry(const std::string &key, const Value &value);
```

一般情况下，只需要将返回的 `Variable::Argument` 指针再传给 `Template` 即可，无需做额外的工作。

如果要创建编号为 `2`，带有 `n = 10` 且 `m = 5` 的配置，可以这样获得配置参数：

```cpp
auto arg = testcase(2, { entry("n", 10), entry("m", 5) });
```

如果要创建子任务编号为 `1`，子任务内的编号为 `5`，带有 `str = "abc"` 且 `n = 1` 的配置，可以这样获得配置参数：

```cpp
auto arg = testcase(1, 5, { entry("str", "abc"), entry("n", 1) });
```

### 把 Task 传给 Template
如上文所述，`Template` 规定了一道题目的生成程序应该如何调用 `Task`，且 `MultiGenerator` 定义了 `NormalTemplate` 和 `IntegratedTemplate`，两种 `Template` 使用方法是一样的，以下以 `NormalTemplate` 为例。

构造 `NormalTemplate` 需要传入一个字符串作为题目的名字。

```cpp
NormalTemplate temp("problem");
```

随后可以使用 `add` 成员函数创建测试点配置，签名如下：

```cpp
template <typename Generator, typename Solution>
void NormalTemplate::add(std::shared_ptr<Variable::Argument> arg);
```

其中 `Generator` 是用户自定义的实现了 `GeneratingTask` 的类，比如上文示例中的 `MyGenerator`，`Solution` 则是实现了  `SolutionTask` 的类，比如上文示例中的 `MySolution`。

`IntegratedTemplate` 的 `add` 函数签名如下：

```cpp
template <typename IntegratedGenerator>
void IntegratedTemplate::add(std::shared_ptr<Variable::Argument> arg);
```

其中 `IntegratedGenerator` 是用户自定义的实现了 `IntegratedGeneratingTask` 的类，比如上文示例中的 `MyIntegratedGenerator`。

您需要把 `testcase` 函数返回的结果传给这些函数：

```cpp
temp.add<MyGenerator, MySolution>(testcase(1, {}));
```

这样 `NormalTemplate` 就会知道要对第 1 个测试点应用 `MyGenerator` 生成数据，用 `MySolution` 求解答案，并且生成的文件为 `problem1.in` 和 `problem1.out`。

可以继续使用 `add` 函数添加测试点，使用方法是一样的，只需修改 `testcase` 函数中的测试点编号即可。注意如果测试点编号出现重复，可能会导致程序崩溃。

### 开始生成数据
这一部分很简单，只需要指定并行任务数即可：

```cpp
temp.execute(8);
```

一般并行任务数会设定为您的 `CPU` 核心数或者线程数。如果您不知道您的 `CPU` 核心数，可以使用 `std::thread::hardware_concurrency` 函数查询：

```cpp
temp.execute(std::thread::hardware_concurrency());
```

### 完整示例
最上面的示例就是一个很好的例子，参考那个即可。

## FAQ
Q1：我在 `Linux` 平台下，如果编译不能够通过，并且含有错误信息中含有 `pthread` 相关的东西，怎么解决？

A1：`Linux` 下使用多线程需要调用 `pthread`，需要给 `g++` 加上 `-pthread` 参数。