+++
categories = ["博文"]
date = "2016-05-06T13:03:20+08:00"
description = ""
isCJKLanguage = true
tags = ["elixir"]
title = "JavaScript 程序员眼中的 Elixir：函数"
+++

这是一个 Elixir 的匿名函数以及执行它的方式：

```elixir
iex(1)> sum = fn a, b -> a + b end
#Function<12.50752066/2 in :erl_eval.expr/5>
iex(2)> sum.(1, 2)
3
```

在区别它和 JavaScript 的匿名函数之前，我更倾向于先理解函数的本质。因为差别之处只在于语法细节，而本质才有助于真正理解函数自身。

如果你问我 JavaScript 的函数究竟是什么，我会说函数就是一个对象，下面几句代码在本质上是一样（箭头函数的特性不在本篇的讨论范围之内）的：

```javascript
const sum = (a, b) => a + b
// or
const sum = function(a, b) {
  return a + b
}

// vs.

const sum = new Function('a', 'b', 'return a + b');
```

因为函数就是 `Function` 对象，所以每一个函数都拥有 `Function.prototype` 的成员，诸如 `call`，`apply` 等等。

那么对象又是什么？可以在 Elixir 中找到对应的东西吗？

无论高级语言之间如何不同（比如 OO 范式与 FP 范式之间的差异），它们最终都会变成二进制的代码交由计算机来执行，所有的“数据”终究都是存储介质里的值。对象就是内存里的值，只不过它具有特定的且相对复杂的结构（相对于原始类型而言）；函数式语言不谈对象，但是它也有特定的复杂结构，所以本质上也会像对象一样最终表现为内存中的值。

现在，对比一下这两句代码：`"Elixir is awesome"` 和 `fn name -> "#{name} is awesome" end`，你如何看待它们的异同？

从某种抽象层次来说，它们没什么不同——都是内存里的值。可以这么说：

- 我们用 `""` 来表示这是一个代表了字符串的值，其内容为 `Elixir is awesome`；因为英文字母和单词可以在语言里表示很多种意思，为了让解释器知道这是字符串而不是别的什么所以才用 `""` 来表示，于是 Elixir 知道如何用正确的底层代码来表示它。

- 类似的，我们用 `fn...end` 来表示这是一个匿名函数，其内容是 `parameter-list -> function body`，这个内容比字符串复杂一些，它用 `->` 分割成两个部分：左边是**参数列表**，右边是**函数体**。我们不用关心在底层是用什么方式来表示这些概念的，但是终究它们还是值而已。

现在我可以很有信心的去学习函数在 Elixir 中的各种细节了，因为在本质上我已经在大脑中建立了一种映射模型，任何复杂一些的、会绕弯的细节我都可以对照其在 JavaScript 中的相同之处；如果有任何地方是用 JavaScript 的函数所无法解释的，我就会提高映射的抽象层次去设想它（们）的本质究竟为何——这是非常适合我的学习方式。

好，因为函数就是值，所以我们可以把函数作为值传递给其他的函数，也可以作为值从其他的函数中返回，当然也可以执行它并为它传值（因为这个值有接收参数列表的内容构造）。这话听着耳熟吧？当我们谈及 “JavaScript 中的函数是一等公民”时，说的就是这些事情。唯一的区别就在于 Elixir 的函数不是对象（因为我们在函数式语言里没有对象这一层抽象），但终究和对象一样也是值而已。

## 函数和模式匹配

还是看这个例子：

```elixir
iex(1)> sum = fn a, b -> a + b end
#Function<12.50752066/2 in :erl_eval.expr/5>
iex(2)> sum.(1, 2)
3
```

当我们调用该函数并传参时，很容易会想成：“传实参 `1` 和 `2` 进去，**赋值**给形参 `a` 和 `b`”……哦，等等！看到赋值俩字吗？这是一个“危险”的信号。

在 JavaScript 中，形式参数等于本地变量的名字，实际参数等于本地变量的值。当函数被调用时，的确在函数体内有隐式的赋值行为。但是在 Elixir 中并不是这样的，永远没有赋值！上面的行为类似于 `{a, b} = {1, 2}` 这样的模式匹配。

这有啥不同呢？不同之处就在于我们可以做这种事情：

```elixir
iex(1)> swap = fn {x, y} -> {y, x} end
#Function<6.50752066/1 in :erl_eval.expr/5>
iex(2)> swap.({0, 1})
{1, 0}
```

当然 JavaScript 有了解构之后也可以做类似的事情：

```javascript
const swap = ([x, y]) => [y, x]
swap([0, 1])
// => [1, 0]
```

