英文原文: [Scala School](http://http://twitter.github.io/scala_school/specs.html)，翻译：[ImportNew](http://www.importnew.com) - [朱伟杰](http://www.importnew.com/author/zhuweijie)

This lesson covers testing with Specs, a Behavior-Driven Design (BDD) Framework for Scala.

* "extends Specification":#example
** nested examples
* "Execution Model":#scope
* "Setup and TearDown":#setup
** doFirst
** doBefore
** doAfter
* "Matchers":#matchers
** mustEqual
** contains
** sameSize?
** Write your own
* "Mocks":#mocks
* "Spies":#spies
* "run in sbt":#sbt

这个章节的内容包含使用Specs进行测试，同时介绍了一个行为驱动设计（Behavior-Driven Desing BBD)的Scala框架。

* <a href="#example">扩展Specification</a>
  * 内嵌示例
* <a href="#scope">执行模型</a>
* <a href="#setup">环境的设置与清理</a>
  * doFirst
  * doBefore
  * doAfter
* <a href="#matchers">匹配器</a>
  * 相等
  * 包含
  * 数量上相同?
  * 编写自己的匹配器
* <a href="#mocks">Mock</a>
* <a href="#spies">Spies</a>
* <a href="#sbt">在sbt中运行</a>

h2(#example). extends Specification

Let's just jump in.


## <a name="example"> 扩展Specification</a>

让我们直接开始吧

<pre class="brush: java; gutter: true">
import org.specs._

object ArithmeticSpec extends Specification {
  "Arithmetic" should {
    "add two numbers" in {
      1 + 1 mustEqual 2
    }
    "add three numbers" in {
      1 + 1 + 1 mustEqual 3
    }
  }
}
</pre>

*Arithmetic* is the *System Under Specification*

*add* is a context.

*add two numbers* and *add three numbers* are examples.

@mustEqual@ indicates an *expectation*

@1 mustEqual 1@ is a common placeholder *expectation* before you start writing real tests.  All examples should have at least one expectation.

*Arithmetic*是“基于指定规范的系统”

*add*是一个上下文。

*add two numbers* 和 *add threes numbers*都是示例。

`mustEqual`表示一个*期望*

在你真正开始写测试之前，`1 mustEqual 1`只是你的*期望*的一个占位符。所有的示例(example)最少要有一个期望（expectation)。

h3. Duplication

Notice how two tests both have @add@ in their name?  We can get rid of that by *nesting* expectations.

### 重复

注意在两个测试用例的名称中都有`add`。我们可以通过*嵌套的*期望(expectation)来消除重复。

<pre class="brush: java; gutter: true">
import org.specs._

object ArithmeticSpec extends Specification {
  "Arithmetic" should {
    "add" in {
      "two numbers" in {
        1 + 1 mustEqual 2
      }
      "three numbers" in {
        1 + 1 + 1 mustEqual 3
      }
    }
  }
}
</pre>

