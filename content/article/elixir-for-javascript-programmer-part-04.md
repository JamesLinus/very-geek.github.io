+++
categories = ["博文"]
date = "2016-05-07T00:00:00+08:00"
description = ""
isCJKLanguage = true
tags = ["elixir"]
title = "JavaScript 程序员眼中的 Elixir：递归"
+++

老实说我一直特别害怕递归，即使我知道它的原理其实并不复杂，但我看到一些稍微复杂一些的递归函数定义我就头大😱 ，大概是大脑缺陷吧。但是自从学了 Elixir 我就不怕不怕啦～（真是脑缺……）

这篇我豁出去跟递归干上了，就让我们用前面讲到的数据结构和函数的基础知识来聊聊 Elixir 里的递归吧。

先看看下面的“预热”代码：

```elixir
iex(1)> [1 | [2 | [3 | [4 | [5 | []]]]]]
[1, 2, 3, 4, 5]
```

晕吗？挺住。我们说过了：列表都可以表示为 `[head | tail]` 的形式，`head` 是第一个值，`tail` 则是剩下的部分。由于 `tail` 也还是列表，所以它自己也可以表示为另外一个 `[head | tail]`……以此类推下去，任何列表都可以还原成上面代码的那种表达形式。

这就是递归！无论多复杂的结构都能用一个最简的形式不停重复自己直到无法再构成这个最简形式为止。下面我们就用递归的形式来实现一些常用的函数，到最后你就会觉得 `[head | tail]` 的形式无比自然。

先来一个简单的：统计列表的长度。这件事情用递归的思想来表达就是：

- 空列表的长度是 0（终止递归的条件）
- 非空列表的长度是 1 加上<列表尾的长度>
	- 因为列表尾是除去列表头的列表，所以它的长度又是 1 加上<列表尾的长度>
		- 因为列表尾是……
			- ... 

看到了吗？终有那么一刻列表尾会递归至空列表，于是只要成功匹配第一条规则就可以终止递归了。正如我们第一篇所谈过的，模式匹配在这里发挥着重要的作用。

```elixir
defmodule AwesomeList do
  def len([]),            do: 0
  def len([head | tail]), do: 1 + len(tail)
end
```

> 一些个人感悟：对我来说递归会让我头脑“卡住”的地方在于我会这么想——“OK，第一遍没问题，但是当每一次递归执行 `len/1` 的时候“+ 1”的中间结果保存在哪里了呢？

> 后来我终于认识到：没有中间结果，只有中间过程！或者你这么想：为了计算 `1 + len(tail)` 的中间结果（假设有保存中间变量这种机制的话），我们先得求出 `len(tail)` 的值然后才能 `+ 1`；那么 `len(tail)` 又会做同样的事情直到最后一层递归的 `tail` 是一个空列表这时候才会返回 `0` 给我们。

> 假设我们针对 `[1, 2, 3]` 来执行这个递归函数，那么中间的过程其实就是 `1 + 1 + 1 + 0`，也就是说还等不到你保存中间任何一个结果的时候，最终的算式就已经形成了。此时哪里还需要“保存中间结果”呢？直接就得到答案了。

递归其实就是用最笨的“公式”一步一步的计算哪怕很复杂的数据结构，让人来计算的话递归恐怕是最没效率的方式了，但是计算机恰恰最擅长这个：大量的简单重复计算。Elixir 的语法（包括多函数体、模式匹配在内）使得递归的写法非常的简单直白，对人来阅读和理解也是相对非常友好的。

> 上面的代码如果写成 .exs 文件然后编译的话会警告你说：变量 head 没有使用。可以写成 `_head` 来让 Elixir 忽略这个不用的变量。

这个例子尽管简单但却非常实用，它可以演变成很多实用的函数，比如说把列表中的每一项累加求和：

```elixir
defmodule AwesomeList do
  def sum([]),            do: 0
  def sum([head | tail]), do: head + sum(tail)
end
```
或者依次平方：

```elixir
defmodule AwesomeList do
  def square([]),            do: []
  def square([head | tail]), do: [head * head | square(tail)]
end
```

`AwesomeList.square/1` 的例子也同时展示了利用头尾构造新的列表，这很像我们使用 `Array.prototype.map` 方法所实现的结果。那么问题来了：可不可以用递归的方式创造一个更加通用的 `map` 函数呢？当然可以，只需要接收一个函数来完成具体的转换就可以了：

```elixir
defmodule AwesomeList do
  def map([], _func),           do: []
  def map([head | tail], func), do: [func.(head) | map(tail, func)]
end
```

于是我们可以用这个通用函数来做任何事情，比如还是依次平方：

```elixir
AwesomeList.map([1, 2, 3], &(&1 * &1))
[1, 4, 9]
```

> 为免初学者不能理解：`&(&1 * &1)` 等价于 `fn (n) -> n * n end`。