不过[解构与模式匹配如何不同已经在前文里讲过](/article/elixir-for-javascript-programmer-part-01)，所以 JavaScript 在这方面并不能比肩于 Elixir。

## 一个函数，多个函数体

下面是在 Node.js 中（异步）读取一个本地文件内容的代码：

```javascript
fs.readFile('config.toml', (err, data) => {
  if err throw err
  console.log(data)
})
```

指定文件能否正确读取是无法提前确定的，因此回调函数会将 `err` 作为第一个参数传进来，于是我们需要先看看有没有错误，有的话就要优先处理它，没有的话才能“自信的”处理实际的文件内容。这种“错误优先”的回调函数是 Node.js 的主流风格，本质上就是一种“事前约定”的分支处理模式。

在 Elixir 中，类似的处理会更倾向于使用模式匹配来做，并且能带给我们非常独特的“一个函数，多个函数体”的语法风格：

```elixir
read_file = fn
  {:ok, data} -> "Read data as: #{IO.read(data, :line)}"
  {_, error} -> "Error as: #{:file.format_error(error)}"
end

read_file.(File.open("config.toml"))
# "Read data as: baseurl = \"http://very-geek.github.io/\"\n"

# or

read_file.(File.open("config.yaml"))
# "Error as: no such file or directory"
```

我觉得这种风格非常有趣且实用：我们不去判断有没有错误而是直接模式匹配入参，匹配成功就会执行对应的函数体。我们曾经说过 Elixir 的世界里函数的行为都遵循着一些“约定”，特别是用元组来做返回值这个约定尤其典型。在这种风格里无所谓是“错误优先”还是“正确优先”，每次执行都会匹配到唯一的结果，不需要判断或者提前处理，代码变得更容易读写和理解，执行的效率也更高了。

### 继承 Erlang 的丰饶“财产”

在上例中我们使用了 `File.open/2`，也使用了 `:file.format_error/1`，这两者之间有什么区别呢？`File` 是 Elixir 的模块，而 `:file` 是 Erlang 的 `File` 模块，也就是说因为 Elixir 构建于 Erlang VM 之上，所以我们可以直接访问底层虚拟机所提供的 APIs，Elixir 缺省使用同名小写的原子来代表 Erlang 里的同名模块。

### 留心多个函数体的匹配顺序

下面是我做的第一个 Elixir 的练习，经典的 “FuzzBuzz” 程序：

```elixir
fizz_buzz = fn
  {0, 0, _} -> IO.puts "FizzBuzz."
  {0, _, _} -> IO.puts "Fizz."
  {_, 0, _} -> IO.puts "Buzz."
  {_, _, n} -> IO.puts n
end

take_number = fn
  n -> fizz_buzz.({rem(n, 3), rem(n, 5), n})
end

Enum.each 10..16, fn n -> take_number.(n) end
```

这个程序很“纯真”😹 ，但是这里面有一个特别容易被初学者踩的坑。如果我们调换一下多个函数体的顺序：

```elixir
fizz_buzz = fn
  {0, _, _} -> IO.puts "Fizz."
  {0, 0, _} -> IO.puts "FizzBuzz."
  {_, 0, _} -> IO.puts "Buzz."
  {_, _, n} -> IO.puts n
end
```

运行的结果就完全错误了。很显然如果两个待匹配且有多个值的模式在匹配时是“谁先匹配谁胜出”的，所以一定要小心这种情况。当然了，这种解法其实并不是最好的选择，以后我们再谈更合适的写法，所以这种问题是可以从根本上避免的。

## 捕获操作符（Capture Operator，`&`）

Elixir 使用一个宏 `&` 来简化匿名函数的创建与使用，这个特性在最初会让人晕乎乎一阵子，不过一旦建立起了坚实的思维模型就不难理解了。

捕获操作符实际上有两种（非常相似的）用法，第一种就是捕获一个既存的函数：

```elixir
iex(1)> nil? = &Kernel.is_nil/1
#Function<6.50752066/1 in :erl_eval.expr/5>
iex(2)> nil?.(nil)
true
iex(4)> nil?.("nil")
false
```

由于捕获后的结果是一个匿名函数，所以调用的时候要留意调用时的语法。

JavaScript 没有捕获操作符，上面的例子要用 JavaScript 来类比的话是这样的：

```javascript
const isArray = target => Array.isArray(target)
isArray([]) // => true
isArray("") // => false
```

另外一种用法实际上就是创建匿名函数，主要用于函数的局部应用（Partially Apply）；此时 `&1`，`&2`, ..., `&n` 用来做参数的占位符：

