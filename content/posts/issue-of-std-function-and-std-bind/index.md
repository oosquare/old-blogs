---
title: "std::function 和 std::bind 的一个使用问题"
subtitle: ""
date: 2022-03-29T18:21:31+08:00
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
  - 标准库
  - 函数对象
  - 问题
categories:
  - 笔记

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

`std::function` 和 `std::bind` 是 `C++` 中非常常用的两个工具，然而要正确使用这两个工具还要更深入的理解。

最近写项目时遇到需要将不可复制构造的对象传给 `std::bind` 的情况，结果遇到了编译错误。代码逻辑可以抽象为下面这样：

```cpp
#include <functional>

class Class {
public:
    Class() {}

    Class(const Class &) = delete;
    Class &operator=(const Class &) = delete;

    Class(Class &&) = default;
    Class &operator=(Class &&) = default;
};

void call(std::function<void()> func) {
    func();
}

int main() {
    auto func = std::bind([](Class &) { /* code */ }, Class());
    call(std::move(func));
    return 0;
}
```

这个错误 `Language Server` 是检测不到的，只有在编译后才能发现。编译错误信息如下：

```
In file included from project.cpp:1:
In file included from /usr/bin/../lib64/gcc/x86_64-pc-linux-gnu/11.2.0/../../../../include/c++/11.2.0/functional:59:
/usr/bin/../lib64/gcc/x86_64-pc-linux-gnu/11.2.0/../../../../include/c++/11.2.0/bits/std_function.h:159:10: error: call to implicitly-deleted copy constructor of 'std::_Bind<(lambda at project.cpp:19:27) (Class)>'
            new _Functor(*__source._M_access<const _Functor*>());
                ^        ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
/usr/bin/../lib64/gcc/x86_64-pc-linux-gnu/11.2.0/../../../../include/c++/11.2.0/bits/std_function.h:196:8: note: in instantiation of member function 'std::_Function_base::_Base_manager<std::_Bind<(lambda at project.cpp:19:27) (Class)>>::_M_clone' requested here
              _M_clone(__dest, __source, _Local_storage());
              ^
/usr/bin/../lib64/gcc/x86_64-pc-linux-gnu/11.2.0/../../../../include/c++/11.2.0/bits/std_function.h:283:13: note: in instantiation of member function 'std::_Function_base::_Base_manager<std::_Bind<(lambda at project.cpp:19:27) (Class)>>::_M_manager' requested here
            _Base::_M_manager(__dest, __source, __op);
                   ^
/usr/bin/../lib64/gcc/x86_64-pc-linux-gnu/11.2.0/../../../../include/c++/11.2.0/bits/std_function.h:423:35: note: in instantiation of member function 'std::_Function_handler<void (), std::_Bind<(lambda at project.cpp:19:27) (Class)>>::_M_manager' requested here
              _M_manager = &_My_handler::_M_manager;
                                         ^
project.cpp:20:10: note: in instantiation of function template specialization 'std::function<void ()>::function<std::_Bind<(lambda at project.cpp:19:27) (Class)>, void, void>' requested here
    call(std::move(func));
         ^
/usr/bin/../lib64/gcc/x86_64-pc-linux-gnu/11.2.0/../../../../include/c++/11.2.0/functional:493:7: note: explicitly defaulted function was implicitly deleted here
      _Bind(const _Bind&) = default;
      ^
/usr/bin/../lib64/gcc/x86_64-pc-linux-gnu/11.2.0/../../../../include/c++/11.2.0/functional:412:29: note: copy constructor of '_Bind<(lambda at project.cpp:19:27) (Class)>' is implicitly deleted because field '_M_bound_args' has a deleted copy constructor
      tuple<_Bound_args...> _M_bound_args;
                            ^
/usr/bin/../lib64/gcc/x86_64-pc-linux-gnu/11.2.0/../../../../include/c++/11.2.0/tuple:744:17: note: explicitly defaulted function was implicitly deleted here
      constexpr tuple(const tuple&) = default;
                ^
/usr/bin/../lib64/gcc/x86_64-pc-linux-gnu/11.2.0/../../../../include/c++/11.2.0/tuple:599:19: note: copy constructor of 'tuple<Class>' is implicitly deleted because base class '_Tuple_impl<0, Class>' has a deleted copy constructor
    class tuple : public _Tuple_impl<0, _Elements...>
                  ^
/usr/bin/../lib64/gcc/x86_64-pc-linux-gnu/11.2.0/../../../../include/c++/11.2.0/tuple:435:17: note: explicitly defaulted function was implicitly deleted here
      constexpr _Tuple_impl(const _Tuple_impl&) = default;
                ^
/usr/bin/../lib64/gcc/x86_64-pc-linux-gnu/11.2.0/../../../../include/c++/11.2.0/tuple:408:7: note: copy constructor of '_Tuple_impl<0, Class>' is implicitly deleted because base class '_Head_base<0UL, Class>' has a deleted copy constructor
    : private _Head_base<_Idx, _Head>
      ^
/usr/bin/../lib64/gcc/x86_64-pc-linux-gnu/11.2.0/../../../../include/c++/11.2.0/tuple:86:17: note: explicitly defaulted function was implicitly deleted here
      constexpr _Head_base(const _Head_base&) = default;
                ^
/usr/bin/../lib64/gcc/x86_64-pc-linux-gnu/11.2.0/../../../../include/c++/11.2.0/tuple:125:39: note: copy constructor of '_Head_base<0, Class, true>' is implicitly deleted because field '_M_head_impl' has a deleted copy constructor
      [[__no_unique_address__]] _Head _M_head_impl;
                                      ^
project.cpp:7:5: note: 'Class' has been explicitly marked deleted here
    Class(const Class &) = delete;
    ^
1 error generated.
```

