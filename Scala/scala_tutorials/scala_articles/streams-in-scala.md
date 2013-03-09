
英文原文: [Looking at Streams in Scala](http://blog.danielwellman.com/2008/03/streams-in-scal.html)，翻译：[ImportNew](http://www.importnew.com) - [朱伟杰](http://www.importnew.com/author/zhuweijie)

# Looking at Streams in Scala

Scala offers  a few powerful abstractions for working with sequences of values. Functional programming style encourages moving from mutable variables and state changes to declarative lists of all known states. Higher order functions like map, flatMap, and filter can very succinctly represent complex requirements.

# Scala 中的 Stream

对于一组值序列的处理，Scala提供了一些功能强大的抽象。函数式编程鼓励使用包含所有状态的声明式列表，而避免使用变量和可变状态。高阶函数，例如map，flatMap和filter可以很简洁地实现复杂的需求。

For example, suppose you were writing a search engine and a user asked for the 10th document about Scala in a database of 10,0000 HTML documents. In a functional style, you might declare the list of all documents, compute the score for each one, and then pick the 10th item. This might look like something like this:

例如，假如你在编写一个搜索引擎，并且你的用户需要查找在一个包含10,0000个HTML文档的关于Scala的第10个文档。在函数式的语言里，你可能会声明一个包含所有文档的列表，然后分别计算它们的分数，最后获取第10份文档。可能和下面的代码看起来相似：

<pre class="brush: java; gutter: true">
documents.filter(isAboutScala)(9)
</pre>

The problem is that if you've got 10,000 big HTML documents, you'll need to first ask each one of 10,000 if they're about Scala to build the filtered list -- only to take the 10th document. Clearly this overhead won't scale well!

这里的问题在于，如果HTML的文档数量有10,000这么多的话，你首先需要一个一个地处理这10,000个文档，来看看它是否是关于Scala的，然后来构建这个过滤了的列表——这仅仅只是为了获取第10个文档。显然这种开销的扩展性不是很好！

One solution is to use Scala's Stream construct; which is like a List which lazily evaluates the next item only when needed. Using a Stream would only evaluate enough documents to find the 10th match - instead of first evaluating the 10,000 documents.

一种解决方案是使用Scala的Stream结构，它和列表相似，只不过它会延迟计算下一个元素，仅当需要的时候才会去计算。使用Stream只会进行必要的处理来查找第10份文档 - 而不是对所有的10,000份文档同时进行计算。

For a full introduction to Streams, see the chapter "Computing with Streams" in "Scala By Example".

要查看Stream的详细介绍，你可以参考[《Scala By Example》](http://www.scala-lang.org/docu/files/ScalaByExample.pdf)里的“Computing with Streams”。

So how do Streams work? How is the lazy evaluation defined? Looking in to the implementation of Stream reveals many nice things about the Scala language and some of the philosophies adopted for building libraries.

那么，Stream究竟是如何起作用的呢？所谓的延迟处理又是怎么定义的呢？深入了解Stream的实现细节，可以发现Scala语言以及构建类库所采纳的一些很好的理念。

The syntax for constructing a Stream requires using Stream.cons and Stream.empty:
构造一个Stream需要使用`Stream.cons`和`Stream.empty`

<pre class="brush: java; gutter: true">
Stream.cons(3, Stream.cons(4, Stream.empty))
</pre>

Upon initial inspection, it looks like cons is a method call on the Stream object. It turns out cons is actually a nested object in the Stream object.

乍一看，`cons`似乎是Stream对象的一个方法。但是，事实上它是Stream对象的一个内部对象。

 *Stream.cons is an object, not a method on the Stream object* 

 *`Stream.cons`是一个对象，而不是Stream对象的一个方法*

Looking at Stream.scala in the Scala libraries source shows this structure:

查看Scala类库里的Stream.scala，你会发现下面的结果：

<pre class="brush: java; gutter: true">
object Stream {
  object cons {
    // ... definition of Stream.cons object
  }
  // ... rest of Stream
}
</pre>

This nesting of objects is also possible in Java. However, Scala's use of the apply() function (which we'll look at below) means that a method invocation on the nested cons object looks like a method call on the Stream object.

**Syntactic sugar and the apply() method**

这种嵌套对象的方式在Java里也是可行的。不过，Scala里使用`apply()`方法（这个我们后面会讨论）意味着嵌套类的cons的方法调用看起来是Stream对象上的方法调用。

 **语法糖和apply()方法**

Recall that a Scala object may define a method apply() like so:

回想一下，Scala对象可能会这样定义apply()方法：

<pre class="brush: java; gutter: true">
object List {
  def apply[A](xs: A*): List[A] = xs.toList
  // ...
}
</pre>

This means you can call the List object to create the list of items 1, 2, and 3 with this syntax:

这就意味着，你可以通过下面的语法来调用List对象创建一个包含1，2和3的列表：

<pre class="brush: java; gutter: true">
val usingApply = List.apply(1, 2, 3)
</pre>

or using Scala syntactic sugar:

或者使用Scala的语法糖：

<pre class="brush: java; gutter: true">
val usingSugar = List(1, 2, 3)
</pre>

For me, the combination of apply()'s syntactic sugar and nested singleton objects begin to describe the 'scalable language' idea that the Scala literature talks about so frequently. What looks like a method call is really a call to a nested object's apply method, but the syntax reads identically. The end user doesn't need to know if you're talking to a nested singleton object in a library class; it is no different than a standard function call.

对于我而言，apply()的语法糖和嵌套的单例内部对象的组合诠释了Scala文化一直在描述的'可扩展的语言（scalable language）'的观点。一个方法调用实际上是对一个内部对象的方法调用，不过语法上却没有区别。终端用户不需要了解你是否在操作类库中的一个类的嵌套内部对象；因为它和标准的函数调用没有区别。

But how does Stream's lazy evaluation work?

**Call by name evaluation**

Let's look again at Stream.cons' apply method signature:

不过，Stream的延迟处理是怎么实现的呢？

**Call by name 求值**

<pre class="brush: java; gutter: true">
def apply[A](head : A, tail : => Stream[A]) : Stream[A]
</pre>

Note the => operator ; this indicates call-by-name evaluation. This is a form of lazy argument evaluation; the language will only evaluate tail if it needs to.

注意`=>`操作符；它表示一个call-by-name求值。这是参数延迟求值的一种形式；tail参数只会在需要使用的时候才会被求值。

Thus, Scala allows you to construct an expensive list of items worry-free; the runtime will evaluate the Stream's items one-by-one until the specified conditions are met -- and will go no further.

这样，Scala就可以允许你毫无顾忌地创建一个开销很大的列表；运行时环境会一个一个地对Stream中的元素进行求值，知道指定的条件达到——并且后面的元素不会被求值。


To see more examples of Stream in action, check out this article about solving Project Euler problems in Scala.

想了解更多关于Scala的Stream的话，你可以参考[这篇文章](http://scala-blogs.org/2007/12/project-euler-fun-in-scala.html)，它介绍了如何使用Scala解决Euler项目里的问题。