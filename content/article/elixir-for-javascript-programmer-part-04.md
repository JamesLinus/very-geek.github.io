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

筛选（`AwesomeList.filter/2`）怎么做？其实很简单，筛选就是有条件输出版本的 `AwesomeList.map/2`，唯一的区别就是我们不总是返回 `[head | filter(tail)]`，而是在 `head` 不满足条件的时候只返回 `filter(tail)`（注意，不是返回 `[filter(tail)]`，因为前面有所返回的逻辑已经保证了肯定是列表，而这里只是继续递归。如果再把递归用 `[]` 包裹起来，那么所有 `else` 分支都会被嵌套一层列表）：

```elixir
defmodule AwesomeList do
  def filter([], _fun), do: []
  def filter([head | tail], fun) do
    if fun.(head) do
      [head | filter(tail, fun)]
    else
      filter(tail, fun)
    end
  end
end
```

## 处理多维列表

假设我们有一些学生花名册的数据，需要用二维的列表来构成：

```elixir
students = [
  # [name, age, gender, grade]
  ["Ann",   16, "female", 1],
  ["Bob",   19, "male",   3],
  ["Cindy", 18, "female", 3],
  ["David", 17, "male",   2],
]
```

让我们试着找出所有三年级的学生吧（最后一个属性是年级）：

```elixir
defmodule Students do
  def for_grade_3([]), do: []
  def for_grade_3([[name, age, gender, 3] | tail]) do
    [[name, age, gender, 3] | for_grade_3([tail])
  end
end
```

以上是我们已经反复实践过多次的“一直到列表为空”的递归模式了，可是仔细想想看：我们的规则是匹配所有**四项列表且第四项是 3** 的列表，然而那些能匹配**四项列表且第四项不是 3** 的列表却被我们遗漏了。因此我们还必须加上一条函数定义：

```elixir
defmodule Students do
  def for_grade_3([]), do: []
  def for_grade_3([[name, age, gender, 3] | tail]) do
    [[name, age, gender, 3] | for_grade_3(tail)]
  end
  def for_grade_3([_ | tail]), do: for_grade_3(tail)
end
```

当然我们可以写的更明确一些，如：`def for_grade_3([[name, age, gender, _] | tail])`，但由于我们根本不关心 `head` 是什么（只是要确定有且未匹配成功）所以可以直接省略为 `_`。

接着我们通过传递第二个参数来让该函数更灵活，这也是做过很多次的事情应该难不倒你，不过请想想看要怎么往：

```elixir
defmodule Students do
  def for_grade([], _grade), do: []
  def for_grade([[name, age, gender, grade] | tail], grade) do
    [[name, age, gender, grade] | for_grade(tail, grade)]
  end
  def for_grade([_ | tail], grade), do: for_grade(tail, grade)
end
```

现在你是不是觉得二维列表的模式匹配写起来很是讨厌？甚至会想为什么不把除了 `grade` 之外的变量都忽略掉（也就是用 `_` 替代）？

这想法是非常有理的，不过还有一个小技巧会让这个想法变得更简单——连续匹配：


```elixir
defmodule Students do
  # 1st for_grade/2

  def for_grade([head = [_, _, _, grade] | tail], grade) do
    [head | for_grade(tail, grade)]
  end

  # 3rd for_grade/2
end
```

我们用占位符取代不关心的几个属性，紧接着又用 `head` 匹配了整个四项列表，于是就可以用它来指代匹配成功的列表项了。非常的优雅高效！👍🏻

最后，我们应用刚学到的知识来写一个生成顺序数字组成的列表，比如说 `AwesomeList.span(2, 6)` 会生成 `[2, 3, 4, 5, 6, 7]` 这样的列表。我们可以分别用守护语句和模式匹配来实现它：

```elixir
# 守护语句版本
defmodule AwesomeList do
  def span(from, to) when from > to, do: []
  def span(from, to),                do: [from | span(from + 1, to)]
end

# 模式匹配版本
defmodule AwesomeList do
  def span(_from = to, to), do: [to]
  def span(from, to),       do: [from | span(from + 1, to)]
end
```

守护语句版本较容易理解，而模式匹配版本简直是 Awesome！不过呢，这两个实现都不够严谨，会出现传错参数（比如 `AwesomeList.span(7, 2)` 的时候）造成死循环的情况。实际上我们可以用标准函数库来直接做到这件事情。那么下一篇我们就来看看列表的标准函数库里那些常用的函数吧。

## 尾递归优化（Tail Recursion Optimization）

最后我们来谈谈递归的一个进阶知识：尾递归优化。在我解释它是什么之前，让我们先看看之前所有递归函数里执行递归（也就是调用自己）的语句：

