这个章节的内容包含

* 关于这个教程
* 表达式
* 值
* 函数
* 类
* 基本继承
* traints
* 类型
###About this class

The first few weeks will cover basic syntax and concepts.  Then we'll start to open it up with more exercises.

Some examples will be given as if written in the interpreter, others as if written in a source file.

Having an interpreter available makes it easy to explore a problem space.

##关于这个教程

在最初的几个星期里，我们会讲解基本的语法和概念。然后我们会通过更多的练习来展示这些概念。有些例子是通过解释器里的几行代码来展示的，有些则是通过源码的形式进行展示。

你最好安装一个scala解释器，它可以方便你探索问题。

### Why Scala?
* Expressive
    * First-class functions
    * Closures
* Concise
    * Type inference
    * Literal syntax for function creation
* Java interopability
    * Can reuse java libraries
    * Can reuse java tools
    * No performance penalty

###为什么用Scala？
* 富有表现力
	* 一等函数
	* 闭包
* 简洁
	* 类型接口
	* 
* 可以和Java进行互操作
	* 可以复用Java的类库
	* 可以复用Java的工具
	* 没有额外的性能问题

### How Scala?

* Compiles to java bytecode
* Works with any standard JVM
    * Or even some non-standard JVMs like Dalvik
    * Scala compiler written by author of Java compiler

### Scala 怎么样？
* Scala代码会编译成Java 字节码
* 可以很好的运行在任何标准的JVM上
	* 甚至可以运行在非标准的JVM上，例如Dalvik
	* Scala的编译器是由Java编译器的作者编写的


### Think Scala

Scala is not just a nicer Java.  You should learn it with a fresh mind, you will get more out of these classes.
### 按照Scala的方式思考
Scala 不仅仅是Java的改善。你需要用一个全新的思维方式来学习它，：你会从这些课程中学到更多。


### Start the Interpreter

Start the included <code>sbt console</code>.

### 启动解释器
启动内置的<code>sbt console</code>


<pre>
$ sbt console

[...]

Welcome to Scala version 2.8.0.final (Java HotSpot(TM) 64-Bit Server VM, Java 1.6.0_20).
Type in expressions to have them evaluated.
Type :help for more information.

scala>
</pre>


###(#expressions). Expressions

<pre>
scala> 1 + 1
res0: Int = 2
</pre>

res0 is an automatically created value name given by the interpreter to the result of your expression.  It has the type Int and contains the Integer 2.

(Almost) everything in Scala is an expression.

