+++
categories = ["博文"]
date = "2016-05-30T00:00:00+08:00"
description = "Elixir tricks #1: How to pattern match empty map?"
isCJKLanguage = true
tags = ["elixir"]
title = "Elixir tricks #1: How to pattern match empty map?"
+++

**假设某函数接受一个参数映射，有时候要检查该映射是否为空，有时候则要检查该映射是否有某一个特定的键名，用模式匹配该如何做呢？**

我看到一个错误的例子：

```elixir
defmodule Example do
  # check to see if empty
  def check(%{}), do: {:empty, %{}}

  # check to see if :foo key exists
  def check(params) when :foo in Map.keys(params), do: {:found, params}

  # otherwise return it as is
  def check(params), do: {:ok, params}
end
```

这个例子比较典型，是多数入门级选手很容易写出来的代码，下面逐个分析一下：

## `%{}` 总是匹配任何映射而不是匹配空映射

```elixir
iex(1)> match? %{}, %{:foo => "foo", :bar => "bar", :baz => "baz"}
true
iex(1)> match? %{}, %{}
false
```

如上所示，如果想知道映射是不是空的，你不能草率的用 `%{}` 去做匹配，因为它能匹配任何一个映射！而且它连结构（Struct）也能匹配（但反过来不行，即你不能用结构来匹配映射）。我想这是模式匹配中的一个例外（不清楚是不是唯一一个），需要特别注意；它的好处在于可以实现针对映射的解构（Destructure）。

眼下我所知的正确姿势有下列几种：

```elixir
# use this recommended way
def check(params) when params == %{}, do: {:empty, %{}}

# or this, but not recommended
def check(params) when map_size(params) == 0, do: {:empty, %{}}
```

## 保护语句不是万能的

第二个函数定义就很有说头了。首先 `when :foo in Map.keys(params)` 是不能用作保护语句的，因为保护语句是在运行时执行的，你必须要非常小心别把耗时的表达式用在这里，上面的例子在编译时会报错如下：

> ** (ArgumentError) invalid args for operator "in", it expects a compile time list or range on the right side when used in guard expressions, got: Map.keys(params)

意思就是：如果你要在保护语句里用 `in` 操作符，那就要确保它右边的列表是一个编译时列表（即在编译时就能确定其值的列表），而上例中的 `params` 是在函数被调用时才传入的参数，因此这句表达时总是需要在运行时中进行求值——这就是一种耗时的运算。

另外这个例子也反映了作者没有理解模式匹配和保护语句的正确使用场景，其实这里才是使用模式匹配的绝佳地方：

```elixir
def check(%{foo: _} = params), do: {:found, params}
```

`%{foo: _}` 匹配参数映射里名为 `:foo` 的键，但由于其值在接下来的函数体内并无用到，所以 `_` 把它忽略掉；接着又把整个映射用 `= params` 绑定给 `params`，于是可以将其作为该函数的返回值使用。

在此处是我觉得有些别扭的地方，因为按照模式匹配的标准定义右值并不是绑定，但是在函数声明里又确实有这种用法，暂时我也没找到权威的阐述……权当 JavaScript 中的解构重命名用好了。

最后把修改后的完整例子也写出来：

```elixir
defmodule Example do
  def check(params) when params == %{}, do {:empty, params}
  def check(%{foo: _} = params), do: {:found, params}
  def check(params), do: {:ok, params}
end

# usages:
iex(1)> Example.check(%{})
{:empty, %{}}
iex(2)> Example.check(%{foo: "foo"})
{:found, %{foo: "foo"}}
iex(3)> Example.check(%{bar: "bar"})
{:ok, %{bar: "bar"}}
iex(4)> Example.check(%{foo: "foo", bar: "bar"})
{:found, %{bar: "bar", foo: "foo"}}
```
