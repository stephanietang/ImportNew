
英文原文: [Scala School](http://twitter.github.com/scala_school/basics2.html)，翻译：[ImportNew](http://www.importnew.com) - [朱伟杰](http://www.importnew.com/author/zhuweijie)

This lesson covers:
* apply
* objects
* Functions are Objects
* packages
* pattern matching
* case classes
* try-catch-finally

这个章节的内容包括

* <a href="#apply">apply</a>
* <a href="#objects">对象</a>
* <a href="#fnobj">函数即对象</a>
* <a href="#packages">包</a>
* <a href="#match">模式匹配</a>
* <a href="#caseclass">case class</a>
* <a href="#exception">try-catch-finally</a>


## apply methods

apply methods give you a nice syntactic sugar for when a class or object has one main use.

## <a name="apply">apply方法</a>

当一个类或者对象有一个主要的用途的时候，apply方法可以提供一种很好的语法糖(syntactic sugar)。

<pre class="brush: java; gutter: true">
scala> class Foo {}
defined class Foo

scala> object FooMaker {
     |   def apply() = new Foo
     | }
defined module FooMaker

scala> val newFoo = FooMaker()
newFoo: Foo = Foo@5b83f762
</pre>

or
或者

<pre class="brush: java; gutter: true">
scala> class Bar {
     |   def apply() = 0
     | }
defined class Bar

scala> val bar = new Bar
bar: Bar = Bar@47711479

scala> bar()
res8: Int = 0
</pre>

Here our instance object looks like we're calling a method. More on that later!
这样来看实例对象，好像我们是在调用一个方法。后面会详细介绍这个！


## Objects

Objects are used to hold single instances of a class. Often used for factories.

## <a name="objects">对象</a>

对象都是用来保持一个类的单个实例。经常在工厂里用到。

<pre class="brush: java; gutter: true">
object Timer {
  var count = 0

  def currentCount(): Long = {
    count += 1
    count
  }
}
</pre>

How to use

怎么去使用

<pre class="brush: java; gutter: true">
scala> Timer.currentCount()
res0: Long = 1
</pre>

Classes and Objects can have the same name.  The object is called a 'Companion Object'.  We commonly use Companion Objects for Factories.

Here is a trivial example that only serves to remove the need to use 'new' to create an instance.

类和对象可以重名。这样的对象被称为“伴随对象（Companion Object）”。我们一般在工厂里使用伴随对象。

<pre class="brush: java; gutter: true">
class Bar(foo: String)

object Bar {
  def apply(foo: String) = new Bar(foo)
}
</pre>


## Functions are Objects

In Scala, we talk about object-functional programming often.  What does that mean? What is a Function, really?

A Function is a set of traits. Specifically, a function that takes one argument is an instance of a Function1 trait. This trait defines the <code>apply()</code> syntactic sugar we learned earlier, allowing you to call an object like you would a function.

## <a name="fnobj">函数即对象</a>

在Scala里，我们经常讨论对象-函数编程。它表示什么呢？函数究竟是什么？

函数是一系列的trait。确切地说，有一个参数的函数是Function1 trait的一个实例。这个trait定义了我们之前学到的`apply()`的语法糖，它允许你像调用函数一样调用对象。
<pre class="brush: java; gutter: true">
scala> object addOne extends Function1[Int, Int] {
     |   def apply(m: Int): Int = m + 1
     | }
defined module addOne

scala> addOne(1)
res2: Int = 2
</pre>

There is Function1 through 22.  Why 22?  It's an arbitrary magic number.  I've never needed a function with more than 22 arguments so it seems to work out.

在Scala里有Function1到22。为什么是22？这是一个任意的魔数。我从来没有遇到需要22个参数的函数，所以这个数字还是很有效的。

The syntactic sugar of apply helps unify the duality of object and functional programming.  You can pass classes around and use them as functions and functions are just instances of classes under the covers.

apply的语法糖使得对象和函数的编程能够组合在一起。你可以把对象作为参数进行传递，同时也可以把它们当作函数使用，而实际上函数也只不过是类的实例。


Does this mean that everytime you define a method in your class, you're actually getting an instance of Function*?  No, methods in classes are methods.  methods defined standalone in the repl are Function* instances.


那这样是不是意味着每次你在类里定义一个方法，那么你就会得到Function*的一个实例呢？不，类里的方法仅仅是普通的方法。在repl里单独定义的方法才是Function*的实例。

Classes can also extend Function and those instances can be called with ().

类也可以继承函数，然后这些类的实例就可以通过()来调用。

<pre class="brush: java; gutter: true">
scala> class AddOne extends Function1[Int, Int] {
     |   def apply(m: Int): Int = m + 1
     | }
defined class AddOne

scala> val plusOne = new AddOne()
plusOne: AddOne = <function1>

scala> plusOne(1)
res0: Int = 2
</pre>

A nice short-hand for <code>extends Function1[Int, Int]</code> is <code>extends (Int => Int)</code>

`extends Function1[Int,Int]`的一种比较好的简写是`extends (Int => Int)`

<pre class="brush: java; gutter: true">
class AddOne extends (Int => Int) {
  def apply(m: Int): Int = m + 1
}
</pre>

## Packages

You can organize your code inside of packages.

## <a name="packages">包</a>

你可以通过包来组织你的代码。

<pre class="brush: java; gutter: true">
package com.twitter.example
</pre>

at the top of a file will declare everything in the file to be in that package.

Values and functions cannot be outside of a class or object.  Objects are a useful tool for organizing static functions.

在一个文件的顶部声明包，那么这个文件里的所有的东西都是在这个包里。

值和函数不能在类和对象外面。对象对于组织静态函数是很有用的。

<pre class="brush: java; gutter: true">
package com.twitter.example

object colorHolder {
  val BLUE = "Blue"
  val RED = "Red"
}
</pre>

Now you can access the members directly

现在你可以直接访问里面的成员

<pre class="brush: java; gutter: true">
println("the color is: " + com.twitter.example.colorHolder.BLUE)
</pre>

Notice what the scala repl says when you define this object:

当你在scala repl里定义这个对象时，它会提示什么：
<pre class="brush: java; gutter: true">
scala> object colorHolder {
     |   val Blue = "Blue"
     |   val Red = "Red"
     | }
defined module colorHolder
</pre>

This gives you a small hint that the designers of Scala designed objects to be part of Scala's module system.

上面的信息暗示你scala的设计者把对象设计成为Scala的模块系统的一部分。

## Pattern Matching


One of the most useful parts of Scala.

Matching on values

## <a name="match">模式匹配</a>

Scala中最有用的一个功能。

对值进行匹配

<pre class="brush: java; gutter: true">
val times = 1

times match {
  case 1 => "one"
  case 2 => "two"
  case _ => "some other number"
}
</pre>

Matching with guards

受保护的匹配

<pre class="brush: java; gutter: true">
times match {
  case i if i == 1 => "one"
  case i if i == 2 => "two"
  case _ => "some other number"
}
</pre>

Notice how we captured the value in the variable 'i'.

注意我们如何捕获变量'i'里的值。

The <code>_</code> in the last case statement is a wildcard; it
ensures that we can handle any statement. Otherwise you will suffer a
runtime error if you pass in a number that doesn't match. We discuss
this more later.

最后的一条语句中的`_`是一个通配符；它保证我们能够处理任何输入。否则的话，如果我们传入一个不匹配的值，将会出现runtime error。我们后面会继续讨论这个。


*See Also* Effective Scala has opinions about <a href="http://twitter.github.com/effectivescala/#Functional programming-Pattern matching">when to use pattern matching</a> and <a href="http://twitter.github.com/effectivescala/#Formatting-Pattern matching">pattern matching formatting</a>. A Tour of Scala describes <a href="http://www.scala-lang.org/node/120">Pattern Matching</a>

*参考* 《Effective Scala》中有关于<a href="http://twitter.github.com/effectivescala/#Functional programming-Pattern matching">什么时候使用模式匹配</>和<a href="http://twitter.github.com/effectivescala/#Formatting-Pattern matching">模式匹配格式</a>的讨论。《A Tour of Scala》中讲解了<a href="http://www.scala-lang.org/node/120">模式匹配</a>。

### Matching on type

You can use <code>match</code> to handle values of different types differently.

### 针对类型进行匹配

你可以使用`match`来对不同的类型的值进行不同的处理。

<pre class="brush: java; gutter: true">
def bigger(o: Any): Any = {
  o match {
    case i: Int if i < 0 => i - 1
    case i: Int => i + 1
    case d: Double if d < 0.0 => d - 0.1
    case d: Double => d + 0.1
    case text: String => text + "s"
  }
}
</pre>

### Matching on class members

Remember our calculator from earlier.

Let's classify them according to type.

### 对类的成员进行匹配

回想一下我们之前的caculator的示例。

我们通过它们的类型来进行归类。

<pre class="brush: java; gutter: true">
def calcType(calc: Calculator) = calc match {
  case calc.brand == "hp" && calc.model == "20B" => "financial"
  case calc.brand == "hp" && calc.model == "48G" => "scientific"
  case calc.brand == "hp" && calc.model == "30B" => "business"
  case _ => "unknown"
}
</pre>

Wow, that's painful. Thankfully Scala provides some nice tools specifically for this.

噢，这样写太麻烦了。幸运的是Scala提供了一些有用的工具来处理这种场景。

## Case Classes

case classes are used to conveniently store and match on the contents of a class. You can construct them without using new.

## <a name="caseclass">Case Classes</a>

case class可以用来方便地存储和匹配类的内容。你不需要用`new`来构造它们。


<pre class="brush: java; gutter: true">
scala> case class Calculator(brand: String, model: String)
defined class Calculator

scala> val hp20b = Calculator("hp", "20b")
hp20b: Calculator = Calculator(hp,20b)

</pre>

case classes automatically have equality and nice toString methods based on the constructor arguments.

case class会自动根据传入的参数生成equlity和toString方法。

<pre class="brush: java; gutter: true">
scala> val hp20b = Calculator("hp", "20b")
hp20b: Calculator = Calculator(hp,20b)

scala> val hp20B = Calculator("hp", "20b")
hp20B: Calculator = Calculator(hp,20b)

scala> hp20b == hp20B
res6: Boolean = true
</pre>

case classes can have methods just like normal classes.

case class可以像正常类一样拥有方法。

###### Case Classes with pattern matching

###### 通过Case Class来进行模式匹配

case classes are designed to be used with pattern matching.  Let's simplify our calculator classifier example from earlier.

case class的设计就是为了进行模式匹配。我们来通过它简化之前的calculator 归类程序。

<pre class="brush: java; gutter: true">
val hp20b = Calculator("hp", "20B")
val hp30b = Calculator("hp", "30B")

def calcType(calc: Calculator) = calc match {
  case Calculator("hp", "20B") => "financial"
  case Calculator("hp", "48G") => "scientific"
  case Calculator("hp", "30B") => "business"
  case Calculator(ourBrand, ourModel) => "Calculator: %s %s is of unknown type".format(ourBrand, ourModel)
}
</pre>

Other alternatives for that last match

最后一行的另外一种写法

<pre class="brush: java; gutter: true">
  case Calculator(_, _) => "Calculator of unknown type"
</pre>

  OR we could simply not specify that it's a Calculator at all.

  或者我们不用把它当作Calculator来处理。

<pre class="brush: java; gutter: true">
  case _ => "Calculator of unknown type"
</pre>

  OR we could re-bind the matched value with another name

  也或者，我们可以用另外的名字来绑定这个匹配。

<pre class="brush: java; gutter: true">
  case c@Calculator(_, _) => "Calculator: %s of unknown type".format(c)
</pre>

## Exceptions

Exceptions are available in Scala via a try-catch-finally syntax that uses pattern matching.

## <a name="exception">异常</a>

在Scala里，异常的处理是通过一个使用模式匹配的try-catch-finally语法进行处理的。


<pre class="brush: java; gutter: true">
try {
  remoteCalculatorService.add(1, 2)
} catch {
  case e: ServerIsDownException => log.error(e, "the remote calculator service is unavailble. should have kept your trustry HP.")
} finally {
  remoteCalculatorService.close()
}
</pre>

<code>try</code>s are also expression-oriented

`try`也是面向表达式的

<pre class="brush: java; gutter: true">
val result: Int = try {
  remoteCalculatorService.add(1, 2)
} catch {
  case e: ServerIsDownException => {
    log.error(e, "the remote calculator service is unavailble. should have kept your trustry HP.")
    0
  }
} finally {
  remoteCalculatorService.close()
}
</pre>

This is not an example of excellent programming style, just an example of try-catch-finally resulting in expressions like most everything else in Scala.

这个并不是一种很好的编码方式，而只是try-catch-finally返回表达式的一个示例，正如Scala里其他的一样。

Finally will be called after an exception has been handled and is not part of the expression.

Finally里的语句会在异常处理完之后执行，并且它不是表达式的一部分。
