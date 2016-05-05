+++
categories = ["博文"]
date = "2016-05-05T00:00:00+08:00"
description = ""
isCJKLanguage = true
tags = ["elixir"]
title = "JavaScript 程序员眼中的 Elixir：集合"
+++

```elixir
iex(1)> [1, :foo, 3.14, "bar", {1, 2, 3}]
[1, :foo, 3.14, "bar", {1, 2, 3}]

iex(2)> {1, :foo, 3.14, "bar", [1, 2, 3]}
{1, :foo, 3.14, "bar", [1, 2, 3]}
```

请问看过上述代码之后是何感想？我第一次看呢感觉就是：晕～

那换个问题，上面两种集合哪个接近 JavaScript 里的数组（Array）？从符号上看应该是第一种：列表（List），可实际上最接近的是第二种：元组（Tuple），但是在 Elixir 中我们却不太会用在 JavaScript 中使用数组那样去使用元组。

## 元组（Tuple）

元组是有序的数值集合，并且它是在内存中连续存储这些数值的，这就意味着通过下标（也是从 0 开始）访问数值以及获得元组的长度这样的操作是很快的。

典型的 Elixir 程序里一般不太会用元组来表示大量数据集合，有三五个就了不起啦～当需要表示更多的数据的时候会倾向于选择其它的集合类型——JavaScript 里那是没得挑，要么就自己去实现好了。原因？在我看来可能是因为元组并没有实现 `Enumerable` 协议的缘故，这个不急我们后面会讲。

Elixir 用于操作元组的函数并不多，翻翻 API 文档也就十分钟全部看完，不过有几个方法是在 `Kernel` 模块里的，要注意别漏了。它们分别是：

`Kernel.elem/2`，`Kernel.is_tuple/1`，`Kernel.put_elem/3`，以及 `Kernel.tuple_size/1`。

Elixir 中有一个非常重要的“习俗”：让函数返回元组，且当没有错误的时候，元组的第一个值是原子 `:ok`，这个习惯特别像 Node.js 的 error first callback，但在 Elixir 配合模式匹配则更加普遍和强大。下面就是一个常见的例子：

```elixir
iex(1)> File.open("config.toml")
{:ok, #PID<0.60.0>}
iex(2)> File.open("config.yaml")
{:error, :enoent}
```

关于这个以后还要深入地说，现在还是把焦点集中在集合上面吧。

## 列表（List）

由于写法完全一致所以列表太容易被看作是 JavaScript 数组了，可是相比元组来说列表与数组的差异就更大了。

首先列表是链表集合而不是线性表集合，列表要么就是空的，要么就是由头（head）和尾（tail）组成：头保存一个值，尾本身又是一个列表。这是一种递归式的结构定义，而它正是大多数 Elixir 程序的核心。

> JavaScript 的数组是线性表集合吗？JavaScript 是动态语言，这意味着数组的存储方式也是可以动态规划的。事实上，不同的引擎对此会有不同的处理方式，因此很难一概而论。大体上我们认为 JavaScript 数组是线性的，因而也会有很多文章教程来探讨在 JavaScript 中创建自己的链表数据结构。

这种结构使得列表利于遍历但不利于随机读取（比如获取列表中第 _n_ 个元素总是要先扫描前面的 _n-1_ 个元素）。不过获取列表的头然后提取列表的尾则总是高效的。

列表有一种特性最值得牢记：对于不可变数据结构来说，想要修改它就意味着要先复制一份新的；然而由于列表是头尾结合的递归结构，要想去掉列表的头就不需要复制列表本身，只需要返回列表尾的指针就可以了。这种特性被广泛应用在列表的遍历与递归操作中。

列表支持下面的操作符（实际上都是 `Kernel` 模块下的函数／宏）：

```elixir
iex(1)> [1, 2, 3] ++ [4, 5, 6]
[1, 2, 3, 4, 5, 6]
iex(2)> [1, 1, 2, 3, 3, 4] -- [2, 3, 4]
[1, 1, 3]
iex(3)> 1 in [1, 2, 3]
true
iex(4)> 0 in [1, 2, 3]
false
```

