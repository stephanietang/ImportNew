英文原文: [Scala School](http://twitter.github.io/scala_school/coll2.html)，翻译：[ImportNew](http://www.importnew.com) - [朱伟杰](http://www.importnew.com/author/zhuweijie)


Scala provides a nice set of collection implementations. It also provides some abstractions for collection types. This allows you to write code that can work with a collection of <code>Foo</code>s without worrying whether that collection is a <code>List</code>, <code>Set</code>, or what-have-you.

Scala提供了一系列的集合类的实现。同时，它对于集合类型也进行了一些抽象。这就使得你可以操作一个集合的`Foo`对象，而不用去关心这个集合是一个`List`，`Set`还是其他的什么。

"This page":http://www.decodified.com/scala/collections-api.xml offers a great way to follow the default implementations and links to all the scaladoc.

<a href="http://www.decodified.com/scala/collections-api.xml">这个网页</a>提供了一个很好的方式来理解scala里的集合的默认实现，并且都链接到了相应的scaladoc。



* "Basics":#basics Collection types you'll use all the time
* "Hierarchy":#hierarchy Collection abstractions
* "Methods":#methods
* "Mutable":#mutable
* "Java collections":#java just work

* <a href="#Basic">基本集合类</a> 常用的集合类型
* <a href="#Hierarchy">继承</a> 集合类的抽象关系
* <a href="#Method"> 方法</a>
* <a href="#Mutable">可变性</a>
* <a href="#Java collections">Java 集合类</a>也可直接使用

h2(#basics). The basics

## <a name="Basic">基本集合类</a>

h3. List

The standard linked list.

### List

标准的linked list。

<pre>
scala> List(1, 2, 3)
res0: List[Int] = List(1, 2, 3)
</pre>

You can cons them up as you would expect in a functional language.

你也可以像在函数式语言里一样把它们串接起来。

<pre>
scala> 1 :: 2 :: 3 :: Nil
res1: List[Int] = List(1, 2, 3)
</pre>

*See also* "API doc":http://www.scala-lang.org/api/current/scala/collection/immutable/List.html

**参考** <a href="http://www.scala-lang.org/api/current/scala/collection/immutable/List.html">API文档</a>


h3. Set

Sets have no duplicates

### Set

Set里不包含重复元素

<pre>
scala> Set(1, 1, 2)
res2: scala.collection.immutable.Set[Int] = Set(1, 2)
</pre>

*See also* "API doc":http://www.scala-lang.org/api/current/scala/collection/immutable/Set.html

**参考** <a href="http://www.scala-lang.org/api/current/scala/collection/immutable/Set.html">API文档</a>

h3. Seq

Sequences have a defined order.

### Seq

Sequence都有一个预定义的顺序。

<pre>
scala> Seq(1, 1, 2)
res3: Seq[Int] = List(1, 1, 2)
</pre>

(Notice that returned a List. <code>Seq</code> is a trait; List is a lovely implementation of Seq. There's a factory object <code>Seq</code> which, as you see here, creates Lists.)

(注意返回的结果是一个List。`Seq`是一个trait；List是它的一个实现类。`Seq`对象是一个工厂对象，正如你所看到的，它会创建一个List。)

*See also* API doc":http://www.scala-lang.org/api/current/scala/collection/Seq.html

**参考** <a href="http://www.scala-lang.org/api/current/scala/collection/Seq.html">API文档</a>

h3. Map

Maps are key value containers.

### Map

Map是保存键-值对的容器。

<pre>
scala> Map('a' -> 1, 'b' -> 2)
res4: scala.collection.immutable.Map[Char,Int] = Map((a,1), (b,2))
</pre>

*See also* "API doc":http://www.scala-lang.org/api/current/scala/collection/immutable/Map.html

**参考** <a href="http://www.scala-lang.org/api/current/scala/collection/immutable/Map.html">API文档</a>

h2(#hierarchy). The Hierarchy

These are all traits, both the mutable and immutable packages have implementations of these as well as specialized implementations.

## <a href="hierarchy">继承关系</a>

上面的这些都是trait，在mutable和immutable包里都有对应的实现，同时也有其他特殊用途的实现。

h3. Traversable

All collections can be traversed.  This trait defines standard function combinators. These combinators are written in terms of @foreach@, which collections must implement.

*See Also* "API doc":#http://www.scala-lang.org/api/current/scala/collection/Traversable.html


### Traversable

所有的集合都可遍历的。这个trait定义了标准函数组合器。这些组合器都是通过foreach这个方法来实现的，并且所有的集合都实现了这个方法。

**参考** <a href="http://www.scala-lang.org/api/current/scala/collection/Traversable.html">API文档</a>



h3. Iterable

Has an @iterator()@ method to give you an Iterator over the elements.

*See Also* "API doc":http://www.scala-lang.org/api/current/scala/collection/Iterable.html

### Iterable

这个trait有一个`iterator()`方法，它返回一个可以遍历所有元素的Iterator。
**参考** <a href="http://www.scala-lang.org/api/current/scala/collection/Iterable.html">API文档</a>


h3. Seq

Sequence of items with ordering.

*See Also* "API doc":http://www.scala-lang.org/api/current/scala/collection/Seq.html

### Seq

一组有序的元素。

**参考** <a href="http://www.scala-lang.org/api/current/scala/collection/Seq.html">API文档</a>


h3. Set

A collection of items with no duplicates.

*See Also* "API doc":http://www.scala-lang.org/api/current/scala/collection/immutable/Set.html

###　Set

一组没有重复元素的集合。

**参考** <a href="http://www.scala-lang.org/api/current/scala/collection/immutable/Set.html">API文档</a>



h3. Map

Key Value Pairs.

*See Also* "API doc":http://www.scala-lang.org/api/current/scala/collection/immutable/Map.html


### Map
键-值对。

**参考** <a href="http://www.scala-lang.org/api/current/scala/collection/immutable/Map.html">API文档</a>

h2(#methods). The methods

h3. Traversable

All of these methods below are available all the way down.  The argument and return types types won't always look the same as subclasses are free to override them.

## <a name="methods">方法</a>

### Traversable

上面所有的方法在子类中都是可用的。参数和返回值可能会改变，因为子类可以覆盖它们。


<pre>
def head : A
def tail : Traversable[A]
</pre>

Here are where the Functional Combinators are defined.

这里我们可以定义函数组合器。

<code>
def map [B] (f: (A) => B) : CC[B]
</code>

returns a collection with every element transformed by @f@

上面的代码返回一个集合，里面的每一个元素都通过函数`f`进行了转换。

<code>
def foreach[U](f: Elem => U): Unit
</code>

executes @f@ over each element in a collection.

对一个集合里的每一个元素都执行函数`f`。

<code>
def find (p: (A) => Boolean) : Option[A]
</code>

returns the first element that matches the predicate function

它返回第一个符合断言函数的元素。

<code>
def filter (p: (A) => Boolean) : Traversable[A]
</code>

returns a collection with all elements matching the predicate function

这个返回一个包含所有符合断言函数的元素的集合。

Partitioning:

切分：

<code>
def partition (p: (A) ⇒ Boolean) : (Traversable[A], Traversable[A])
</code>

Splits a collection into two halves based on a predicate function

通过一个断言函数把一个集合拆成两份

<code>
def groupBy [K] (f: (A) => K) : Map[K, Traversable[A]]
</code>

Conversion:

Interestingly, you can convert one collection type to another.

转换：

有趣的是，集合之间可以相互进行转换。

<pre>
def toArray : Array[A]
def toArray [B >: A] (implicit arg0: ClassManifest[B]) : Array[B]
def toBuffer [B >: A] : Buffer[B]
def toIndexedSeq [B >: A] : IndexedSeq[B]
def toIterable : Iterable[A]
def toIterator : Iterator[A]
def toList : List[A]
def toMap [T, U] (implicit ev: <:<[A, (T, U)]) : Map[T, U]
def toSeq : Seq[A]
def toSet [B >: A] : Set[B]
def toStream : Stream[A]
def toString () : String
def toTraversable : Traversable[A]
</pre>

Let's convert a Map to an Array. You get an Array of the Key Value pairs.

我们可以把一个Map转换成一个数组，然后得到一个键值对数组。

<pre>
scala> Map(1 -> 2).toArray
res41: Array[(Int, Int)] = Array((1,2))
</pre>

h3. Iterable

Adds access to an iterator.



### Iterable

这个接口给你提供了一个迭代器iterator。

<pre>
  def iterator: Iterator[A]
</pre>

What does an Iterator give you?

Iterator提供了什么操作呢？

<pre>
def hasNext(): Boolean
def next(): A
</pre>

This is very Java-esque.  You often won't see iterators used in Scala, you are much more likely to see the functional combinators or a for-comprehension used.

这个是非常Java-式的。在Scala中，你很少会见到使用iterator的地方，大部分的时候你看到的都是函数组合器以及for-comprehension。

h3. Set

### Set

<pre>
  def contains(key: A): Boolean
  def +(elem: A): Set[A]
  def -(elem: A): Set[A]
</pre>

h3. Map

Sequence of key and value pairs with lookup by key.

Pass a List of Pairs into apply() like so

### Map

一序列键值对，提供按key进行查询的操作。

可以通过给apply()方法传入一个键值对的列表来创建Map：

<pre>
scala> Map("a" -> 1, "b" -> 2)
res0: scala.collection.immutable.Map[java.lang.String,Int] = Map((a,1), (b,2))
</pre>

Or also like:

或者按照下面的方式：

<pre>
scala> Map(("a", 2), ("b", 2))
res0: scala.collection.immutable.Map[java.lang.String,Int] = Map((a,2), (b,2))
</pre>

h6. Digression

What is <code>-></code>? That isn't special syntax, it's a method that returns a Tuple.

###### 题外话

`->`是什么呢？这是一个特殊的语法，它是一个返回值为元组的方法。

<pre>
scala> "a" -> 2

res0: (java.lang.String, Int) = (a,2)
</pre>

Remember, that is just sugar for

记住，它只是下面这种形式的语法糖：
<pre>
scala> "a".->(2)

res1: (java.lang.String, Int) = (a,2)
</pre>

You can also build one up via <code>++</code>

你也可以通过`++`来构造Map

<pre>
scala> Map.empty ++ List(("a", 1), ("b", 2), ("c", 3))
res0: scala.collection.immutable.Map[java.lang.String,Int] = Map((a,1), (b,2), (c,3))
</pre>

h3. Commonly-used subclasses

*HashSet and HashMap* Quick lookup, the most commonly used forms of these collections. "HashSet API":http://www.scala-lang.org/api/current/scala/collection/immutable/HashSet.html, "HashMap API":http://www.scala-lang.org/api/current/scala/collection/immutable/HashMap.html

*TreeMap* A subclass of SortedMap, it gives you ordered access. "TreeMap API":http://www.scala-lang.org/api/current/scala/collection/immutable/TreeMap.html

*Vector* Fast random selection and fast updates. "Vector API":http://www.scala-lang.org/api/current/scala/collection/immutable/Vector.html

### 常用的子类

**HashSet 和 HashMap** 支持快速查找的最常用的集合类。<a href="http://www.scala-lang.org/api/current/scala/collection/immutable/HashSet.html">HashSet API</a>, <a href="http://www.scala-lang.org/api/current/scala/collection/immutable/HashMap.html">HashMap API</a>


**TreeMap** SortedMap的子类，它提供有序的访问方式。 <a href="http://www.scala-lang.org/api/current/scala/collection/immutable/TreeMap.html">TreeMap API</a>

*Vector* 支持快速查找和更新 <a href="http://www.scala-lang.org/api/current/scala/collection/immutable/Vector.html">Vector API</a>


<pre>
scala> IndexedSeq(1, 2, 3)
res0: IndexedSeq[Int] = Vector(1, 2, 3)
</pre>

**Range** Ordered sequence of Ints that are spaced apart.  You will often see this used where a counting for-loop was used before. "Range API":http://www.scala-lang.org/api/current/scala/collection/immutable/Range.html

**Range** 有序的通过空格分隔的Int序列。  在之前很多用来计数的for循环经常使用它。<a href="http://www.scala-lang.org/api/current/scala/collection/immutable/Range.html">Range API</a>

<pre>
scala> for (i <- 1 to 3) { println(i) }
1
2
3
</pre>

Ranges have the standard functional combinators available to them.

Range也支持标准的函数组合器。
<pre>
scala> (1 to 3).map { i => i }
res0: scala.collection.immutable.IndexedSeq[Int] = Vector(1, 2, 3)
</pre>

h3. Defaults

Using apply methods on the traits will give you an instance of the default implementation, For instance, Iterable(1, 2) returns a List as its default implementation.

### 默认值

在trait上使用apply方法会得到该trait的一个默认实现，例如，Iterable(1,2)返回一个List作为它的默认实现。

<pre>
scala> Iterable(1, 2)

res0: Iterable[Int] = List(1, 2)
</pre>

Same with Seq, as we saw earlier

Seq也是这样的，我们之前见过

<pre>
scala> Seq(1, 2)
res3: Seq[Int] = List(1, 2)

scala> Iterable(1, 2)
res1: Iterable[Int] = List(1, 2)

scala> Sequence(1, 2)
warning: there were deprecation warnings; re-run with -deprecation for details
res2: Seq[Int] = List(1, 2)
</pre>

Set

<pre>
scala> Set(1, 2)
res31: scala.collection.immutable.Set[Int] = Set(1, 2)
</pre>

h3. Some descriptive traits

*IndexedSeq* fast random-access of elements and a fast length operation. "API doc":http://www.scala-lang.org/api/current/scala/collection/IndexedSeq.html

*LinearSeq* fast access only to the first element via head, but also has a fast tail operation. "API doc":http://www.scala-lang.org/api/current/scala/collection/LinearSeq.html

### 一些描述性的trait


**IndexedSeq** 支持对元素进行快速随机访问以及快速的length操作 <a href="http://www.scala-lang.org/api/current/scala/collection/IndexedSeq.html">API doc</a>

**LinearSeq** 只对第一个和最后一个元素支持快速访问。 <a href="http://www.scala-lang.org/api/current/scala/collection/LinearSeq.html">API doc</a>

h4. Mutable vs. Immutable

h4. Mutable vs. Immutable

#### 可变性 vs. 不可变性 

immutable

Pros

* Can't change in multiple threads

Con

* Can't change at all

不可变性

优点

* 在多线程场景下不会被改变

缺点

* 没有办法进行修改


Scala allows us to be pragmatic, it encourages immutability but does not penalize us for needing mutability.  This is very similar to var vs. val.  We always start with val and move back to var when required.

We favor starting with the immutable versions of collections but switching to the mutable ones if performance dictates.  Using immutable collections means you won't accidentally change things in multiple threads.

Scala允许我们根据实际需求决定可变性，它鼓励我们只用不可变量，但是也不禁止我们使用变量。这个和var与val的场景非常相似。我们一开始都是使用val，当实际需要的时候会改成var。

h2(#mutable). Mutable

All of the above classes we've discussed were immutable.  Let's discuss the commonly used mutable collections.

*HashMap* defines @getOrElseUpdate@, @+=@ "HashMap API":http://www.scala-lang.org/api/current/scala/collection/mutable/HashMap.html

## <a name="mutable">可变集合</a>

我们前面讨论的所有的类都是不可变的。我们现在来讨论一下常用的可变集合。

**HashMap** 定义了`getOrElseUpdate`, `+=` "HashMap API":http://www.scala-lang.org/api/current/scala/collection/mutable/HashMap.html


<pre>
scala> val numbers = collection.mutable.Map(1 -> 2)
numbers: scala.collection.mutable.Map[Int,Int] = Map((1,2))

scala> numbers.get(1)
res0: Option[Int] = Some(2)

scala> numbers.getOrElseUpdate(2, 3)
res54: Int = 3

scala> numbers
res55: scala.collection.mutable.Map[Int,Int] = Map((2,3), (1,2))

scala> numbers += (4 -> 1)
res56: numbers.type = Map((2,3), (4,1), (1,2))
</pre>

*ListBuffer and ArrayBuffer* Defines @+=@ "ListBuffer API":http://www.scala-lang.org/api/current/scala/collection/mutable/ListBuffer.html, "ArrayBuffer API":http://www.scala-lang.org/api/current/scala/collection/mutable/ArrayBuffer.html

*LinkedList and DoubleLinkedList* "LinkedList API":http://www.scala-lang.org/api/current/scala/collection/mutable/LinkedList.html, "DoubleLinkedList API":http://www.scala-lang.org/api/current/scala/collection/mutable/DoubleLinkedList.html

*PriorityQueue* "API doc":http://www.scala-lang.org/api/current/scala/collection/mutable/PriorityQueue.html

*Stack and ArrayStack* "Stack API":http://www.scala-lang.org/api/current/scala/collection/mutable/Stack.html, "ArrayStack API":http://www.scala-lang.org/api/current/scala/collection/mutable/ArrayStack.html

*StringBuilder* Interestingly, StringBuilder is a collection. "API doc":http://www.scala-lang.org/api/current/scala/collection/mutable/StringBuilder.html

**ListBuffer和ArrayBuffer** 定义了`+=`操作符 <a href="http://www.scala-lang.org/api/current/scala/collection/mutable/ListBuffer.html">ListBuffer API</a>, [ArrayBuffer API](http://www.scala-lang.org/api/current/scala/collection/mutable/ArrayBuffer.html)

**LinkedList和DoubleLinkedList** [LinkedList API](http://www.scala-lang.org/api/current/scala/collection/mutable/LinkedList.html), [DoubleLinkedList API](http://www.scala-lang.org/api/current/scala/collection/mutable/DoubleLinkedList.html)

**PriorityQueue** [API doc](http://www.scala-lang.org/api/current/scala/collection/mutable/PriorityQueue.html)

**Stack和ArrayStack** [Stack API](http://www.scala-lang.org/api/current/scala/collection/mutable/Stack.html), [ArrayStack API](http://www.scala-lang.org/api/current/scala/collection/mutable/ArrayStack.html)

**StringBuilder** 有趣的是StringBuilder是竟然一个集合 [API doc](http://www.scala-lang.org/api/current/scala/collection/mutable/StringBuilder.html)


h2(#java). Life with Java

You can easily move between Java and Scala collection types using conversions that are available in the <a href="http://www.scala-lang.org/api/current/index.html#scala.collection.JavaConverters$">JavaConverters package</a>. It decorates commonly-used Java collections with <code>asScala</code> methods and Scala collections with <code>asJava</code> methods.

## <a name="java"> 和Java进行交互</a>

你可以通过 <a href="http://www.scala-lang.org/api/current/index.html#scala.collection.JavaConverters$">JavaConverters 包</a>在Java和Scala的集合类之间进行转换. 它给常用的Java集合提供了`asScala`方法，同时给常用的Scala集合提供了`asJava`方法。


<pre>
   import scala.collection.JavaConverters._
   val sl = new scala.collection.mutable.ListBuffer[Int]
   val jl : java.util.List[Int] = sl.asJava
   val sl2 : scala.collection.mutable.Buffer[Int] = jl.asScala
   assert(sl eq sl2)
</pre>

Two-way conversions:

双向转换：

<pre>
scala.collection.Iterable <=> java.lang.Iterable
scala.collection.Iterable <=> java.util.Collection
scala.collection.Iterator <=> java.util.{ Iterator, Enumeration }
scala.collection.mutable.Buffer <=> java.util.List
scala.collection.mutable.Set <=> java.util.Set
scala.collection.mutable.Map <=> java.util.{ Map, Dictionary }
scala.collection.mutable.ConcurrentMap <=> java.util.concurrent.ConcurrentMap
</pre>

In addition, the following one way conversions are provided:

另外，下面提供了一些单向的转换方法：

<pre>
scala.collection.Seq => java.util.List
scala.collection.mutable.Seq => java.util.List
scala.collection.Set => java.util.Set
scala.collection.Map => java.util.Map
</pre>