```elixir
# AwesomeList.len/1
1 + len(tail)

# AwesomeList.sum/1
head + sum(tail)

# AwesomeList.sum/2
sum(tail, head + acc)

# AwesomeList.square/1
[head * head | square(tail)]

# AwesomeList.map/2
[func.(head) | map(tail, func)]

# AwesomeList.reduce/3
reduce(tail, func, func.(head, acc))

# AwesomeList.max/2
when a > b, do: max([a | tail])

# AwesomeList.swap/2
[b, a | swap(tail)]
```

你看出什么蹊跷了吗？（提示：可以重点对比 `AwesomeList.sum/1` 和 `AwesomeList.sum/2`）

有些递归函数的执行就是直接调用自己，比如 `AwesomeList.sum/2`；而有的则不是直接调用自己，比如 `AwesomeList.sum/1`。说得更明确一些：递归函数的最后一个操作有的是直接调用自己，有的则是别的操作和调用自己的组合。

- `do: sum(tail, head + acc)`：这叫**最后一步操作是调用自己**
- `do: head + sum(tail)`：这就不同了，**最后一步是 `+` 调用自己的结果**

这有什么区别呢？前者可以实现“尾递归优化”而后者不能。

我们知道递归就是不断执行自身的过程，不过我们还没提到这种不断执行自身的过程会带来一个副作用，即调用栈溢出（Stack Overflow）。尾递归调用是解释器提供的一种针对调用栈的优化，它让每一次递归的结果都替换掉自己，于是在同一时刻调用栈里只有一个函数，不会出现调用栈溢出的问题。

而尾递归调用的必要条件就是**递归调用必须是最后一步操作**。理论上讲如果所有的递归函数定义都能满足尾递归调用的条件固然是完美的结果，但有的时候尾递归优化的递归函数并不是那么好写的，或者是要比普通的递归函数实现的复杂度高很多。因此是否需要尾递归优化主要还是先看你的递归函数是不是要处理特别大的集合数据（这样才会需要大数量级的调用栈堆叠）。

我们改写几个之前的递归函数让它们支持尾递归优化，以此为例来结束本篇文章吧。

```elixir
# 尾递归优化版 AwesomeList.len/2
defmodule AwesomeList do
  def len(list),                  do: len(list, 0)
  def len([], count),             do: count
  def len([_head | tail], count), do: len(tail, count + 1)
end
```

这个实现略微晦涩一些，我分别解释一下三条函数定义：

1. 首先匹配整个列表，此时暂不考虑分离 _head_ 和 _tail_ 且若匹配成功便递归并传入计算长度的初始值 `0`。于是所有的列表都会先走这一条定义然后通过递归继续匹配下去
1. 接着如果匹配到了空列表，那么直接返回当前的长度 `count`。因为第一条传递了初始值，所以这一条定义匹配成功时总是直接返回 `0` 且作为后续递归的终止条件
1. 其它情况下递归分离 _head_ 和 _tail_ （忽略不需要的 _head_ ），并且给计数加一

其实并不难理解，要点就是为了在最后一步调用自身，一切中间过程都要想办法通过递归调用传递下去罢了。

最后再来一个尾递归优化版的 `AwesomeList.square` 吧：

```elixir
# 尾递归优化版 AwesomeList.square/2
defmodule AwesomeList do
  def square(origin),              do: square(origin, [])
  def square([], list),            do: Enum.reverse(list)
  def square([head | tail], list), do: square(tail, [head * head | list])
end
```

1. 我们需要一个列表来保存每一步递归时对原始列表元素 `head` 的平方值，因此和前面的思路类似，先通过“囫囵”匹配（这是我自个儿发明的词，意思是不考虑匹配准确的结构，只要是列表就好）让递归调用传一个初始为空的列表 `list` 进去
1. 接着匹配到空列表的话就返回反转后的 `list`。有两种情况：原始列表就是空的，那么就是返回反转的空 `list`，结果还是空列表；递归一直到空的，这时的 `list` 平方已经计算完成但顺序不对（我们是从头取元素然后又从头插入，所以是逆序），翻转后正确
1. 第三条就是很常规的递归了，每次都会更新 `list`

如果你觉得 `Enum.reverse/1` 那里让你很不爽，那可以把第三条函数定义写成：

```elixir
def square([head | tail], list), do: square(tail, list ++ [head * head])
```

这样便可以去掉 `Enum.reverse/1` 了，然而我并不建议这样做，因为 `++` 填充到列表尾部操作的消耗远比插入头部大得多，这一点我们在介绍列表特性的时候已经讲过了。
