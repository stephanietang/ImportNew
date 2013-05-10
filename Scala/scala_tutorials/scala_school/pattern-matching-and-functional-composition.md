英文原文: [Scala School](http://twitter.github.com/scala_school/pattern-matching-and-functional-composition.html)，翻译：[ImportNew](http://www.importnew.com) - [朱伟杰](http://www.importnew.com/author/zhuweijie)

# Pattern matching & functional composition

# 模式匹配和函数复合

This lesson covers:

* Function Composition

	* compose
	* andThen

* Currying vs Partial Application
* PartialFunctions

	* range and domain
	* composition with orElse

* What is a case statement?

这节的内容包含：

* <a href="#composition">函数复合</a>
	* compose
	* andThen
* <a href="#curryvspartial">Currying 与 Partial Application</a>
* <a href="#PartialFunction">PartialFunctions</a>
	* range and domain
	* composition with orElse
* <a href="#case">case语句究竟是什么？</a>


## Function Composition

Let's make two helpful functions:

## <a name="composition">函数复合</a>
我们首先定义两个有用的函数：

<pre class="brush: java; gutter: true">
scala> def addUmm(x: String) = x + " umm"
addUmm: (x: String)String

scala>  def addAhem(x: String) = x + " ahem"
addAhem: (x: String)String
</pre>

### compose

<code>compose</code> makes a new function that composes other functions <code>f(g(x))</code>

### compose
`compose`可以通过组合其他的函数来创建一个新的函数`f(g(x))`

<pre class="brush: java; gutter: true">
scala> val ummThenAhem = addAhem _ compose addUmm _
ummThenAhem: (String) => String = <function1>

scala> ummThenAhem("well")
res0: String = well umm ahem
</pre>

### andThen

<code>andThen</code> is like <code>compose</code>, but calls the first function and then the second, <code>g(f(x))</code>

### andThen

`andThen`和`compose`相似，只不过它创建的函数是先调用第一个函数，然后再调用第二个，即是`g(fx(x))`

<pre class="brush: java; gutter: true">
scala> val ahemThenUmm = addAhem _ andThen addUmm _
ahemThenUmm: (String) => String = <function1>

scala> ahemThenUmm("well")
res1: String = well ahem umm
</pre>

## Currying vs Partial Application

## <a name="curryvspartial">Currying与Partial Application</a>

### case statements

### case语句

#### So just what are case statements?

It's a subclass of function called a PartialFunction.

#### 那么，case语句究竟是什么呢？

它是函数PartialFunction的一个子类。


#### What is a collection of multiple case statements?

They are multiple PartialFunctions composed together.

#### 那么一组case语句的集合又是什么呢？

它们是多个PartialFunction组合的结果。

## Understanding PartialFunction

A function works for every argument of the defined type. In other words, a function defined as (Int) => String takes any Int and returns a String.

A Partial Function is only defined for certain values of the defined type.  A Partial Function (Int) => String might not accept every Int.

<code>isDefinedAt</code> is a method on PartialFunction that can be used to determine if the PartialFunction will accept a given argument.

__Note__ <code>PartialFunction</code> is unrelated to a partially applied function that we talked about earlier.

*See Also* Effective Scala has opinions about <a href="http://twitter.github.com/effectivescala/#Functional programming-Partial functions">PartialFunction</a>.

## <a name="PartialFunction">理解PartialFunction</a>

一个函数会在所有参数都是指定的类型下才能正常执行。换句话说，一个定义为(Int)=>String的函数，只能接收一个Int类型的参数，并且返回一个String类型。

一个Partial函数只能处理参数所定义类型的某些值。例如，一个Partical函数(Int)=>String不一定能接收所有的Int作为参数。

`isDefinedAt`是PartialFunction的一个方法，它可以用来判断这个PartialFunction是否能够接收一个指定的参数。

__注意__`PartialFunction`和我们之前讨论的部分应用函数(partial applied function)没有关系。

**参考** 《Effective Scala》中关于<a href="http://twitter.github.com/effectivescala/#Functional programming-Partial functions">PartialFunction</a>的观点。

<pre class="brush: java; gutter: true">
scala> val one: PartialFunction[Int, String] = { case 1 => "one" }
one: PartialFunction[Int,String] = <function1>

scala> one.isDefinedAt(1)
res0: Boolean = true

scala> one.isDefinedAt(2)
res1: Boolean = false
</pre>

You can apply a partial function.

你可以调用一个partial函数。

<pre class="brush: java; gutter: true">
scala> one(1)
res2: String = one
</pre>

PartialFunctions can be composed with something new, called orElse, that reflects whether the PartialFunction is defined over the supplied argument.

Partial函数可以和orElse进行组合，它可以反映出这个Partial函数对于给定的参数是否有定义。

<pre class="brush: java; gutter: true">
scala> val two: PartialFunction[Int, String] = { case 2 => "two" }
two: PartialFunction[Int,String] = <function1>

scala> val three: PartialFunction[Int, String] = { case 3 => "three" }
three: PartialFunction[Int,String] = <function1>

scala> val wildcard: PartialFunction[Int, String] = { case _ => "something else" }
wildcard: PartialFunction[Int,String] = <function1>

scala> val partial = one orElse two orElse three orElse wildcard
partial: PartialFunction[Int,String] = <function1>

scala> partial(5)
res24: String = something else

scala> partial(3)
res25: String = three

scala> partial(2)
res26: String = two

scala> partial(1)
res27: String = one

scala> partial(0)
res28: String = something else
</pre>

### The mystery of case.

Last week we saw something curious. We saw a case statement used where a function is normally used.

### <a name="case">case之谜</a>
上周我们看了一些很奇怪的用法。我们见过case语句用在函数的位置上。

<pre class="brush: java; gutter: true">
scala> case class PhoneExt(name: String, ext: Int)
defined class PhoneExt

scala> val extensions = List(PhoneExt("steve", 100), PhoneExt("robey", 200))
extensions: List[PhoneExt] = List(PhoneExt(steve,100), PhoneExt(robey,200))

scala> extensions.filter { case PhoneExt(name, extension) => extension < 200 }
res0: List[PhoneExt] = List(PhoneExt(steve,100))
</pre>

Why does this work?

filter takes a function. In this case a predicate function of (PhoneExt) => Boolean.

A PartialFunction is a subtype of Function so filter can also take a PartialFunction!

为什么这样是可行的呢？

filter接收一个函数作为参数。所以这里应该是一个断言函数(PhoneExt) => Boolean.

这是因为ParitalFunction是Function的子类，所以filter可以接收一个PartialFunction作为参数！
