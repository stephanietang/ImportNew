英文原文: [Scala School](http://twitter.github.io/scala_school/type-basics.html)，翻译：[ImportNew](http://www.importnew.com) - [朱伟杰](http://www.importnew.com/author/zhuweijie)

This lesson covers:

* "What are static types?":#background
* "Types in Scala":#scala
* "Parametric Polymorphism":#parametricpoly
* "Type inference: Hindley-Milner vs. local type inference":#inference
* "Variance":#variance
* "Bounds":#bounds
* "Quantification":#quantification

这一节的内容包含：

* <a href="#background">什么是静态类型？</a>
* <a href="#type">Scala中的类型</a>
* <a href="#parametricpoly">参数多态</a>
* <a href="#inference">参数推导：Hindley-Milener 与 局部类型推导</a>
* <a href="#variance">Variance</a>
* <a href="#bounds">范围(Bounds)</a>
* <a href="#quantification">量化(Quantification)</a>

h2(#background). What are static types?  Why are they useful?

According to Pierce: "A type system is a syntactic method for automatically checking the absence of certain erroneous behaviors by classifying program phrases according to the kinds of values they compute."

Types allow you to denote function domain & codomains. For example, from mathematics, we are used to seeing:

## <a name="background">什么是静态类型？为什么它们很有用？</a>

根据Picrce的说法：“类型系统是一个可以根据代码段计算出来的值对它们进行分类，然后通过语法的手段来自动检测程序错误的系统。”


类型可以让你表示函数的域和值域。例如，在数学里，我们经常看到下面的函数：


<pre class="brush: java; gutter: true">
f: R -> N
</pre>

this tells us that function "f" maps values from the set of real numbers to values of the set of natural numbers.

In the abstract, this is exactly what _concrete_ types are.  Type systems give us some more powerful ways to express these sets.

Given these annotations, the compiler can now _statically_ (at compile time) verify that the program is _sound_. That is, compilation will fail if values (at runtime) will not comply to the constraints imposed by the program.

Generally speaking, the typechecker can only guarantee that _unsound_ programs do not compile. It cannot guarantee that every sound program _will_ compile.

With increasing expressiveness in type systems, we can produce more reliable code because it allows us to prove invariants about our program before it even runs (modulo bugs in the types themselves, of course!). Academia is pushing the limits of expressiveness very hard, including value-dependent types!

Note that all type information is removed at compile time. It is no longer needed. This is called erasure.

这个定义告诉我们函数"f"的作用是把实数集里的数映射到自然集里。

抽象地说，这才是_具体_意义上的类型。类型系统给了我们一些表示这些集合的更强大的方式。

有了这些类型标识，编译器现在可以 __静态地__(在编译期)判断这个程序是__正确的__。换句话说，如果一个值（在运行期）不能够满足程序里的限制条件的话，那么在编译期就会出错。

通常来说，类型检测器(typechecker)只能够保证__不正确的__的程序不能通过编译。但是，它不能够保证所有正确的程序__都能__通过编译。

由于类型系统的表达能力不断增加，使得我们能够生成出更加可靠的代码，因为它使得我们能够控制程序上的不可变，即是是程序还没有运行的情况下（在类型上限制bug的出现）。学术界一直在努力突破类型系统的表达能力的极限，包含值相关的类型。


h2(#scala). Types in Scala

Scala's powerful type system allows for very rich expression. Some of its chief features are:

* *parametric polymorphism* roughly, generic programming
* *(local) type inference* roughly, why you needn't say <code>val i: Int = 12: Int</code>
* *existential quantification* roughly, defining something _for some_ unnamed type
* *views* we'll learn these next week; roughly, "castability" of values of one type to another

## <a name="type">Scala中的类型</a>

Scala强大的类型系统让我们可以使用更具有表现力的表达式。一些主要的特点如下：

* 支持**参数多态**，泛型编程
* 支持**（局部）类型推导**，这就是你为什么不需要写`val i: Int = 12: Int`
* 支持**存在量化(existential quantification)**，给一些没有名称的类型定义一些操作
* 支持**视图**。 我们下个星期再讨论；给定的值从一个类型到其他类型的“可转换性”

h2(#parametricpoly). Parametric polymorphism

Polymorphism is used in order to write generic code (for values of different types) without compromising static typing richness.

For example, without parametric polymorphism, a generic list data structure would always look like this (and indeed it did look like this in Java prior to generics):

## <a name="parametricpoly">参数多态</a>

多态可以用来编写泛型代码（用于处理不同类型的值)，并且没必要给静态类型做出让步。

例如，没有参数多态的话，一个泛型的列表数据结构通常会是下面这样的写法（在Java还没有泛型的时候，确实是这样的）：

<pre class="brush: java; gutter: true">
scala> 2 :: 1 :: "bar" :: "foo" :: Nil
res5: List[Any] = List(2, 1, bar, foo)
</pre>

Now we cannot recover any type information about the individual members.

现在我们可以恢复每个元素的类型信息

<pre class="brush: java; gutter: true">
scala> res5.head
res6: Any = 2
</pre>

And so our application would devolve into a series of casts ("asInstanceOf[]") and we would lack type safety (because these are all dynamic).

Polymorphism is achieved through specifying _type variables_.

这样一来，我们的应用里会包含一系列的类型转换("asInstanceOf[]")，代码会缺少类型安全（因为他们都是动态类型的）。

多态是通过指定__类型变量__来达到的。

<pre class="brush: java; gutter: true">
scala> def drop1[A](l: List[A]) = l.tail
drop1: [A](l: List[A])List[A]

scala> drop1(List(1,2,3))
res1: List[Int] = List(2, 3)
</pre>

h3. Scala has rank-1 polymorphism

Roughly, this means that there are some type concepts you'd like to express in Scala that are "too generic" for the compiler to understand. Suppose you had some function

### 多态是scala里的一等公民

大致上来说，这意味着有一些你想在Scala里表达的类型概念会显得“太过于泛型”了，导致编译器无法理解。假如你有这样一个函数：

<pre class="brush: java; gutter: true">
def toList[A](a: A) = List(a)
</pre>

which you wished to use generically:

你想要按照泛型的方式来使用它：

<pre class="brush: java; gutter: true">
def foo[A, B](f: A => List[A], b: B) = f(b)
</pre>

This does not compile, because all type variables have to be fixed at the invocation site. Even if you "nail down" type <code>B</code>,

但是这样会编译不过，因为所有的类型变量在运行期必须是确定的。

<pre class="brush: java; gutter: true">
def foo[A](f: A => List[A], i: Int) = f(i)
</pre>

...you get a type mismatch.

...你会看到一个类型不匹配的错误

h2(#inference). Type inference

A traditional objection to static typing is that it has much syntactic overhead. Scala alleviates this by providing _type inference_.

The classic method for type inference in functional programming languages is _Hindley-Milner_, and it was first employed in ML.

Scala's type inference system works a little differently, but it's similar in spirit: infer constraints, and attempt to unify a type.

## <a name="inference">类型推导</a>

对于静态类型的一个比较常见的缺陷就是有太多的类型语法。Scala提供了__类型推导__来解决这个问题。

函数式语言里比较经典的类型推导的方法是 _Hindlry-Milner_，并且它是在ML里首先使用的。

Scala的类型推导有一点点不同，不过思想上是一致的：推导所有的约束条件，然后统一到一个类型上。

In Scala, for example, you cannot do the following:

在Scala里，例如，你不能这样写：

<pre class="brush: java; gutter: true">
scala> { x => x }
<console>:7: error: missing parameter type
       { x => x }
</pre>

Whereas in OCaml, you can:

但是在OCaml里，你可以:

<pre class="brush: java; gutter: true">
# fun x -> x;;
- : 'a -> 'a = <fun>
</pre>

In scala all type inference is _local_. Scala considers one expression at a time. For example:

在Scala里，所有的类型推导都是__局部的__。Scala一次只考虑一个表达式。例如：

<pre class="brush: java; gutter: true">
scala> def id[T](x: T) = x
id: [T](x: T)T

scala> val x = id(322)
x: Int = 322

scala> val x = id("hey")
x: java.lang.String = hey

scala> val x = id(Array(1,2,3,4))
x: Array[Int] = Array(1, 2, 3, 4)
</pre>

Types are now preserved, The Scala compiler infers the type parameter for us. Note also how we did not have to specify the return type explicitly.

在这里，类型都被隐藏了。Scala编译器自动推导参数的类型。注意我们也没有必要显示指定返回值的类型了。

h2(#variance). Variance

Scala's type system has to account for class hierarchies together with polymorphism.  Class hierarchies allow the expression of subtype relationships. A central question that comes up when mixing OO with polymorphism is: if <tt>T'</tt> is a subclass of <tt>T</tt>, is <tt>Container[T']</tt> considered a subclass of <tt>Container[T]</tt>? Variance annotations allow you to express the following relationships between class hierarchies & polymorphic types:

## <a name="variance">Varicace</a>

Scala的类型系统需要把类的继承关系和多态结合起来。类的继承使得类之间存在父子的关系。当把面向对象和多态结合在一起时，一个核心的问题就出来了：如果<tt>T'</tt>是<tt>T</tt>的子类，那么<tt>Container[T']</tt>是不是<tt>Container[T]</tt>的子类呢？Variance注释允许你在类继承和多态类型之间表达下面的这些关系：

<table>
	<tbody><tr>
		<td>                </td>
		<td><strong>Meaning</strong>                     </td>
		<td> <strong>Scala notation</strong></td>
	</tr>
	<tr>
		<td><strong>covariant(协变)</strong>     </td>
		<td>C[T’] is a subclass of C[T]   </td>
		<td> [+T]</td>
	</tr>
	<tr>
		<td><strong>contravariant(逆变)</strong> </td>
		<td>C[T] is a subclass of C[T’]   </td>
		<td> [-T]</td>
	</tr>
	<tr>
		<td><strong>invariant(不变)</strong>     </td>
		<td>C[T] and C[T’] are not related</td>
		<td> [T]</td>
	</tr>
</tbody></table>

<table>
	<tbody><tr>
		<td>                </td>
		<td><strong>含义</strong>                     </td>
		<td> <strong>Scala中的标记</strong></td>
	</tr>
	<tr>
		<td><strong>covariant(协变)</strong>     </td>
		<td>C[T’]是C[T]的子类</td>
		<td> [+T]</td>
	</tr>
	<tr>
		<td><strong>contravariant(逆变)</strong> </td>
		<td>C[T]是C[T’]子类</td>
		<td> [-T]</td>
	</tr>
	<tr>
		<td><strong>invariant(不变)</strong>     </td>
		<td>C[T]和C[T’]不相关</td>
		<td> [T]</td>
	</tr>
</tbody></table>

The subtype relationship really means: for a given type T, if T' is a subtype, can you substitute it?

子类关系的真正意思是：对于一个给定的类型T，如果T'是它的子类，那么T'可以代替T吗？

<pre class="brush: java; gutter: true">
scala> class Contravariant[-A]
defined class Contravariant

scala> val cv: Contravariant[String] = new Contravariant[AnyRef]
cv: Contravariant[AnyRef] = Contravariant@49fa7ba

scala> val fail: Contravariant[AnyRef] = new Contravariant[String]
<console>:6: error: type mismatch;
 found   : Contravariant[String]
 required: Contravariant[AnyRef]
       val fail: Contravariant[AnyRef] = new Contravariant[String]
                                     ^
</pre>

Contravariance seems strange. When is it used? Somewhat surprising!

逆变(Contravariance)看起来比较奇怪。什么时候要用到它呢？确实让人感到惊讶！
<pre class="brush: java; gutter: true">
trait Function1 [-T1, +R] extends AnyRef
</pre>

If you think about this from the point of view of substitution, it makes a lot of sense. Let's first define a simple class hierarchy:

如果你从替代的角度来看这个的话，非常容易理解这一点。我们首先来定义一个简单的类继承关系：

<pre class="brush: java; gutter: true">
scala> class Animal { val sound = "rustle" }
defined class Animal

scala> class Bird extends Animal { override val sound = "call" }
defined class Bird

scala> class Chicken extends Bird { override val sound = "cluck" }
defined class Chicken
</pre>

Suppose you need a function that takes a <code>Bird</code> param:

假设你需要一个接收`Bird`类型参数的函数：

<pre class="brush: java; gutter: true">
scala> val getTweet: (Bird => String) = // TODO
</pre>

The standard animal library has a function that does what you want, but it takes an <code>Animal</code> parameter instead.  In most situations, if you say "I need a ___, I have a subclass of ___", you're OK. But function parameters are contravariant. If you need a function that takes a <code>Bird</code> and you have a function that takes an <code>Chicken</code>, that function would choke on a <code>Duck</code>. But a function that takes an <code>Animal</code> is OK:

标准的animal类库有一个函数可以完成你想要的任务，但是它的参数是`Animal`。在大部分的场景下，如果你说“我需要一个___，我有一个___的子类”，这样是可以的。但是函数的参数都是可逆变的（contravariant），如果你需要一个接受一个`Bird`的函数，并且你有一个接收一个`Chicken`的函数，这个函数会卡在`Duck`上。但是一个接收`Animal`作为参数的函数就没有问题：

<pre class="brush: java; gutter: true">
scala> val getTweet: (Bird => String) = ((a: Animal) => a.sound )
getTweet: Bird => String = <function1>
</pre>

A function's return value type is covariant. If you need a function that returns a <code>Bird</code> but have a function that returns a <code>Chicken</code>, that's great.

函数的返回值是可逆变的。如果你需要一个返回`Bird`的函数，但是你只有一个返回`Chicken`的函数，这样也是可以的。

<pre class="brush: java; gutter: true">
scala> val hatch: (() => Bird) = (() => new Chicken )
hatch: () => Bird = <function0>
</pre>

h2(#bounds). Bounds

Scala allows you to restrict polymorphic variables using _bounds_. These bounds express subtype relationships.

## <a name="bounds">范围(Bounds)</a>

Scala允许你通过 _bounds_ 来限制多态的范围。这些范围表示的是子类的关系。

<pre class="brush: java; gutter: true">
scala> def cacophony[T](things: Seq[T]) = things map (_.sound)
<console>:7: error: value sound is not a member of type parameter T
       def cacophony[T](things: Seq[T]) = things map (_.sound)
                                                        ^

scala> def biophony[T <: Animal](things: Seq[T]) = things map (_.sound)
biophony: [T <: Animal](things: Seq[T])Seq[java.lang.String]

scala> biophony(Seq(new Chicken, new Bird))
res5: Seq[java.lang.String] = List(cluck, call)
</pre>

Lower type bounds are also supported; they come in handy with contravariance and clever covariance. <code>List[+T]</code> is covariant; a list of Birds is a list of Animals. <code>List</code> defines an operator <code>::(elem T)</code> that returns a new <code>List</code> with <code>elem</code> prepended. The new <code>List</code> has the same type as the original:

还可以支持更低的类型范围；它们可以通过逆变(contravariance)和合适的协变（covariance）来实现。`List[+T]`是协变量；一个Bird的列表同时也是一个Animal的列表。`List`定义了一个操作符`::[elem T]`返回一个装载`elem`的列表。这个新的`List`和原来的列表有着相同的类型：

<pre class="brush: java; gutter: true">
scala> val flock = List(new Bird, new Bird)
flock: List[Bird] = List(Bird@7e1ec70e, Bird@169ea8d2)

scala> new Chicken :: flock
res53: List[Bird] = List(Chicken@56fbda05, Bird@7e1ec70e, Bird@169ea8d2)
</pre>

<code>List</code> _also_ defines <code>::[B >: T](x: B)</code> which returns a <code>List[B]</code>. Notice the <code>B >: T</code>. That specifies type <code>B</code> as a superclass of <code>T</code>. That lets us do the right thing when prepending an <code>Animal</code> to a <code>List[Bird]</code>:

`List` _同时_ 也定义了`::[B >: T]`，它返回一个`List[B]`。注意`B >: T`，它表示`T`的父类。这个会在我们把一个`Animal`添加到一个`List[Bird]`的时候提醒我们进行纠正。

<pre class="brush: java; gutter: true">
scala> new Animal :: flock
res59: List[Animal] = List(Animal@11f8d3a8, Bird@7e1ec70e, Bird@169ea8d2)
</pre>

Note that the return type is <code>Animal</code>.

注意返回的类型是`Animal`。

h2(#quantification). Quantification

Sometimes you do not care to be able to name a type variable, for example:

## <a name="quantification">量化（Quantification）</a>

有时候你不需要给一个类型变量以名称，例如

<pre class="brush: java; gutter: true">
scala> def count[A](l: List[A]) = l.size
count: [A](List[A])Int
</pre>

Instead you can use "wildcards":

你可以用“通配符”来替代：

<pre class="brush: java; gutter: true">
scala> def count(l: List[_]) = l.size
count: (List[_])Int
</pre>

This is shorthand for:
这个可以替代：

<pre class="brush: java; gutter: true">
scala> def count(l: List[T forSome { type T }]) = l.size
count: (List[T forSome { type T }])Int
</pre>

Note that quantification can get tricky:

注意量化（quantification）可能会显得比较诡异：

<pre class="brush: java; gutter: true">
scala> def drop1(l: List[_]) = l.tail
drop1: (List[_])List[Any]
</pre>

Suddenly we lost type information! To see what's going on, revert to the heavy-handed syntax:

突然之间我们丢失了类型信息！为了一探究竟，我们来看看最原始的语法：

<pre class="brush: java; gutter: true">
scala> def drop1(l: List[T forSome { type T }]) = l.tail
drop1: (List[T forSome { type T }])List[T forSome { type T }]
</pre>

We can't say anything about T because the type does not allow it.

我们不能说明T的任何信息，因为在这里的类型不允许。

You may also apply bounds to wildcard type variables:

你也可以对通配符类型使用范围来进行限定：


<pre class="brush: java; gutter: true">
scala> def hashcodes(l: Seq[_ <: AnyRef]) = l map (_.hashCode)
hashcodes: (Seq[_ <: AnyRef])Seq[Int]

scala> hashcodes(Seq(1,2,3))
<console>:7: error: type mismatch;
 found   : Int(1)
 required: AnyRef
Note: primitive types are not implicitly converted to AnyRef.
You can safely force boxing by casting x.asInstanceOf[AnyRef].
       hashcodes(Seq(1,2,3))
                     ^

scala> hashcodes(Seq("one", "two", "three"))
res1: Seq[Int] = List(110182, 115276, 110339486)
</pre>

*See Also* <a href="http://www.drmaciver.com/2008/03/existential-types-in-scala/">Existential types in Scala by D. R. MacIver</a>

**参考** D. R. MacIver的<a href="http://www.drmaciver.com/2008/03/existential-types-in-scala/">Scala中的存在类型</a>
