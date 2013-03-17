
This lesson covers:

* Basic Data Structures
  * "Lists"
  * "Sets"
  * "Tuple"
  * "Maps"

* Functional Combinators

  * "map"
  * "foreach"
  * "filter"
  * "zip"
  * "partition"
  * "find"
  * "drop and dropWhile"
  * "foldRight and foldLeft"
  * "flatten"
  * "flatMap"
  * "Generalized functional combinators"
  * "Map?"


这个章节的内容包含

* 基本数据结构

  * <a href="#lists">List</a>
  * <a href="#sets">Set</a>
  * <a href="#tuple">Tuple</a>
  * <a href="#Maps">Maps</a>

* 函数组合器

  * <a href="#map">map</a>
  * <a href="#foreach">foreach</a>
  * <a href="#filter">filter</a>
  * <a href="#zip">zip</a>
  * <a href="#partition">partition</a>
  * <a href="#find">find</a>
  * <a href="#drop">drop and dropWhile</a>
  *  <a href="fold">foldRight and foldLeft</a>
  * <a href="#flatten">flatten</a>
  * <a href="#flatMap">flatMap</a>
  * <a href="#generalized">Generalized functional conbinators</a>
  * <a href="#vsMap">为什么用Map</a>

# Basic Data Structures

Scala provides some nice collections.

*See Also* Effective Scala has opinions about how to use <a href="http://twitter.github.com/effectivescala/#Collections">collections</a>.

# 基本数据结构
Scala提供了一些很方便的集合类。
*参考* <a href="http://twitter.github.com/effectivescala/#Collections">《Effective Scala》中关于怎么使用集合类的内容。

## Lists

## <a name="lists"> List</a>

<pre class="brush: java; gutter: true">
scala> val numbers = List(1, 2, 3, 4)
numbers: List[Int] = List(1, 2, 3, 4)
</pre>

## Sets

## <a name="sets">Set</a>

Sets have no duplicates
集合中没有重复元素

<pre class="brush: java; gutter: true">
scala> Set(1, 1, 2)
res0: scala.collection.immutable.Set[Int] = Set(1, 2)
</pre>

## Tuple

## <a name="tuple">元组(Tuple)</a>

A tuple groups together simple logical collections of items without using a class.
元组可以直接把一些具有简单逻辑关系的一组数据组合在一起，并且不需要额外的类。

<pre class="brush: java; gutter: true">
scala> val hostPort = ("localhost", 80)
hostPort: (String, Int) = (localhost, 80)
</pre>

Unlike case classes, they don't have named accessors, instead they have accessors that are named by their position and is 1-based rather than 0-based.

和case class不同，元组的元素不能通过名称进行访问，不过它们可以通过基于它们位置的名称进行访问，这个位置是从1开始而非从0开始。

<pre class="brush: java; gutter: true">
scala> hostPort._1
res0: String = localhost

scala> hostPort._2
res1: Int = 80
</pre>

Tuples fit with pattern matching nicely.

元组可以很好地和模式匹配配合使用。

<pre class="brush: java; gutter: true">
hostPort match {
  case ("localhost", port) => ...
  case (host, port) => ...
}
</pre>

Tuple has some special sauce for simply making Tuples of 2 values: <code>-></code>

创建一个包含2个值的元组有一个很简单的方式：`->`


<pre class="brush: java; gutter: true">
scala> 1 -> 2
res0: (Int, Int) = (1,2)
</pre>

*See Also* Effective Scala has opinions about <a href="http://twitter.github.com/effectivescala/#Functional programming-Destructuring bindings">destructuring bindings</a> ("unpacking" a tuple).

*参考* 《Effective Scala》中关于<a href="http://twitter.github.com/effectivescala/#Functional programming-Destructuring bindings">解除绑定</a>（拆封一个元组）的观点。

## Maps

It can hold basic datatypes.

## <a name="maps">Map</a>

Map里可以存放基本的数据类型。

<pre class="brush: java; gutter: true">
Map(1 -> 2)
Map("foo" -> "bar")
</pre>

This looks like special syntax but remember back to our discussion of Tuple that <code>-></code> can be use to create Tuples.
这个看起来是一个特殊的语法，不过回想一下前面我们讨论元组的时候，'->'符号是可以用来创建元组的。

Map() also uses that variable argument syntax we learned back in Lesson #1: <code>Map(1 -> "one", 2 -> "two")</code> which expands into <code>Map((1, "one"), (2, "two"))</code> with the first element being the key and the second being the value of the Map.

Map()可以使用我们在第一节里讲到的可变参数的语法：`Map( 1 -> "one", 2 -> "two")`，它会被扩展为`Map((1,"one"),(2,"two"))`，其中第一个元素参数是key，第二个元素是value。

