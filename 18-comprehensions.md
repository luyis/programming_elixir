18-列表速构（Comprehension）
========
[生成器和过滤器](#182-%E7%94%9F%E6%88%90%E5%99%A8%E5%92%8C%E8%BF%87%E6%BB%A4%E5%99%A8)<br/>
[比特串生成器](#182-bitstring%E7%94%9F%E6%88%90%E5%99%A8)<br/>
[Into](#183-into)<br/>

>Comprehensions翻译成“速构”不知道贴不贴切，这参照了《Erlang/OTP in Action》译者的用辞。
“速构”是函数式语言中常见的概念，它大体上指的是用一套规则（比如从另一个列表，过滤掉一些元素）来生成元素填充新列表。  
这个概念我们在中学的数学课上就可能已经接触过，在大学高数中更为常见：如```{ x | x ∈ N }```，指的是这个集合里所有元素是自然数。
该集合是由自然数集合的元素映射生成过来的。相关知识可见[WIKI](http://en.wikipedia.org/wiki/List_comprehension)。

Elixir中，使用枚举类型（如列表）来做循环操作是很常见的。对对象列表进行枚举时，通常要有选择性地过滤掉其中一些元素，还有可能做一些变换。列表速构（comprehensions）就是为此目的诞生的语法糖：把这些常见任务分组，放到特殊的```for```中执行。

例如，我们可以这样计算列表中每个元素的平方：
```
iex> for n <- [1, 2, 3, 4], do: n * n
[1, 4, 9, 16]
```
注意看，```<-```符号就是模拟自```∈ ```的形象。
这个例子用熟悉（当然，如果你高数课没怎么听那就另当别论）的数学符号表示就是：
```
S = { X^2 | X ∈ [1,4], X ∈ N}
```
这个例子用常见的编程语言去理解，基本上类似于foreach...in...什么的。但是更强大。

一个列表速构由三部分组成：生成器，过滤器和收集器。

##18.2-生成器和过滤器
在上面的例子中，```n <- [1, 2, 3, 4]```就是生成器。它字面意思上生成了即将要在速构中使用的数值。
任何枚举类型都可以传递给生成器表达式的右端：
```
iex> for n <- 1..4, do: n * n
[1, 4, 9, 16]
```
这个例子中的生成器是一个__范围__。

生成器表达式支持模式匹配，它会忽略所有不匹配的模式。
想象一下如果不用范围而是用一个键值列表，键只有```:good```和```:bad```两种，来计算中间被标记成‘good’的元素的平方：
```
iex> values = [good: 1, good: 2, bad: 3, good: 4]
iex> for {:good, n} <- values, do: n * n
[1, 4, 16]
```

过滤器能过滤掉某些产生的值。例如我们可以只对奇数进行平方运算：
```
iex> require Integer
iex> for n <- 1..4, Integer.odd?(n), do: n * n
[1, 9]
```
过滤器会保留所有判断结果是非nil或非false的值。

总的来说，速构比直接使用枚举或流模块的函数提供了更精确的表述。
不但如此，速构还接受多个生成器和过滤器。下面就是一个例子，代码接受目录列表，删除这些目录下的所有文件：
```
for dir  <- dirs,
  file <- File.ls!(dir),
  path = Path.join(dir, file),
  File.regular?(path) do
    File.rm!(path)
end
```

需要记住的是，在速构中，变量赋值这种事应在生成器中进行。因为在过滤器或代码块中的赋值操作不会反映到速构外面去。

## 18.2-比特串生成器
速构也支持比特串作为生成器，而且这种生成器在组织处理比特串的流时非常有用。
下面的例子中，程序从二进制数据（表示为<<像素1的R值，像素1的G值，像素1的B值，像素2的R值，像素2的G...>>）中接收一个像素的列表，把它们转换为元组：
```
iex> pixels = <<213, 45, 132, 64, 76, 32, 76, 0, 0, 234, 32, 15>>
iex> for <<r::8, g::8, b::8 <- pixels>>, do: {r, g, b}
[{213,45,132},{64,76,32},{76,0,0},{234,32,15}]
```
比特串生成器可以和“普通的”枚举类型生成器混合使用，过滤器也是。

## 18.3-Into
在上面的例子中，速构返回一个列表作为结果。
但是，通过使用```:into```选项，速构的结果可以插入到一个不同的数据结构中。
例如，你可以使用比特串生成器加上```:into```来轻松地构成无空格字符串：
```
iex> for <<c <- " hello world ">>, c != ?\s, into: "", do: <<c>>
"helloworld"
```

集合、图以及其他字典类型都可以传递给```:into```选项。总的来说，```:into```接受任何实现了_Collectable_协议的数据结构。

例如，IO模块提供了流。流既是Enumerable也是Collectable。
你可以使用速构实现一个回声终端，让其返回任何输入的东西的大写形式：
```
iex> stream = IO.stream(:stdio, :line)
iex> for line <- stream, into: stream do
...>   String.upcase(line) <> "\n"
...> end
```

现在在终端中输入任意字符串，你会看到同样的内容以大写形式被打印出来。
不幸的是，这个例子会让你的shell陷入到该速构代码中，只能用Ctrl+C两次来退出。