```elixir
# 以下是类似 JavaScript Array.prototype.map 的用法
Enum.map [1, 2, 3], &(&1 * &1)
# [1, 4, 9]

# 如果觉得脑袋转不过弯来，那么以下是等价的写法
Enum.map [1, 2, 3], fn n -> n * n end
# [1, 4, 9]

# 或者可以写得更明确（以及具备可复用性）些：
square = &(&1 * &1)
Enum.map [1, 2, 3], square
# [1, 4, 9]
```

所以呢，`&(&1 * &1)` 就等于 `fn n -> n * n end` 而已，这种转换多玩几次就利索了。再看一个高阶函数的例子：

```elixir
double = &(&1 * 2)
apply = &(&1.(&2))
apply.(double, 3)
# 6
```

简直 Amazing 了～，等价的匿名函数写法如下：

```elixir
double = fn n -> n * 2 end
apply = fn (func, value) -> func.(value) end
apply.(double, 3)
# 6
```

等价的 JavaScript 写法：

```javascript
const double = n => n * 2
const apply = (func, value) => func(value)
apply(double, 3)
// => 6
```

我一遍遍的重复是因为在实战中捕获操作符用得很多，所以尽早的把它变成一种“本能”会让你后续的学习变得更加平顺。

## 模块与具名函数

我们一直在说匿名函数，具名函数又如何呢？和 JavaScript 一样，函数在 Elixir 中也有具名与匿名之分，但是具名函数在 Elixir 中不能随处定义，它们只能定义在模块（Module）之中。

当代码的规模开始增长，我们就需要更好的组织代码方式。OO 语言有封装的概念，不管是基于类（Class）还是基于原型（Prototype），本质上都还是组织代码的形式。只不过封装这个概念具有包裹和控制“状态”的意图在里面，而 Elixir 的模块相对单纯一些，因为不可变数据的特点决定了没有状态管理这么回事儿。

具名函数在模块之中扮演了“代码逻辑分割者”的角色，最基本的例子如下：

```elixir
defmodule Calculator do
  def sum(a, b) do
    a + b
  end
end
```

和 Ruby 简直太像了不是么？

模块扮演的是“命名空间”的角色，上面的例子我们可以用 `Calculator.sum/2` 的签名形式来代表，后面的 `/2` 指的是 `Calculator.sum` 函数接收的参数个数。 在 Elixir 中，具名函数的标识由两部分组成：函数名及参数数量。这一特性决定了 Elixir 允许在模块中定义同名的函数，只要参数数量不等就可以。所以我们可以这样写：

```elixir
defmodule Calculator do
  def sum(a) do
    a + 1
  end

  def sum(a, b) do
    a + b
  end
end
```

在 Elixir 的眼中上面两个 `sum` 是完全不同的具名函数，然而在人类的眼中因为函数名称相同，于是它们之间似乎表达着某种关联性。这是 OK 的，符合自然语言的习惯，只不过我们需要注意别让两个同名函数做“风马牛不相及”的事情——这是人的责任而不是机器的。

### 函数体与作用域

和 JavaScript 相同，Elixir 最小的的作用域范围也是函数级别的。我们可以把 `do...end` 看作是作用域分隔的标示符，但要注意的是 `do...end` 不是底层的语法，它是一个语法糖，最终在编译的时候会被解释成 `do:`。也就是说，函数也可以写作：

```elixir
def sum(a, b), do: a + b  # 注意逗号！
```

我们可以用这种“内联”的形式来定义函数，只需要在脑海中明确这种语法糖的转换关系。我们甚至连模块定义都能内联（但为了代码的可读性起见，最好别这么做）：

```elixir
defmodule Calculator, do: (def sum(a, b), do: a + b)
```

此外，`do...end` 所构造的代码块并非只能用作具名函数定义，它也被用作其他的一些控制结构中，我们以后会看到更多的例子。