Maps can themselves contain Maps or even functions as values.

Map里也可以包含Map，甚至也可以把函数当作值存在Map里。

<pre class="brush: java; gutter: true">
Map(1 -> Map("foo" -> "bar"))
</pre>

<pre class="brush: java; gutter: true">
Map("timesTwo" -> { timesTwo(_) })
</pre>

## Option

## <a name="option"> Option


<code>Option</code> is a container that may or may not hold something.

The basic interface for Option looks like:

`Option`是一个包含或者不包含某些事物的容器。

Option的基本接口类似于：

<code>
trait Option[T] {
  def isDefined: Boolean
  def get: T
  def getOrElse(t: T): T
}
</code>

Option itself is generic and has two subclasses: <code>Some[T]</code> or <code>None</code>

Let's look at an example of how Option is used:

<code>Map.get</code> uses <code>Option</code> for its return type. Option tells you that the method might not return what you're asking for.

Option本身是泛型的，它有两个子类：`Some[T]`和`None`

我们来看一个Option的示例：
`Map.get`使用`Option`来作为它的返回类型。Option的作用是告诉你这个方法可能不会返回你请求的值。

<pre class="brush: java; gutter: true">
scala> val numbers = Map(1 -> "one", 2 -> "two")
numbers: scala.collection.immutable.Map[Int,String] = Map((1,one), (2,two))

scala> numbers.get(2)
res0: Option[java.lang.String] = Some(two)

scala> numbers.get(3)
res1: Option[java.lang.String] = None
</pre>

Now our data appears trapped in this <code>Option</code>. How do we work with it?

A first instinct might be to do something conditionally based on the <code>isDefined</code> method.

现在，我们要的数据存在于这个`Option`里。那么我们该怎么处理它呢？

一个比较直观的方法就是根据`isDefined`方法的返回结果作出不同的处理。

<code>
// We want to multiply the number by two, otherwise return 0.
//如果这个值存在的话，那么我们把它乘以2，否则返回0。
val result = if (res1.isDefined) {
  res1.get * 2
} else {
  0
}
</code>

We would suggest that you use either <code>getOrElse</code> or pattern matching to work with this result.

<code>getOrElse</code> lets you easily define a default value.

不过，我们更加建议你使用`getOrElse`或者模式匹配来处理这个结构。

<code>
val result = res1.getOrElse(0) * 2
</code>

Pattern matching fits naturally with <code>Option</code>.

模式匹配可以很好地和`Option`进行配合使用。

<code>
val result = res1 match {
  case Some(n) => n * 2
  case None => 0
}
</code>

*See Also* Effective Scala has opinions about <a href="http://twitter.github.com/effectivescala/#Functional programming-Options">Options</a>.
*参考* 《Effective Scala》中关于 <a href="http://twitter.github.com/effectivescala/#Functional programming-Options">Options</a>的内容。


# Functional Combinators
# <a name="combinators">函数组合器</a>