列表模块里的函数也比元组要多得多，不过在实践中建议优先选择 `Enum` 模块下的函数来操作列表。另外 `Kernel.hd/1` 与 `Kernel.tl/1` 分别用来取列表的头和尾。

下面是一个有趣的特性叫做字符列表（Char List），目前仅作了解即可。

```elixir
iex(1)> [104, 101, 108, 108, 111]
'hello'
iex(2)> 'hello' == "hello"
false
```

## 关键字列表（Keyword List）

虽然名字里还有列表二字，但关键字列表却像 JavaScript 对象字面量那样是键值对形式的数据集合。既然如此为啥还非要算作列表呢？看：

```elixir
iex(1)> [name: "nightire", age: 35] = [{:name, "nightire"}, {:age, 35}]
[name: "nightire", age: 35]
iex(2)> [name: "nightire", age: 35] == [{:name, "nightire"}, {:age, 35}]
true
```

原来关键字列表就是一个二元元组的列表，每一个元组的第一个元素必须是原子（Atom），第二个元素可以是其它任何值，以此来代表一对关联数据结构。

关键字列表的重要性源自于它自身的三大特点：

- 键必须是原子
- 键是有序的，遵照定义时的顺序
- 键可以给定多次

下面是一些例子：

```elixir
iex(1)> person = [name: "nightire", age: 35]
[name: "nightire", age: 35]
iex(2)> person[:name]
"nightire"
iex(3)> person[:age]
35

iex(4)> nightire = person ++ [likes: ["music", "programming"]]
[name: "nightire", age: 35, likes: ["music", "programming"]]
iex(5)> nightire[:likes]
["music", "programming"]

iex(6)> younger = [age: 18] ++ nightire
[age: 18, name: "nightire", age: 35, likes: ["music", "programming"]]
iex(7)> younger[:age]
18
```

关键字列表特别适合用作函数传参或者是用作构造 DSL，比如说：

```elixir
query = from p in People,
   where: p.age > 18,
   where: p.weight < 100
  select: p
```

以及：

```elixir
DB.save record, name: "nightire", admin: true
```

这个例子同时也告诉我们 Elixir 允许在关键字列表作为函数调用的最后一个参数时省略 `[]`。省略环绕标记这件事情有时候会让人晕一下下，特别是要小心下面这种情形：

```elixir
iex(1)> [1, 2, foo: "bar"]
[1, 2, {:foo, "bar"}]
iex(2)> {1, 2, foo: "bar"}
{1, 2, [foo: "bar"]}
```

除了作为函数参数列表的末尾项之外，作为元组及列表的末尾项也能省略环绕符号，这样的语法特性能省却很多字符但同时也会增加头脑中抽象解释和转换的负担。

为了操作关键字列表 Elixir 专门提供了 `Keyword` 模块，底下的函数也不少。以前关键字列表是实现了 `Dict` 模块的，后来 `Dict` 被废弃了，以后还是专用 `Keyword` 好。

## 映射（Map）

> 我看到一些中文资料把作为数据结构的 `Map` 称作图，这是不对的。数据结构中的图应该是 `Graph`，而 `Map` 虽然有地图的意思但却和数据结构所说的“图”大不相同。`Map` 同时也有映射的意思，其实指的就是一一对应的键／值对，在这里的含义非常准确。

JavaScript 中的对象兼具关键字列表和映射的特性，实际上关键字列表所具备的特性使得它并不适合作为纯数据载体，而映射则特别适合。我猜想这也是为什么 ES2015 会添加 `Map` 来替代一部分原属于 `Object` 的职责。

什么意思呢？首先，关键字列表的键只能是原子，相对的 JavaScript 对象的键名只能是字符串（其实还有符号类型，但是属于新数据类型，流传度不高），这就不够灵活了对吧。映射则没有这种限制，任何值都可以做映射中的键和值——包括复合类型；而且还能是表达式。另外如果映射中的键名全部是原子则可以使用关键字列表的语法来定义以提高便利性。