> `do...end` 有一个容易让初学者挠头的陷阱，要牢记：`do...end` 代码块总是绑定在最外层的函数调用上，所以必要的时候要使用 `()` 分隔表达式的优先层级。具体的例子可以看[官方的范例](http://elixir-lang.org/getting-started/case-cond-and-if.html#doend-blocks)（在最末尾处）。

除了最基本的函数作用域之外模块也会定义新的作用域，但是这个作用域不会“穿透”至其下的函数作用域内。也就是说，定义于模块作用域内的本地变量是无法在函数体内访问的。

```elixir
defmodule Calculator do
  factor = 1

  def sum(a) do
    a + factor
  end
end
```

上述代码是错误的，无法通过编译，Elixir 会认为 `sum/1` 里的 `factor` 是没有定义的函数，同时会警告 `Calculator` 里定义的 `factor` 是未使用的变量。于是我们可以想到定义一个 `factor/0` 来单纯返回一个值，这是可以的；不过模块可以定义一种特殊的“模块属性（Module Attribute）”来提供可访问的值：

```elixir
defmodule Calculator do
  @factor 1

  def sum(a) do
    a + @factor
  end
end
```

函数体内不能定义模块属性，因为本质上模块属性并非变量，从 OO 语言的角度来看倒是可以理解为模块级别的“常量”。模块属性可以重复定义（所以也不能等同于常量），函数在访问模块属性的时候总是向上寻找离自己最近的模块属性定义所对应的值。

### 私有函数

模块内部可以使用 `defp` 定义私有函数，私有函数只能在所定义的模块内部访问，其他的特性和一般的具名函数没有差别。

但是当你定义了多个同名函数（参数个数不同）时，它们要么都是私有的要么都不是，不能两种函数混合定义。

### 参数默认值

函数定义的参数可以有默认值，不过语法很古怪：

```elixir
def sum(a, b // 1) do
  a + b
end
```

我们已经知道 Elixir 不存在“赋值”一说，所以默认值不用 `=` 也就不难理解了。

如果一个函数有多个定义且需要给参数默认值，那就需要先写一个空的函数头声明（也就是没有实际函数体的声明），然后再写剩下的定义：

```elixir
defmodule Concat do
  def join(a, b \\ nil, sep \\ " ")

  def join(a, b, _sep) when is_nil(b) do
    a
  end

  def join(a, b, sep) do
    a <> sep <> b
  end
end
```

### 守护语句（Guard Clause）

守护语句的概念在 JavaScript（及其他高级语言）中也是有的，我们经常在函数中使用守护性的预先判断来提前返回或中断函数的执行：

```javascript
function rename(oldName, newName) {
  if (!oldName) return false // or throw errors if needed
  if (!newname) throw 'You need to provide a new name!'
  oldName = newName
}
```

概念是很简单的，不过在 Elixir 中的写法很不同了，Elixir 在语法层面直接提供了守护语句的写法：

```elixir
defmodule TellMe do
  def what_is(x) when is_atom(x) do
    IO.puts "#{x} is an atom"
  end

  def what_is(x) when is_number(x) do
    IO.puts "#{x} is a number"
  end

  def what_is(x) when is_list(x) do
    IO.puts "#{x} is a list"
  end
end
```

上面这段代码纯属演示性质，实际意义不大，不过我们可以看到守护语句填补了模式匹配的一些“空白”，这一点是很有意义的。

### 模式匹配＋守护语句

这两者的搭配是 Elixir 语言中很精彩的且具有代表性的应用技巧。模式匹配能满足结构和值相等性的比较并能为变量进行必要的值绑定，但是如果你要针对值做一些额外的比较——比如说类型检测或是转换测试之类的，就需要守护语句来协助了。

一个很经典的例子是递归阶乘在 Elixir 里的实现：

```elixir
defmodule Factorial do
  def of(0), do: 1
  def of(n), do: n * of(n - 1)
end
```

简单、优雅，但是不完善——如果我们传入的 `n` 是负数会怎样呢？（无限循环，因为第一个 `of/1` 永远也匹配不到）

如果我们不关心处理负数的情况，那就可以简单的加上守护语句：

```elixir
defmodule Factorial do
  def of(0), do: 1
  def of(n) when n > 0, do: n * of(n - 1)
end
```

现在如果你传入负数的话，Elixir 会抛出错误告诉你没有能够处理此类参数的 `Factorial.of/1`，这正好符合我们的需要。

在默认环境下，守护语句并不是万能的，[Elixir 只提供了一些缺省的表达式供其使用](http://elixir-lang.org/getting-started/case-cond-and-if.html#expressions-in-guard-clauses)。不过守护语句可用的表达式是可以扩展的，这算是一个高级话题，我们留待后谈。

我认为前面所介绍的内容已经涵盖了 Elixir 函数的基本特性，有了这些知识就足以进一步探索 Elixir 的世界了。但是和函数相关的还有一个非常非常重要的特性管道操作符还没有涉及，管道操作符是 Elixir 令人着迷的特色之一，但是谈它之前有必要先讲枚举（Enumerable）和流（Stream），函数在其中扮演的是“穿针引线”的角色。因此，本篇告一段落，下一篇我们先从列表和递归讲起，开始真正触及函数式编程的核心之处。