<code>List(1, 2, 3) map squared</code> applies the function <code>squared</code> to the elements of the the list, returning a new list, perhaps <code>List(1, 4, 9)</code>. We call operations like <code>map</code> <em>combinators</em>. (If you'd like a better definition, you might like <a href="http://stackoverflow.com/questions/7533837/explanation-of-combinators-for-the-working-man">Explanation of combinators</a> on Stackoverflow.) Their most common use is on the standard data structures.

`List(1,2,3) map squared`会在列表的每个元素上分别应用`squared`函数，并且返回一个新的列表，可能是`List(1,4,9)`。我们把类似于`map`这样的操作称为<em>组合器<em>。（如果你需要一个更好的定义，你或许会喜欢Stackoverflow上的<a href="http://stackoverflow.com/questions/7533837/explanation-of-combinators-for-the-working-man">关于组合器的解释</a>。

## map

Evaluates a function over each element in the list, returning a list with the same number of elements.
在列表中的每个元素上计算一个函数，并且返回一个包含相同数目元素的列表。

<pre class="brush: java; gutter: true">
scala> numbers.map((i: Int) => i * 2)
res0: List[Int] = List(2, 4, 6, 8)
</pre>

or pass in a partially evaluated function
或者传入一个部分计算的函数

<pre class="brush: java; gutter: true">

scala> def timesTwo(i: Int): Int = i * 2
timesTwo: (i: Int)Int

scala> numbers.map(timesTwo _)
res0: List[Int] = List(2, 4, 6, 8)
</pre>

h2(#foreach). foreach

foreach is like map but returns nothing. foreach is intended for side-effects only.

## <a name ="foreach">foreach</a>

foreach和map相似，只不过它没有返回值，foreach只要是为了对参数进行作用。

<pre class="brush: java; gutter: true">
scala> numbers.foreach((i: Int) => i * 2)
</pre>

returns nothing.
没有返回值。

You can try to store the return in a value but it'll be of type Unit (i.e. void)

你可以尝试把返回值放在一个变量里，不过它的类型应该是Unit（或者是void）

<pre class="brush: java; gutter: true">
scala> val doubled = numbers.foreach((i: Int) => i * 2)
doubled: Unit = ()
</pre>

## filter

removes any elements where the function you pass in evaluates to false.  Functions that return a Boolean are often called predicate functions.

## <a name="filter">filter</a>

移除任何使得传入的参数返回false的元素。返回Boolean类型的函数一般都称为断言函数。

<pre class="brush: java; gutter: true">
scala> numbers.filter((i: Int) => i % 2 == 0)
res0: List[Int] = List(2, 4)
</pre>

<pre class="brush: java; gutter: true">
scala> def isEven(i: Int): Boolean = i % 2 == 0
isEven: (i: Int)Boolean

scala> numbers.filter(isEven _)
res2: List[Int] = List(2, 4)
</pre>

## zip

## <a name="zip">zip</a>

zip aggregates the contents of two lists into a single list of pairs.

zip把两个列表的元素合成一个由元素对组成的列表里。

<pre class="brush: java; gutter: true">
scala> List(1, 2, 3).zip(List("a", "b", "c"))
res0: List[(Int, String)] = List((1,a), (2,b), (3,c))
</pre>

## partition

## <a name="partition">partition</a>

<code>partition</code> splits a list based on where it falls with respect to a predicate function.
`partition`根据断言函数的返回值对列表进行拆分。
<pre class="brush: java; gutter: true">
scala> val numbers = List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
scala> numbers.partition(_ %2 == 0)
res0: (List[Int], List[Int]) = (List(2, 4, 6, 8, 10),List(1, 3, 5, 7, 9))
</pre>

h2(#find). find

find returns the first element of a collection that matches a predicate function.

## <a name="find">find</a>
返回集合里第一个匹配断言函数的元素

<pre class="brush: java; gutter: true">
scala> numbers.find((i: Int) => i > 5)
res0: Option[Int] = Some(6)
</pre>

h2(#drop). drop & dropWhile

## <a name ="drop">drop & dropWhile</a>

<code>drop</code> drops the first i elements

`drop`丢弃前i个元素

<pre class="brush: java; gutter: true">
scala> numbers.drop(5)
res0: List[Int] = List(6, 7, 8, 9, 10)
</pre>

<code>dropWhile</code> removes the first elements that match a predicate function. For example, if we <code>dropWhile</code> odd numbers from our list of numbers, <code>1</code> gets dropped (but not <code>3</code> which is "shielded" by <code>2</code>).

`dropWhile`移除前几个匹配断言函数的元素。例如，如果我们从numbers列表里`dropWhile`奇数的话，`1`会被移除（`3`则不会，因为它被`2`所“保护”）。


<pre class="brush: java; gutter: true">
scala> numbers.dropWhile(_ % 2 != 0)
res0: List[Int] = List(2, 3, 4, 5, 6, 7, 8, 9, 10)
</pre>

## foldLeft

## <a name="fold">foldLeft</a>

<pre class="brush: java; gutter: true">
scala> numbers.foldLeft(0)((m: Int, n: Int) => m + n)
res0: Int = 55
</pre>

0 is the starting value (Remember that numbers is a List[Int]), and m
acts as an accumulator.

0是起始值（注意numbers是一个List[Int])，m是累加值。

Seen visually:
更加直观的来看：

<pre class="brush: java; gutter: true">
scala> numbers.foldLeft(0) { (m: Int, n: Int) => println("m: " + m + " n: " + n); m + n }
m: 0 n: 1
m: 1 n: 2
m: 3 n: 3
m: 6 n: 4
m: 10 n: 5
m: 15 n: 6
m: 21 n: 7
m: 28 n: 8
m: 36 n: 9
m: 45 n: 10
res0: Int = 55
</pre>

### foldRight
### foldRight

Is the same as foldLeft except it runs in the opposite direction.

这个和foldLeft相似，只不过是方向相反。

<pre class="brush: java; gutter: true">
scala> numbers.foldRight(0) { (m: Int, n: Int) => println("m: " + m + " n: " + n); m + n }
m: 10 n: 0
m: 9 n: 10
m: 8 n: 19
m: 7 n: 27
m: 6 n: 34
m: 5 n: 40
m: 4 n: 45
m: 3 n: 49
m: 2 n: 52
m: 1 n: 54
res0: Int = 55
</pre>

## flatten

flatten collapses one level of nested structure.

## <a name="flatten">flatten</a>
flatten可以把嵌套的结构展开。

<pre class="brush: java; gutter: true">
scala> List(List(1, 2), List(3, 4)).flatten
res0: List[Int] = List(1, 2, 3, 4)
</pre>

h2(#flatMap). flatMap

flatMap is a frequently used combinator that combines mapping and flattening. flatMap takes a function that works on the nested lists and then concatenates the results back together.

## <a name="flatMap">flaoMap</a>

flatMap是一个常用的combinator，它结合了map和flatten的功能。flatMap接收一个可以处理嵌套列表的函数，然后把返回结果连接起来。

<pre class="brush: java; gutter: true">
scala> val nestedNumbers = List(List(1, 2), List(3, 4))
nestedNumbers: List[List[Int]] = List(List(1, 2), List(3, 4))

scala> nestedNumbers.flatMap(x => x.map(_ * 2))
res0: List[Int] = List(2, 4, 6, 8)
</pre>

Think of it as short-hand for mapping and then flattening:
可以把它当作map和flatten两者的缩写：

<pre class="brush: java; gutter: true">
scala> nestedNumbers.map((x: List[Int]) => x.map(_ * 2)).flatten
res1: List[Int] = List(2, 4, 6, 8)
</pre>

that example calling map and then flatten is an example of the "combinator"-like nature of these functions.

这个调用map和flatten的示例是这些函数的类“组合器”特点的展示。

*See Also* Effective Scala has opinions about <a href="http://twitter.github.com/effectivescala/#Functional programming-`flatMap`">flatMap</a>.
*参考* 《Effective Scala》中关于<a href="http://twitter.github.com/effectivescala/#Functional programming-`flatMap`">flatMap</a>的内容.



## Generalized functional combinators

Now we've learned a grab-bag of functions for working with collections.

What we'd like is to be able to write our own functional combinators.

Interestingly, every functional combinator shown above can be written on top of fold.  Let's see some examples.

## <a name="generalized"> Generalized functional combinators

现在，我们学习了一大堆处理集合的函数。
不过，我们更加感兴趣的是怎么写我们自己的functional combinator。
有趣的是，上面展示的每个functional combinator都是可以通过fold来实现的。我们来看一些示例。
<pre class="brush: java; gutter: true">
def ourMap(numbers: List[Int], fn: Int => Int): List[Int] = {
  numbers.foldRight(List[Int]()) { (x: Int, xs: List[Int]) =>
    fn(x) :: xs
  }
}

scala> ourMap(numbers, timesTwo(_))
res0: List[Int] = List(2, 4, 6, 8, 10, 12, 14, 16, 18, 20)
</pre>

Why <tt>List[Int]()</tt>? Scala wasn't smart enough to realize that you wanted an empty list of Ints to accumulate into.

为什么要<tt>List[Int]</tt>?因为Scala还不能聪明到知道你需要在一个空的Int列表上来进行累加。

## Map?

All of the functional combinators shown work on Maps, too.  Maps can be thought of as a list of pairs so the functions you write work on a pair of the keys and values in the Map.
我们上面所展示的所有函数组合器都能都Map进行处理。Map可以当作是由键值对组成的列表，这样你写的函数就可以对Map里的key和value进行处理。

## <a name= "vsMap">Map?</a>

<pre class="brush: java; gutter: true">
scala> val extensions = Map("steve" -> 100, "bob" -> 101, "joe" -> 201)
extensions: scala.collection.immutable.Map[String,Int] = Map((steve,100), (bob,101), (joe,201))
</pre>

Now filter out every entry whose phone extension is lower than 200.
现在过滤出所有分机号码小于200的元素。

<pre class="brush: java; gutter: true">
scala> extensions.filter((namePhone: (String, Int)) => namePhone._2 < 200)
res0: scala.collection.immutable.Map[String,Int] = Map((steve,100), (bob,101))
</pre>

Because it gives you a tuple, you have to pull out the keys and values with their positional accessors. Yuck!

Lucky us, we can actually use a pattern match to extract the key and value nicely.

因为你拿到的是一个元组，所以你不得不通过它们的位置来取得对应的key和value，太恶心了！
幸运的是，我们实际上可以用一个模式匹配来优雅地获取key和value。

<pre class="brush: java; gutter: true">
scala> extensions.filter({case (name, extension) => extension < 200})
res0: scala.collection.immutable.Map[String,Int] = Map((steve,100), (bob,101))
</pre>

Why does this work? Why can you pass in a partial pattern match?

为什么这样也可以呢？为什么你可以传入一个局部的模式匹配呢？

Stay tuned for next week!
这个内容就留在下周吧！