```elixir
iex(1)> %{0 => 0, "one" => 1, :two => 2, [1, 2, 3] => 3}
%{0 => 0, :two => 2, [1, 2, 3] => 3, "one" => 1}

iex(2)> %{one: 1, two: 2}
%{one: 1, two: 2}
iex(3)> %{one: 1, two: 2} == %{:one => 1, :two => 2}
true
```

其次，映射里的数据是不关心顺序的，而且键也不能重复，如果有重复的键添加进来则会覆盖之前的同名键值对。

```elixir
iex(1)> %{one: 1, one: "1"}
%{one: "1"}
```

这一特性使得映射在模式匹配时比关键字列表有用（特别是键名唯一性），看看这些例子：

```elixir
iex(1)> %{} = %{:foo => "foo", "bar" => :bar}
%{:foo => "foo", "bar" => :bar}

iex(2)> %{:foo => str} = %{:foo => "foo", "bar" => :bar}
%{:foo => "foo", "bar" => :bar}
iex(3)> str
"foo"

iex(4)> %{:baz => any} = %{:foo => "foo", "bar" => :bar}
** (MatchError) no match of right hand side value: %{:foo => "foo", "bar" => :bar}
```

做这种事情的时候是没法依靠关键字列表的：

```elixir
iex(1)> [foo: str] = [foo: "foo", bar: "bar"]
** (MatchError) no match of right hand side value: [foo: "foo", bar: "bar"]
```

在 Elixir 中，会经常这样使用映射：

```elixir
response_types = %{
  {:error, :enoent} => :fatal,
  {:error, :busy} => :retry
}
# %{{:error, :busy} => :retry, {:error, :enoent} => :fatal}

response_types[{:error, :busy}]
# :retry
```

而在 JavaScript 中如果只考虑对象的话这是不可想象的，因为你不能用另一个对象去做键。这也是为什么 ES2015 要添加 `Map` 类型的缘故，因为它可以做到：

```javascript
const responseTypes = new Map()
responseTypes.set(["error", "enoent"], "fatal")
responseTypes.set(["error", "busy"], "retry")

for (let [key, value] of responseTypes) {
  console.log(key, value)
}
// => ["error", "enoent"] "fatal"
// => ["error", "busy"] "retry"
```

因为 JavaScript 没有元组（尽管有类似原子的符号（Symbol）），所以这个例子不会像 Elixir 那么便利（除非先把作为键名的数组引用保存下来），不过同样的数据结构还是做得到的。

`Map` 模块提供了和 `Keyword` 模块非常类似的函数，以前它们都是实现自 `Dict` 的，但是 again，`Dict` 被废弃了哦！

映射提供了独有的语法来访问和更新键名为原子的值，这一点很有用处：

```elixir
iex(1)> map = %{:a => 1, 2 => :b}
%{2 => :b, :a => 1}
iex(2)> map.a
1
iex(3)> map.c
** (KeyError) key :c not found in: %{2 => :b, :a => 1}

iex(3)> %{map | :a => "1"}
%{2 => :b, :a => "1"}
iex(4)> %{map | :c => 3}
** (KeyError) key :c not found in: %{2 => :b, :a => 1}
    (stdlib) :maps.update(:c, 3, %{2 => :b, :a => 1})
    (stdlib) erl_eval.erl:255: anonymous fn/2 in :erl_eval.expr/5
    (stdlib) lists.erl:1262: :lists.foldl/3
```

乍一看这似乎只是一个语法特性，但在实践中这种语法会带来风格上的变化，[这篇文章](http://blog.plataformatec.com.br/2014/09/writing-assertive-code-with-elixir/)描述了 Elixir 程序员是如何使用这种语法来实现更加简洁的“断定式”编码风格的。以后我也会单独撰文来说说这种 _assertive code_ 风格。

综合以上可以发现 JavaScript 的 `Object` 是综合了关键字列表与映射的特性的，相对而言更加接近关键字列表；过去由于却少 `Map` 类型所以也把它当作映射来用，但实际上映射和关键字列表有本质上的差异，适用的场景也非常不同。在学习了 Elixir 之后，反而对 JavaScript 的 `Object` 和 `Map` 也有了非常明确的理解，真可谓相得益彰啊。

那么下一篇我要讲函数了，等不及要写点真正有用的东西了😸 ！