这个编译信息具有一定的误导性，有可能首先会想到的是 `std::bind` 生成的函数对象不支持复制构造和移动构造，但实际上查看源码后发现，`std::bind` 返回一个 `_Bind<_Signature>` 类，其中一个特化为：

```cpp
template <typename _Functor, typename... _Bound_args>
class _Bind<_Functor(_Bound_args...)> : public _Weak_result_type<_Functor> {
    _Functor _M_f;
    tuple<_Bound_args...> _M_bound_args;

    // ...
};
```

其中 `_M_f` 在这里是编译器将 `lambda` 表达式转换后的函数对象，复制构造和移动构造都可以支持，`_M_bound_args` 则是绑定的参数，使用 `std::tuple` 实现，其复制构造函数和移动构造函数均为 `= default`，所以至少移动构造函数也是可用的，也就是 `std::bind` 返回的这个函数对象 `_Bind<_Signature>` 也是可以移动构造的，因此 `std::move(func)` 是没有问题的。

所以问题出在 `std::function` 上，再查看 `std::function` 的源码，找到其构造函数对其他函数对象的重载：

```cpp
template <typename _Functor, typename = /* ... */, typename = /* ... */>
function(_Functor __f) : _Function_base() {
    typedef _Function_handler<_Res(_ArgTypes...), _Functor> _My_handler;

    if (_My_handler::_M_not_empty_function(__f)) {
        _My_handler::_M_init_functor(_M_functor, std::move(__f));
        _M_invoker = &_My_handler::_M_invoke;
        _M_manager = &_My_handler::_M_manager;
    }
}
```

如果 `std::function` 接受了一个函数对象，那么就会使用 `_My_handler::_M_init_functor(_M_functor, std::move(__f))` 将该函数对象复制到自身内部的 `_M_functor` 成员上，而这个函数最终会调用以下两个函数之一：

```cpp
static void _M_init_functor(_Any_data &__functor, _Functor &&__f, true_type) {
    ::new (__functor._M_access()) _Functor(std::move(__f));
}

static void _M_init_functor(_Any_data &__functor, _Functor &&__f, false_type) {
    __functor._M_access<_Functor *>() = new _Functor(std::move(__f));
}
```

事实上也只会调用以上这几个函数，这个过程也都是移动构造，理论上即使删除了复制构造函数也是可以正常工作的，其实问题出在其他函数使用了复制，比如下面这对：

```cpp
static void _M_clone(_Any_data &__dest, const _Any_data &__source, true_type) {
    ::new (__dest._M_access()) _Functor(__source._M_access<_Functor>());
}

static void _M_clone(_Any_data &__dest, const _Any_data &__source, false_type) {
    __dest._M_access<_Functor *>() = new _Functor(*__source._M_access<const _Functor *>());
}
```

这里 `__source._M_access<_Functor>()` 显然不是右值，只能调用复制构造函数。而模板实例化是全部的，不是只对有使用到的代码进行处理。所以解决方案就是不使用 `std::function`，然而只要就无法对函数签名进行限制，只能使用像下面的这种方法：

```cpp
#include <functional>

class Class {
public:
    Class() {}

    Class(const Class &) = delete;
    Class &operator=(const Class &) = delete;

    Class(Class &&) = default;
    Class &operator=(Class &&) = default;
};

template <typename Functor>
void call(Functor func) {
    func();
}

int main() {
    auto func = std::bind([](Class &) { /* code */ }, Class());
    call(std::move(func));
    return 0;
}
```