或者来利用标准函数库里的函数，比如说把所有单词首字母大写：

```elixir
AwesomeList.map(["apple", "banana", "cherry"], &(String.capitalize/1))
["Apple", "Banana", "Cherry"]
```

对比一下 [JavaScript 对 `map` 函数的实现](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map#Polyfill)（引用自 Mozilla Developer Network），你会发现递归的实现是多么美好呀～😻 

## 传递而不是暂存

前面我们写了一个很简单的累加求和，然而它的局限性很大，因为它是“暗示”从零开始做累加的，如果我们希望从一个非零的数字开始怎么做呢？实际上这个要求就等于要做一个更加通用的 `Array.prototype.reduce` 函数了，但是让我们放缓脚步一步一步来。

第一，我们让累加函数接收一个初识累加值 _accumulator_ ，我们并不会修改它（记住不可变数据）而是通过递归把它传递下去：

```elixir
defmodule AwesomeList do
  def sum([], acc),            do: acc
  def sum([head | tail], acc), do: sum(tail, head + acc)
end
```

> 这个累加和之前的累加实现的是一样的功能，但不同的是使用了一个聚合器（如果你给初识值为 `0` 的话）。所以说在很多情况下，递归实现是不需要中间值的。

第二，我们借助前面实现 `AwesomeList.map/2` 的经验来实现 `AwesomeList.reduce/3`，它要接收的三个参数是：列表，处理函数和聚合器（如累加器）：

```elixir
defmodule AwesomeList do
  def reduce([], _func, acc),           do: acc
  def reduce([head | tail], func, acc), do: reduce(tail, func, func.(head, acc))
end
```

我们用 `func.(head, acc)` 替代了之前的 `head + acc`，于是就让 `reduce` 不仅限于累加而是取决于 `func` 能做什么。我们来“累减”玩玩？

```elixir
AwesomeList.reduce([1, 2, 3, 4], 10, &(&2 - &1))
```

**请特别注意 `&(&2 - &1)` 这里两个参数反过来写的，**这是因为 `AwesomeList.reduce/3` 的实现里我们执行 `func.(head, acc)` 的传參顺序用来做累减需要反过来用 `acc` 去减 `head` 才能得到正确的结果。

## 复杂一些的列表模式匹配

不是所有的列表处理都是一次一个就可以完成的，我们试一个典型的例子：两两比较，直到找出列表中的最大项。

首先，如果处理的列表为空，那么应该返回一个无效的结果，我们简单的返回 `nil` 好了：

```elixir
defmodule AwesomeList do
  def max([]), do: nil
end
```

其次，如果处理的列表只有一项，那就直接返回它：


```elixir
defmodule AwesomeList do
  def max([]),  do: nil
  def max([x]), do: x
end
```

接下来是我们尚未探索过的特性。我们知道任何列表都可以表达为：`[head | tail]`，不过模式匹配的时候联结操作符（也就是 `|`）可以分割不只一项，也就是说如果列表项超过一项的时候，我们也可以表达为：`[a, b | tail]`，于是分割的时候就不止取一个 `head` 而是前面的两项。于是当我们有两项被匹配出来之后，我们就可以使用守护语句来比较它们的大小：

```elixir
defmodule AwesomeList do
  def max([]),                       do: nil
  def max([x]),                      do: x
  def max([a, b | tail]) when a > b, do: max([a | tail])
end
```

如果第三条定义也无法匹配成功就说明 `b >= a`，所以我们直接忽略 `a` 就好了：

```elixir
defmodule AwesomeList do
  def max([]),                       do: nil
  def max([x]),                      do: x
  def max([a, b | tail]) when a > b, do: max([a | tail])
  def max([_, b | tail]),            do: max([b | tail])
end
```

这个实现可以工作，不过我们还可以利用标准函数让它变得更简洁：

```elixir
defmodule AwesomeList do
  def max([]),            do: nil
  def max([x]),           do: x
  def max([head | tail]), do: Kernel.max(head, max(tail))
end
```
因为 `Kernel.max/2` 会帮你完成大小比较的事情，所以便不需要自己去写守护语句了。当然不总是有现成的函数可以用，比如说两两交换吧：

```elixir
defmodule AwesomeList do
  def swap([]), do: []
  def swap([a, b | tail]), do: [b, a | swap(tail)]
end
```

这个函数的实现有一个最大的问题：如果你传入奇数个数的列表，那么递归到最终会出现只剩一个的情况，因为我们没有处理这种情况的定义于是就抛出错误了。

```elixir
defmodule AwesomeList do
  def swap([]),            do: []
  def swap([a, b | tail]), do: [b, a | swap(tail)]
  def swap([a]),           do: [a]
end
```

后两个函数定义的顺序会影响最终结果，所以要注意模式匹配的顺序问题。
