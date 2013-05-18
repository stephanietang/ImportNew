英文原文: [Scala School](http://twitter.github.io/scala_school/advanced-types.html)，翻译：[ImportNew](http://www.importnew.com) - [朱伟杰](http://www.importnew.com/author/zhuweijie)
This lesson covers:

* "View bounds":#viewbounds ("type classes")
* "Other Type Bounds":#otherbounds
* "Higher kinded types & ad-hoc polymorphism":#higher
* "F-bounded polymorphism / recursive types":#fbounded
* "Structural types":#structural
* "Abstract types members":#abstractmem
* "Type erasures & manifests":#manifest
* "Case study: Finagle":#finagle


这个章节的内容包含：
* <a href="#viewbounds">视图边界("类型类")</a>
* <a href="#otherbounds">其他类型边界</a>
* <a href="#higher">高度类型化的类型&临时多态</a>
* <a href="#fbounded">F-bounded多态/可递归类型</a>
* <a href="#structral">结构化的类型</a>
* <a href="#abstract">抽象类型的成员</a>
* <a href="#manifest">类型擦除和Manifest</a>
* <a href="#finagle">实例学习：Finagle</a>


h2(#viewbounds). View bounds ("type classes")

Sometimes you don't need to specify that one type is equal/sub/super another, just that you could fake it with conversions. A view bound specifies a type that can be "viewed as" another. This makes sense for an operation that needs to "read" an object but doesn't modify the object.

*Implicit* functions allow automatic conversion. More precisely, they allow on-demand function application when this can help satisfy type inference. e.g.:

## <a name="viewbounds">视图边界("类型类")</a>

有时候你并不需要指定一个类型是另外一个类型，或者是它的子类或者父类，对于这点你可能会和类型转换搞混淆。一个视图边界定义了一个可以“看作”另一个类型的类型。这个对于需要“读取”一个对象，但是不需要修改它的场景是非常实用的。

*Implicit*函数允许自动进行类型转换。更加确切地说，它会在有助于类型推导的时候允许按需的函数应用，例如：

<pre class="brush: java; gutter: true">
scala> implicit def strToInt(x: String) = x.toInt
strToInt: (x: String)Int

scala> "123"
res0: java.lang.String = 123

scala> val y: Int = "123"
y: Int = 123

scala> math.max("123", 111)
res1: Int = 123
</pre>

view bounds, like type bounds demand such a function exists for the given type.  You specify a type bound with <code><%</code> e.g.,

视图边界，和类型边界相识，也需要一个对于指定类型存在的函数。你可以用一个`%`来表示一个类型边界，例如：

<pre class="brush: java; gutter: true">
scala> class Container[A <% Int] { def addIt(x: A) = 123 + x }
defined class Container
</pre>

This says that *A* has to be "viewable" as *Int*.  Let's try it.

这个表示类型*A*可以被“看作”是类型“Int”。让我们来试试。

<pre class="brush: java; gutter: true">
scala> (new Container[String]).addIt("123")
res11: Int = 246

scala> (new Container[Int]).addIt(123) 
res12: Int = 246

scala> (new Container[Float]).addIt(123.2F)
<console>:8: error: could not find implicit value for evidence parameter of type (Float) => Int
       (new Container[Float]).addIt(123.2)
        ^
</pre>

h2(#otherbounds). Other type bounds


Methods can enforce more complex type bounds via implicit parameters. For example, <code>List</code> supports <code>sum</code> on numeric contents but not on others. Alas, Scala's numeric types don't all share a superclass, so we can't just say <code>T <: Number</code>. Instead, to make this work, Scala's math library <a href="http://www.azavea.com/blogs/labs/2011/06/scalas-numeric-type-class-pt-1/">defines an implicit <code>Numeric[T]</code> for the appropriate types T</a>.  Then in <code>List</code>'s definition uses it:

## <a name="otherbounds">其他类型边界</a>

 方法可以通过implicit参数来使用更加复杂的类型边界。例如，`List`对于数字内容支持sum函数，但是对于其他的则不行。悲剧的是，Scala的数字类型并不都共享同一个父类，因此我们不能使用`T <: Number`来实现。为了达到这样的效果，Scala的math库，为合适的类型定义了一个implicit`Numeric[T]`。然后再在List的定义中使用它：
<pre class="brush: java; gutter: true">
sum[B >: A](implicit num: Numeric[B]): B
</pre>

If you invoke <code>List(1,2).sum()</code>, you don't need to pass a _num_ parameter; it's set implicitly. But if you invoke <code>List("whoop").sum()</code>, it complains that it couldn't set <code>num</code>.

Methods may ask for some kinds of specific "evidence" for a type without setting up strange objects as with <code>Numeric</code>. Instead, you can use of these type-relation operators:

如果你调用`List(1,2).sum()`，你需要传入一个num参数，它会被隐式地进行设置。但是如果你通过`List("whoop").sum()`的方式来调用的话，会无法完成参数的设置。

方法也可能会需要一些特定的“证据”来表明哪些类型可以进行设置，从而避免把奇怪的对象给设置成`Numeric`。并且，在这里你还可以使用之前介绍的类型关系操作符：

|A =:= B|A must be equal to B|
|A <:< B|A must be a subtype of B|
|A <%< B|A must be viewable as B|

<pre class="brush: java; gutter: true">
scala> class Container[A](value: A) { def addIt(implicit evidence: A =:= Int) = 123 + value }
defined class Container

scala> (new Container(123)).addIt
res11: Int = 246

scala> (new Container("123")).addIt
<console>:10: error: could not find implicit value for parameter evidence: =:=[java.lang.String,Int]
</pre>

Similarly, given our previous implicit, we can relax the constraint to viewability:

同样的，对于前面的implicit，我们可以把限制放松到可以进行对应的视图转换即可：

<pre class="brush: java; gutter: true">
scala> class Container[A](value: A) { def addIt(implicit evidence: A <%< Int) = 123 + value }
defined class Container

scala> (new Container("123")).addIt
res15: Int = 246
</pre>

h3. Generic programming with views



In the Scala standard library, views are primarily used to implement generic functions over collections.  For example, the "min" function (on *Seq[]*), uses this technique:

### 通过视图来进行泛型编程

在Scala的标准类库里，视图主要用来实现集合类的泛型函数。例如，“min"函数（在Seq[]里），使用到了这个技术：
<pre class="brush: java; gutter: true">
def min[B >: A](implicit cmp: Ordering[B]): A = {
  if (isEmpty)
    throw new UnsupportedOperationException("empty.min")

  reduceLeft((x, y) => if (cmp.lteq(x, y)) x else y)
}
</pre>

The main advantages of this are:

* Items in the collections aren't required to implement *Ordered*, but *Ordered* uses are still statically type checked.
* You can define your own orderings without any additional library support:

使用这个的主要优点在于：
* 集合中的元素不需要去实现Ordered，但是依然可以使用Ordered进行静态类型检测
* 你可以直接定义你自己的排序，而不需要额外的类库支持

<pre class="brush: java; gutter: true">
scala> List(1,2,3,4).min
res0: Int = 1

scala> List(1,2,3,4).min(new Ordering[Int] { def compare(a: Int, b: Int) = b compare a })
res3: Int = 4
</pre>

As a sidenote, there are views in the standard library that translates *Ordered* into *Ordering* (and vice versa).

旁注，在标准库中有可以把Ordered转换为Ordering（以及反向转换）视图的方法

<pre class="brush: java; gutter: true">
trait LowPriorityOrderingImplicits {
  implicit def ordered[A <: Ordered[A]]: Ordering[A] = new Ordering[A] {
    def compare(x: A, y: A) = x.compare(y)
  }
}
</pre>

h4. Context bounds & implicitly[]


Scala 2.8 introduced a shorthand for threading through & accessing implicit arguments.

## 上下文边界&implicitly[]

Scala 2.8 引入了一个使用和访问implicit参数的快速方法。

<pre class="brush: java; gutter: true">
scala> def foo[A](implicit x: Ordered[A]) {}
foo: [A](implicit x: Ordered[A])Unit

scala> def foo[A : Ordered] {}                        
foo: [A](implicit evidence$1: Ordered[A])Unit
</pre>

Implicit values may be accessed via *implicitly*

Implicit的值可以通过*implicitly*来进行访问。

<pre class="brush: java; gutter: true">
scala> implicitly[Ordering[Int]]
res37: Ordering[Int] = scala.math.Ordering$Int$@3a9291cf
</pre>

Combined, these often result in less code, especially when threading through views.

把这些给组合起来，可以使得代码变得更加简洁，特别是在处理视图的时候。

h2(#higher). Higher-kinded types & ad-hoc polymorphism

Scala can abstract over "higher kinded" types. For example, suppose that you needed to use several types of containers for several types of data. You might define a <code>Container</code> interface that might be implemented by means of several container types: an <code>Option</code>, a <code>List</code>, etc. You want to define an interface for using values in these containers without nailing down the values' type.

## <a name="higher">高度类型化的类型&临时多态</a>
Scala可以抽象出“高度类型化”的类型。例如，假设你需要多个类型的container来处理多个类型的数据。你可能会定义一个`Container`接口，然后它会被多个container类型实现：一个`Option`，一个`List`,等等。你想要定义一个Container接口，并且你需要使用使用其中的值，但是你不想要确定值的实际类型。

This is analagous to function currying. For example, whereas "unary types" have constructors like <code>List[A]</code>, meaning we have to satisfy one "level" of type variables in order to produce a concrete types (just like an uncurried function needs to be supplied by only one argument list to be invoked), a higher-kinded type needs more.

这个和currying函数的场景非常相似。例如，鉴于“一元的类型”有着类似`List[A]`的构造器，这就意味着我们需要满足一“级”类型变量的条件，这样才能够产生具体的类型（就像一个非currying的函数只能有一个参数列表，它才能够被调用），一个高度类型化的类型需要更多的信息。


<pre class="brush: java; gutter: true">
scala> trait Container[M[_]] { def put[A](x: A): M[A]; def get[A](m: M[A]): A }

scala> val container = new Container[List] { def put[A](x: A) = List(x); def get[A](m: List[A]) = m.head }
container: java.lang.Object with Container[List] = $anon$1@7c8e3f75

scala> container.put("hey")
res24: List[java.lang.String] = List(hey)

scala> container.put(123)
res25: List[Int] = List(123)
</pre>

Note that *Container* is polymorphic in a parameterized type ("container type").

If we combine using containers with implicits, we get "ad-hoc" polymorphism: the ability to write generic functions over containers.

如果我们结合implicit和container接口，我们就能够得到“即时”多态（"ad-hoc" polymorphism）：这是一种可以在container上编写泛型函数的功能。

<pre class="brush: java; gutter: true">
scala> trait Container[M[_]] { def put[A](x: A): M[A]; def get[A](m: M[A]): A }

scala> implicit val listContainer = new Container[List] { def put[A](x: A) = List(x); def get[A](m: List[A]) = m.head }

scala> implicit val optionContainer = new Container[Some] { def put[A](x: A) = Some(x); def get[A](m: Some[A]) = m.get }

scala> def tupleize[M[_]: Container, A, B](fst: M[A], snd: M[B]) = {
     | val c = implicitly[Container[M]]                             
     | c.put(c.get(fst), c.get(snd))
     | }
tupleize: [M[_],A,B](fst: M[A],snd: M[B])(implicit evidence$1: Container[M])M[(A, B)]

scala> tupleize(Some(1), Some(2))
res33: Some[(Int, Int)] = Some((1,2))

scala> tupleize(List(1), List(2))
res34: List[(Int, Int)] = List((1,2))
</pre>

h2(#fbounded). F-bounded polymorphism

Often it's necessary to access a concrete subclass in a (generic) trait. For example, imagine you had some trait that is generic, but can be compared to a particular subclass of that trait.

## <a name="fbounded">F-bounded多态</a>

很多时候，我们需要在一个（泛型的）traint里访问一个具体的子类。例如，假设你有一些泛型的trait，但是需要和trait的一个特定的子类进行比较。

<pre class="brush: java; gutter: true">
trait Container extends Ordered[Container]
</pre>

However, this now necessitates the compare method

现在，在这里需要一个compare方法。

<pre class="brush: java; gutter: true">
def compare(that: Container): Int
</pre>

And so we cannot access the concrete subtype, eg.:

这样的话，我们就不能访问具体的子类型了，例如:


<pre class="brush: java; gutter: true">
class MyContainer extends Container {
  def compare(that: MyContainer): Int
}
</pre>

fails to compile, since we are specifying Ordered for *Container*, not the particular subtype.

To reconcile this, we instead use F-bounded polymorphism.

这段代码会编译失败，因为我们给*Container*指定的是Order，而不是具体的子类型。
我们可以使用F-bounded多态来修复它。

<pre class="brush: java; gutter: true">
trait Container[A <: Container[A]] extends Ordered[A]
</pre>

Strange type!  But note now how Ordered is parameterized on *A*, which itself is *Container[A]*

很奇怪的类型！但是请注意Ordered在*A*上是如何指定类型的，A本身也是一个*Container[A]*。

So, now 

现在

<pre class="brush: java; gutter: true">
class MyContainer extends Container[MyContainer] { 
  def compare(that: MyContainer) = 0 
}
</pre>

They are now ordered:

现在它们都是有序的：

<pre class="brush: java; gutter: true">
scala> List(new MyContainer, new MyContainer, new MyContainer)
res3: List[MyContainer] = List(MyContainer@30f02a6d, MyContainer@67717334, MyContainer@49428ffa)

scala> List(new MyContainer, new MyContainer, new MyContainer).min
res4: MyContainer = MyContainer@33dfeb30
</pre>

Given that they are all subtypes of *Container[_]*, we can define another subclass & create a mixed list of *Container[_]*:

考虑到它们都是*Container[_]*的子类，我们可以定义另一个子类，并且创建一个*Container[_]*的混合列表。

<pre class="brush: java; gutter: true">
scala> class YourContainer extends Container[YourContainer] { def compare(that: YourContainer) = 0 }
defined class YourContainer

scala> List(new MyContainer, new MyContainer, new MyContainer, new YourContainer)                   
res2: List[Container[_ >: YourContainer with MyContainer <: Container[_ >: YourContainer with MyContainer <: ScalaObject]]] 
  = List(MyContainer@3be5d207, MyContainer@6d3fe849, MyContainer@7eab48a7, YourContainer@1f2f0ce9)
</pre>

Note how the resulting type is now lower-bound by *YourContainer with MyContainer*. This is the work of the type inferencer. Interestingly- this type doesn't even need to make sense, it only provides a logical greatest lower bound for the unified type of the list. What happens if we try to use *Ordered* now?

注意最终的类型是如何被*YourContainer 和 MyContainer*进行限制的。这是类型推导器的工作。有趣的是——这个类型并不需要有实际的意义，它只是为List的所有类型提供了一个逻辑上的最小边界。那么，如果我们使用*Ordered*会怎么样呢？

<pre class="brush: java; gutter: true">
(new MyContainer, new MyContainer, new MyContainer, new YourContainer).min
<console>:9: error: could not find implicit value for parameter cmp:
  Ordering[Container[_ >: YourContainer with MyContainer <: Container[_ >: YourContainer with MyContainer <: ScalaObject]]]
</pre>

No *Ordered[]* exists for the unified type. Too bad.

对于这个统一的类型没有*Ordered[]* 存在。这个太不给力了。

h2(#structural). Structural types

Scala has support for *structural types* -- type requirements are expressed by interface _structure_ instead of a concrete type.

## <a name="structural">结构化的类型</a>

Scala 支持*结构化的类型* -- 对于这个类型的需求一般用接口结构（iterface structure）来表示,而非使用具体的某个类型。

<pre class="brush: java; gutter: true">
scala> def foo(x: { def get: Int }) = 123 + x.get
foo: (x: AnyRef{def get: Int})Int

scala> foo(new { def get = 10 })                 
res0: Int = 133
</pre>

This can be quite nice in many situations, but the implementation uses reflection, so be performance-aware!

这个特性在很多场景都特别有用，但是具体的实现用的是反射，所以需要注意性能问题。

h2(#abstractmem). Abstract type members

In a trait, you can leave type members abstract.

## <a name="abstractmem">抽象类型的成员</a>

在一个trait里，你可以使用抽象类型的成员。

<pre class="brush: java; gutter: true">
scala> trait Foo { type A; val x: A; def getX: A = x }
defined trait Foo

scala> (new Foo { type A = Int; val x = 123 }).getX   
res3: Int = 123

scala> (new Foo { type A = String; val x = "hey" }).getX
res4: java.lang.String = hey
</pre>

This is often a useful trick when doing dependency injection, etc.

You can refer to an abstract type variable using the hash-operator:

在处理依赖注入等场景时，这是一个很有用的手段。
你可以通过hash操作来引用一个抽象的类型变量：

<pre class="brush: java; gutter: true">
scala> trait Foo[M[_]] { type t[A] = M[A] }
defined trait Foo

scala> val x: Foo[List]#t[Int] = List(1)
x: List[Int] = List(1)
</pre>


h2(#manifest). Type erasures & manifests

As we know, type information is lost at compile time due to _erasure_. Scala features *Manifests*, allowing us to selectively recover type information. Manifests are provided as an implicit value, generated by the compiler as needed.

## <a name="manifest">类型擦除和Manifest</a>

我们都知道，由于_擦除_的原因，类型信息在编译期都丢失了。Scala提供了*Manifests*，它可以让我们有选择地进行类型恢复。Manifest是一个implicit值，它是由编译器按需生成的。

<pre class="brush: java; gutter: true">
scala> class MakeFoo[A](implicit manifest: Manifest[A]) { def make: A = manifest.erasure.newInstance.asInstanceOf[A] }

scala> (new MakeFoo[String]).make
res10: String = ""
</pre>

h2(#finagle). Case study: Finagle

See: https://github.com/twitter/finagle

## <a name="finagle">示例学习：Finagle</a>

参考：https://github.com/twitter/finagle

<pre class="brush: java; gutter: true">
trait Service[-Req, +Rep] extends (Req => Future[Rep])

trait Filter[-ReqIn, +RepOut, +ReqOut, -RepIn]
  extends ((ReqIn, Service[ReqOut, RepIn]) => Future[RepOut])
{
  def andThen[Req2, Rep2](next: Filter[ReqOut, RepIn, Req2, Rep2]) =
    new Filter[ReqIn, RepOut, Req2, Rep2] {
      def apply(request: ReqIn, service: Service[Req2, Rep2]) = {
        Filter.this.apply(request, new Service[ReqOut, RepIn] {
          def apply(request: ReqOut): Future[RepIn] = next(request, service)
          override def release() = service.release()
          override def isAvailable = service.isAvailable
        })
      }
    }
    
  def andThen(service: Service[ReqOut, RepIn]) = new Service[ReqIn, RepOut] {
    private[this] val refcounted = new RefcountedService(service)

    def apply(request: ReqIn) = Filter.this.apply(request, refcounted)
    override def release() = refcounted.release()
    override def isAvailable = refcounted.isAvailable
  }    
}
</pre>

A service may authenticate requests with a filter.

一个服务可以通过一个filter来验证请求。

<pre class="brush: java; gutter: true">
trait RequestWithCredentials extends Request {
  def credentials: Credentials
}

class CredentialsFilter(credentialsParser: CredentialsParser)
  extends Filter[Request, Response, RequestWithCredentials, Response]
{
  def apply(request: Request, service: Service[RequestWithCredentials, Response]): Future[Response] = {
    val requestWithCredentials = new RequestWrapper with RequestWithCredentials {
      val underlying = request
      val credentials = credentialsParser(request) getOrElse NullCredentials
    }

    service(requestWithCredentials)
  }
}
</pre>

Note how the underlying service requires an authenticated request, and that this is statically verified. Filters can thus be thought of as service transformers.

注意底层的服务对于请求验证的实现，它是静态地实现的。Filter也可以被认为是服务的转换器。

Many filters can be composed together:

现在，多个filter可以组合一起使用。

<pre class="brush: java; gutter: true">
val upFilter =
  logTransaction     andThen
  handleExceptions   andThen
  extractCredentials andThen
  homeUser           andThen
  authenticate       andThen
  route
</pre>

Type safely!
安全地使用类型！