h2(#scope). Execution Model

## <a name="scope">执行模型</a>

<pre class="brush: java; gutter: true">
object ExecSpec extends Specification {
  "Mutations are isolated" should {
    var x = 0
    "x equals 1 if we set it." in {
      x = 1
      x mustEqual 1
    }
    "x is the default value if we don't change it" in {
      x mustEqual 0
    }
  }
}
</pre>

h2(#setup). Setup, Teardown

h3. doBefore & doAfter

## <a name="setup">环境的设置与清理</a>

### doBefore 和 doAfter


<pre class="brush: java; gutter: true">
"my system" should {
  doBefore { resetTheSystem() /** user-defined reset function */ }
  "mess up the system" in {...}
  "and again" in {...}
  doAfter { cleanThingsUp() }
}
</pre>

*NOTE* @doBefore@/@doAfter@ are only run on leaf examples.

*注意* `doBefore`/`doAfter`只会在叶子示例（leaf example）上运行。

h3. doFirst & doLast

@doFirst@/@doLast@ is for single-time setup. (need example, I don't use this)

### doFirst 和 doLast

`doFirst`/`doLast`主要用于一次性的设置。（需要示范吗，不过我不使用它们）

<pre class="brush: java; gutter: true">
"Foo" should {
  doFirst { openTheCurtains() }
  "test stateless methods" in {...}
  "test other stateless methods" in {...}
  doLast { closeTheCurtains() }
}
</pre>

h2(#matchers). Matchers

You have data, you want to make sure it's right. Let's tour the most commonly used matchers. (See Also "Matchers Guide"http://code.google.com/p/specs/wiki/MatchersGuide)

## <a name="matchers">匹配器</a>

你有一些数据，并且想要验证它是正确地。我们可以使用matcher。下面我们来看看常用的一些matcher。（参考<a href="http://code.google.com/p/specs/wiki/MatchersGuide">匹配器指南</a>）

h3. mustEqual

We've seen several examples of mustEqual already.

### 相等

我们前面已经见过一些使用mustEqual的例子。

<pre class="brush: java; gutter: true">
1 mustEqual 1

"a" mustEqual "a"
</pre>

Reference equality, value equality.

可以判断引用或者值是否相同。

h3. elements in a Sequence

### 序列里的值

<pre class="brush: java; gutter: true">
val numbers = List(1, 2, 3)

numbers must contain(1)
numbers must not contain(4)

numbers must containAll(List(1, 2, 3))
numbers must containInOrder(List(1, 2, 3))

List(1, List(2, 3, List(4)), 5) must haveTheSameElementsAs(List(5, List(List(4), 2, 3), 1))
</pre>


h3. Items in a Map

### Map里的元素

<pre class="brush: java; gutter: true">
map must haveKey(k)
map must notHaveKey(k)

map must haveValue(v)
map must notHaveValue(v)
</pre>

h3. Numbers

### 数字

<pre class="brush: java; gutter: true">
a must beGreaterThan(b)
a must beGreaterThanOrEqualTo(b)

a must beLessThan(b)
a must beLessThanOrEqualTo(b)

a must beCloseTo(b, delta)
</pre>


h3. Options

### Options

<pre class="brush: java; gutter: true">
a must beNone

a must beSome[Type]

a must beSomething

a must beSome(value)
</pre>

h3. throwA

### throwA

<pre class="brush: java; gutter: true">
a must throwA[WhateverException]
</pre>

This is shorter than a try catch with a fail in the body.

You can also expect a specific message

这个比使用带有fail的try catch简洁多了。

你也可以加上一个特定消息的判定。

<pre class="brush: java; gutter: true">
a must throwA(WhateverException("message"))
</pre>

You can also match on the exception:

你也可以对期望进行match匹配：

<pre class="brush: java; gutter: true">
a must throwA(new Exception) like {
  case Exception(m) => m.startsWith("bad")
}
</pre>


h3. Write your own Matchers

### 编写你自己的匹配器

<pre class="brush: java; gutter: true">
import org.specs.matcher.Matcher
</pre>

h4. As a val

#### 作为一个val

<pre class="brush: java; gutter: true">
"A matcher" should {
  "be created as a val" in {
    val beEven = new Matcher[Int] {
      def apply(n: => Int) = {
        (n % 2 == 0, "%d is even".format(n), "%d is odd".format(n))
      }
    }
    2 must beEven
  }
}
</pre>

The contract is to return a tuple containing whether the expectation is true, and a message for when it is and isn't true.

这里对于返回值有一个约定，返回的结果是一个元组，它包含这个期望（expectation）是否为true，以及在不为true的情况下的消息。

h4. As a case class

####　作为一个case class

<pre class="brush: java; gutter: true">
case class beEven(b: Int) extends Matcher[Int]() {
  def apply(n: => Int) =  (n % 2 == 0, "%d is even".format(n), "%d is odd".format(n))
}
</pre>

Using a case class makes it more shareable.

使用case class 可以提高它的共用性。

h2(#mocks). Mocks

## <a name="mocks">Mocks</a>

<pre class="brush: java; gutter: true">
import org.specs.Specification
import org.specs.mock.Mockito

class Foo[T] {
  def get(i: Int): T
}

object MockExampleSpec extends Specification with Mockito {
  val m = mock[Foo[String]]

  m.get(0) returns "one"

  m.get(0)

  there was one(m).get(0)

  there was no(m).get(1)
}
</pre>

*See Also* "Using Mockito":http://code.google.com/p/specs/wiki/UsingMockito

*参考* [Using Mockito](http://code.google.com/p/specs/wiki/UsingMockito)

h2(#spies). Spies

Spies can also be used in order to do some "partial mocking" of real objects:

## <a name="spies">Spies</a>

Spy可以用来对一个具体的对象进行“部分mock”：

<pre class="brush: java; gutter: true">
val list = new LinkedList[String]
val spiedList = spy(list)

// methods can be stubbed on a spy
//在spy里，一个方法返回值可以被替代
spiedList.size returns 100

// other methods can also be used
// 同时其他的方法可以正常使用
spiedList.add("one")
spiedList.add("two")

// and verification can happen on a spy

// 然后可以在spy上进行验证

there was one(spiedList).add("one")
</pre>

However, working with spies can be tricky:

不过，使用spy需要一些技巧：

<pre class="brush: java; gutter: true">
// if the list is empty, this will throws an IndexOutOfBoundsException
//如果列表为空的话，下面的操作会抛出IndexOutOfBoundsException
spiedList.get(0) returns "one"
</pre>

@doReturn@ must be used in that case:

可以通过`doReturn`来处理上面的场景：

<pre class="brush: java; gutter: true">
doReturn("one").when(spiedList).get(0)
</pre>


h2(#sbt). Run individual specs in sbt

## <a name="sbt">在sbt里单独运行指定的spec</a>


<pre class="brush: java; gutter: true">
> test-only com.twitter.yourservice.UserSpec
</pre>

Will run just that spec.

这样会只运行指定的spec。


<pre class="brush: java; gutter: true">
> ~ test-only com.twitter.yourservice.UserSpec
</pre>

Will run that test in a loop, with each file modification triggering a test run.

这样的话，会根据文件的改动来不停地触发这个测试用例的运行。