h2(#val). Values

You can give the result of an expression a name.

<pre>
scala> val two = 1 + 1
two: Int = 2
</pre>

You cannot change the binding to a val.

### 表达式
<pre>
scala> 1 + 1
res0: Int = 2
</pre>

res0是由解释器自动给你的表达式的结果生成的变量名。它的类型是Int，值是2.
在Scala里（几乎）所有的东西都是一个表达式。

### 值
你可以给表达式的结果设定一个名字。
<pre>
scala> val two = 1 + 1
two: Int = 2
</pre>
你不可以改变val定义的变量的绑定。



h3. Variables

If you need to change the binding, you can use a <code>var</code> instead

### 变量
如果你需要改变绑定，你可以使用<code>var</code>来代替。

<pre>
scala> var name = "steve"
name: java.lang.String = steve

scala> name = "marius"
name: java.lang.String = marius
</pre>

h2(#functions). Functions

You can create functions with def.

### 函数
你可以用def来创建函数

<pre>
scala> def addOne(m: Int): Int = m + 1
addOne: (m: Int)Int
</pre>

In Scala, you need to specify the type signature for function parameters.  The interpreter happily repeats the type signature back to you.

在Scala里，你需要给函数的参数指定类型签名。解释器会回显你指定的类型签名。

<pre>
scala> val three = addOne(2)
three: Int = 3
</pre>

You can leave off parens on functions with no arguments
对于没有参数的函数，在调用的时候可以省略掉括号

<pre>
scala> def three() = 1 + 2
three: ()Int

scala> three()
res2: Int = 3

scala> three
res3: Int = 3
</pre>

h3. Anonymous Functions

You can create anonymous functions.

### 匿名海曙
你可以通过下面的方式创建匿名函数

<pre>
scala> (x: Int) => x + 1
res2: (Int) => Int = <function1>
</pre>

This function adds 1 to an Int named x.
这个函数给一个Int型的整数加一。

<pre>
scala> res2(1)
res3: Int = 2
</pre>

You can pass anonymous functions around or save them into vals.
你可以把匿名函数作为参数进行传递，或者把它们保存在val中。

<pre>
scala> val addOne = (x: Int) => x + 1
addOne: (Int) => Int = <function1>

scala> addOne(1)
res4: Int = 2
</pre>

If your function is made up of many expressions, you can use {} to give yourself some breathing room.
如果你的函数是由多个表达式组成的，你可以把它们写在一对{}里。

<pre>
def timesTwo(i: Int): Int = {
  println("hello world")
  i * 2
}
</pre>

This is also true of an anonymous function
也可以通过下面的方式来创建一个匿名函数

<pre>
scala> { i: Int =>
  println("hello world")
  i * 2
}
res0: (Int) => Int = <function1>
</pre>

You will see this syntax often used when passing an anonymous function as an argument.
在把匿名函数作为一个参数进行传递时，你会经常看到这种语法。

h3. Partial application

You can partially apply a function with an underscore, which gives you another function. Scala uses the underscore to mean different things in different contexts, but you can usually think of it as an unnamed magical wildcard. In the context of `{ _ + 2 }` it means an unnamed parameter. You can use it like so:

### 部分应用(偏函数)

你可以使用下划线来部分应用一个函数，这样就可以产生一个新的函数。Scala用下划线来表示在不同上下文下的不同内容，不过你可以任务它是一个没有名字的神奇通配符。在`{ _ + 2}`这样的上下文里，它表示一个没有名字的参数。你也可以这样来使用它：

<pre>
scala> def adder(m: Int, n: Int) = m + n
adder: (m: Int,n: Int)Int
</pre>

<pre>
scala> val add2 = adder(2, _:Int)
add2: (Int) => Int = <function1>

scala> add2(3)
res50: Int = 5
</pre>

You can partially apply any argument in the argument list, not just the last one.
你可以部分应用参数列表里的任何参数，而并非只是最后一个。

h3. Curried functions

Sometimes it makes sense to let people apply some arguments to your function now and others later.

Here's an example of a function that lets you build multipliers of two numbers together. At one call site, you'll decide which is the multiplier and at a later call site, you'll choose a multipicand.


### Curried functions
有时候，使用者先给你的函数传入部分参数，后面再传入其他的参数，这样做是很有用的。

<pre>
scala> def multiply(m: Int)(n: Int): Int = m * n
multiply: (m: Int)(n: Int)Int
</pre>

You can call it directly with both arguments.
你可以直接把两个参数都传入。

<pre>
scala> multiply(2)(3)
res0: Int = 6
</pre>

You can fill in the first parameter and partially apply the second.
你也可以先传入第一个参数，然后部分应用第二个参数。

<pre>
scala> val timesTwo = multiply(2) _
timesTwo: (Int) => Int = <function1>

scala> timesTwo(3)
res1: Int = 6
</pre>

You can take any function of multiple arguments and curry it. Let's try with our earlier <code>adder</code>
你可以把任何有多个参数的函数当作curried function来使用。我们来用之前的<code>adder</code>做个示范。

<pre>
scala> (adder _).curried
res1: (Int) => (Int) => Int = <function1>
</pre>

h3. Variable length arguments

There is a special syntax for methods that can take parameters of a repeated type. To apply String's `capitalize` function to several strings, you might write:


###  可变长度的参数
对于方法，还有一个特别的语法用来接收多个相同类型的参数。对于一组字符串，要对它们应用String的`capitalize`函数，你可以这样写：

<pre>
def capitalizeAll(args: String*) = {
  args.map { arg =>
    arg.capitalize
  }
}

scala> capitalizeAll("rarity", "applejack")
res2: Seq[String] = ArrayBuffer(Rarity, Applejack)
</pre>

h2(#class). Classes

### 类

<pre>
scala> class Calculator {
     |   val brand: String = "HP"
     |   def add(m: Int, n: Int): Int = m + n
     | }
defined class Calculator
定一个一个Calculator类

scala> val calc = new Calculator
calc: Calculator = Calculator@e75a11

scala> calc.add(1, 2)
res1: Int = 3

scala> calc.brand
res2: String = "HP"
</pre>

Contained are examples are defining methods with def and fields with val.  methods are just functions that can access the state of the class.
上面的例子中，展示了通过def定义方法已经使用val定义成员变量。方法也是函数，只不过它可以访问类的状态。



h3. Constructor
### Constructor

Constructors aren't special methods, they are the code outside of method definitions in your class.  Let's extend our Calculator example to take a constructor argument and use it to initialize internal state.
Constructor并不是特殊的方法，它们是当前类外部的一段代码。我们来扩展一下Calculator的例子，使得它能够接受一个参数，并且使用这个参数来初始化类的内部状态。

<pre>
class Calculator(brand: String) {
  /    *
   * A constructor.
   */
  val color: String = if (brand == "TI") {
    "blue"
  } else if (brand == "HP") {
    "black"
  } else {
    "white"
  }

  // An instance method.
  def add(m: Int, n: Int): Int = m + n
}
</pre>

Note the two different styles of comments.
注意两种不同方式的注释。

You can use the constructor to construct an instance:
你可以使用这个constructor来构造一个实例：

<pre>
scala> val calc = new Calculator("HP")
calc: Calculator = Calculator@1e64cc4d

scala> calc.color
res0: String = black
</pre>

h3. Expressions

Our BasicCalculator example gave an example of how Scala is expression-oriented.  The value color was bound based on an if/else expression. Scala is highly expression-oriented: most things are expressions rather than statements.

### 表达式
这个BasicCalculator例子向我们展示了Scala是如何面向表达式的。变量color的值取决于一个if/else表达式的结果。Scala是高度面向表达式的：大部分的语句都是表达式，而非声明。

h3. Aside: Functions vs Methods

Functions and methods are largely interchangeable. Because functions and methods are so similar, you might not remember whether that <em>thing</em> you call is a function or a method. When you bump into a difference between methods and functions, it might confuse you.

### 题外话：函数 vs 方法
函数和方法在很大的程度上是可以相互替代的。因为函数和方法是如此的相似，以至于你可能无法分清你调用的对象是一个方法还是一个函数。当你考虑函数和方法的区别时，你就开始混乱了。


<pre>
scala> class C {
     |   var acc = 0
     |   def minc = { acc += 1 }
     |   val finc = { () => acc += 1 }
     | }
defined class C

scala> val c = new C
c: C = C@1af1bd6

scala> c.minc // calls c.minc()

scala> c.finc // returns the function as a value:
res2: () => Unit = <function0>
</pre>

When you can call one "function" without parentheses but not another, you might think <em>Whoops, I thought I knew how Scala functions worked, but I guess not. Maybe they sometimes need parentheses?</em> You might understand functions, but be using a method.

当你调用一个“函数”但是没有带上括号但实际上没效果时，你也许会想<em>噢，我想我明白Scala的函数是怎么起作用的，但是我认为也许不是这样的。或许有时候是需要括号的？</em>或许你对函数比较了解，但是实际上却在使用一个方法。


In practice, you can do great things in Scala while remaining hazy on the difference between methods and functions. If you're new to Scala and read <a href="https://www.google.com/search?q=difference+scala+function+method">explanations of the differences</a>, you might have trouble following them. That doesn't mean you're going to have trouble using Scala. It just means that the difference between functions and methods is subtle enough such that explanations tend to dig into deep parts of the language.

实际上，就是你在区别函数和方法的问题上犯迷糊，你还是可以用Scala来做一些很出色的事情。如果你是一个Scala新手，并且你在google上阅读<a href="https://www.google.com/search?q=difference+scala+function+method">它们的区别</a>，你可能会不是很明白。但是这并不表示你会用不好Scala。它只能说明函数和方法的区别是比较微妙的，如果要解释的话，那么就需要对语言进行深入了解。


h2(#extends). Inheritance
## 继承

<pre>
class ScientificCalculator(brand: String) extends Calculator(brand) {
  def log(m: Double, base: Double) = math.log(m) / math.log(base)
}
</pre>

*See Also* Effective Scala points out that a <a href="http://twitter.github.com/effectivescala/#Types%20and%20Generics-Type%20aliases">Type alias</a> is better than <code>extends</code> if the subclass isn't actually different from the superclass. A Tour of Scala describes <a href="http://www.scala-lang.org/node/125">Subclassing</a>.
*参考* 《Effective Scala》中指出<a href="http://twitter.github.com/effectivescala/#Types%20and%20Generics-Type%20aliases">类型别名(Type alias)</a>比<code>extends</code>更好，如果子类和父类并没有什么大的区别的话。《A Tour of Scala》中讲述了<a href="http://www.scala-lang.org/node/125">Subclassing</a>子类化(Subclassing)</a>的技术。

h3. Overloading methods
### 重载方法

<pre>
class EvenMoreScientificCalculator(brand: String) extends ScientificCalculator(brand) {
  def log(m: Int): Double = log(m, math.exp(1))
}
</pre>

h3. Abstract Classes

You can define an <em>abstract class</em>, a class that defines some methods but does not implement them. Instead, subclasses that extend the abstract class define these methods. You can't create an instance of an abstract class.

### 抽象方法

<pre>
scala> abstract class Shape {
     |   def getArea():Int    // subclass should define this
     | }
defined class Shape

scala> class Circle(r: Int) extends Shape {
     |   def getArea():Int = { r * r * 3 }
     | }
defined class Circle

scala> val s = new Shape
<console>:8: error: class Shape is abstract; cannot be instantiated
       val s = new Shape
               ^

scala> val c = new Circle(2)
c: Circle = Circle@65c0035b
</pre>

h2(#trait). Traits

<code>traits</code> are collections of fields and behaviors that you can extend or mixin to your classes.
## Traits
<code>traints</code>表示一系列可以扩展或者混入到你的类里的成员和行为。

<pre>
trait Car {
  val brand: String
}

trait Shiny {
  val shineRefraction: Int
}
</pre>

<pre>
class BMW extends Car {
  val brand = "BMW"
}
</pre>

One class can extend several traits using the <code>with</code> keyword:
通过<code>with</code>关键字，一个类可以扩展多个traint：

<pre>
class BMW extends Car with Shiny {
  val brand = "BMW"
  val shineRefraction = 12
}
</pre>

*See Also* Effective Scala has opinions about <a href="http://twitter.github.com/effectivescala/#Object oriented programming-Traits">trait</a>.
*参考* 《Effective Scala》中关于 <a href="http://twitter.github.com/effectivescala/#Object oriented programming-Traits">trait</a>的观点.

*When do you want a Trait instead of an Abstract Class?* If you want to define an interface-like type, you might find it difficult to choose between a trait or an abstract class. Either one lets you define a type with some behavior, asking extenders to define some other behavior. Some rules of thumb:

*什么时候你需要使用Trait代替抽象类?* 如果你想定义一个类似接口的类型，那么你很难在trait和抽象类之间做出选择。它们都可以让你定义一个具有某些行为的类型，然后要求扩展者去实习其他的行为。下面是一些经验法则：

<ul>
<li>Favor using traits. It's handy that a class can extend several traits; a class can extend only one class.
<li>If you need a constructor parameter, use an abstract class. Abstract class constructors can take paramaters; trait constructors can't. For example, you can't say <code>trait t(i: Int) {}</code>; the <code>i</code> parameter is illegal.
</ul>
<ul>
  <li>优先使用traint。一个类可以扩展多个traint，但是只能扩展一个抽象类。
  <li>如果你需要在构造类的时候传入参数的话，那就是用抽象类。抽象类的构造器可以传入参数，trait则不能。例如，你不能这样写<code>trait t(i:Int) {} </code>；参数<code>i</code>是非法的。

You are not the first person to ask this question. See fuller answers at "stackoverflow:Scala traits vs abstract classes":http://stackoverflow.com/questions/1991042/scala-traits-vs-abstract-classes, "Difference between Abstract Class and Trait":http://stackoverflow.com/questions/2005681/difference-between-abstract-class-and-trait, and "Programming in Scala: To trait, or not to trait?":http://www.artima.com/pins1ed/traits.html#12.7

你并不是第一个问这个问题的人。你可以在"stackoverflow:Scala traits vs abstract classes":http://stackoverflow.com/questions/1991042/scala-traits-vs-abstract-classes, "Difference between Abstract Class and Trait":http://stackoverflow.com/questions/2005681/difference-between-abstract-class-and-trait, and "Programming in Scala: To trait, or not to trait?":http://www.artima.com/pins1ed/traits.html#12.7查看更加完整的解释。

h2(#types). Types

Earlier, you saw that we defined a function that took an <code>Int</code> which is a type of Number. Functions can also be generic and work on any type. When that occurs, you'll see a type parameter introduced with the square bracket syntax. Here's an example of a Cache of generic Keys and Values.

## 类型
在前面的例子里，你见过我们定义了一个函数，它接收一个<code>Int</code>参数，而Int的类型是Number。函数也可以是泛型的，并且可以作用于任何类型。当你遇到这种情况时，你会看到类型参数是放在方括号里的。下面是一个支持泛型键-值的Cache示例。

<pre>
trait Cache[K, V] {
  def get(key: K): V
  def put(key: K, value: V)
  def delete(key: K)
}
</pre>

Methods can also have type parameters introduced.
方法也可以引入类型参数。

<pre>
def remove[K](key: K)
</pre>