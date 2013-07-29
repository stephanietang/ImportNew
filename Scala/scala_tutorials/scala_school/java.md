英文原文: [Scala School](http://twitter.github.io/scala_school/java.html)，翻译：[ImportNew](http://www.importnew.com) - [朱伟杰](http://www.importnew.com/author/zhuweijie)

This lesson covers Java interoperability.

* Javap
* Classes
* Exceptions
* Traits
* Objects
* Closures and Functions
* Variance

这个章节主要讲解和Java进行互操作。

* <a href="#javap">Javap</a>
* <a href="#class">类</a>
* <a href="#exception">异常</a>
* <a href="#trait">Trait</a>
* <a href="#object">对象</a>
* <a href="#closure">闭包函数(closures functions)</a>

h2. Javap

javap is a tool that ships with the JDK.  Not the JRE.  There's a difference.  Javap decompiles class definitions and shows you what's inside.  Usage is pretty simple

## <a name="javap">Javap</a>

javap是JDK附带的一个工具，而不是JRE。它们之间还是有差别的。Javap反编译class文件，并且向你展示它里面放的是什么。使用起来很简单。

<pre class="brush: java; gutter: true">
[local ~/projects/interop/target/scala_2.8.1/classes/com/twitter/interop]$ javap MyTrait
Compiled from "Scalaisms.scala"
public interface com.twitter.interop.MyTrait extends scala.ScalaObject{
    public abstract java.lang.String traitName();
    public abstract java.lang.String upperTraitName();
}
</pre>

If you're hardcore you can look at byte code

如果你想了解底层的话，你可以查看对应的字节码

<pre class="brush: java; gutter: true">
[local ~/projects/interop/target/scala_2.8.1/classes/com/twitter/interop]$ javap -c MyTrait\$class
Compiled from "Scalaisms.scala"
public abstract class com.twitter.interop.MyTrait$class extends java.lang.Object{
public static java.lang.String upperTraitName(com.twitter.interop.MyTrait);
  Code:
   0: aload_0
   1: invokeinterface #12,  1; //InterfaceMethod com/twitter/interop/MyTrait.traitName:()Ljava/lang/String;
   6: invokevirtual #17; //Method java/lang/String.toUpperCase:()Ljava/lang/String;
   9: areturn

public static void $init$(com.twitter.interop.MyTrait);
  Code:
   0: return

}
</pre>

If you start wondering why stuff doesn't work in Java land, reach for javap!

如果你在Java平台上有什么问题，你可以通过javap来排查。

h2. Classes

The four major items to consider when using a Scala _class_ from Java are

* Class parameters
* Class vals
* Class vars
* Exceptions

We'll construct a simple scala class to show the full range of entities

## <a name="class">类</a>

从Java的角度来使用Scala的\_class\_需要注意的四个要点如下：

* 类参数
* 类常量
* 类变量
* 异常

我们来创建一个简单的scala类来展示这几个要点

<pre class="brush: java; gutter: true">
package com.twitter.interop

import java.io.IOException
import scala.throws
import scala.reflect.{BeanProperty, BooleanBeanProperty}

class SimpleClass(name: String, val acc: String, @BeanProperty var mutable: String) {
  val foo = "foo"
  var bar = "bar"
  @BeanProperty
  val fooBean = "foobean"
  @BeanProperty
  var barBean = "barbean"
  @BooleanBeanProperty
  var awesome = true

  def dangerFoo() = {
    throw new IOException("SURPRISE!")
  }

  @throws(classOf[IOException])
  def dangerBar() = {
    throw new IOException("NO SURPRISE!")
  }
}
</pre>

h3. Class parameters

* by default, class parameters are effectively constructor args in Java land.  This means you can't access them outside the class.
* declaring a class parameter as a val/var is the same as this code

### 类参数

* 默认情况下，类参数实际上就是Java里构造函数的参数。这就意味着你不能在这个class之外访问它们。
* 把类参数定义成一个val/var的方式和下面的代码相同

<pre class="brush: java; gutter: true">
class SimpleClass(acc_: String) {
  val acc = acc_
}
</pre>
which makes it accessible from Java code just like other vals

下面就可以通过Java代码来访问它们了。

h3. Vals

* vals get a method defined for access from Java.  You can access the value of the val "foo" via the method "foo()"

### 常量（Val）

* 常量（val）都会定义有对应的供Java代码访问的方法。你可以通过"foo()"方法来取得常量（val）“foo”的值。

h3. Vars

* vars get a method <name>_$eq defined.  You can call it like so

### 变量（Var）

* 变量（var）会多定义一个<name>_$eq方法。你可以这样调用来设置变量的值：

<pre class="brush: java; gutter: true">
foo$_eq("newfoo");
</pre>

h3. BeanProperty

You can annotate vals and vars with the @BeanProperty annotation.  This generates getters/setters that look like POJO getter/setter definitions.  If you want the isFoo variant, use the BooleanBeanProperty annotation.  The ugly foo$_eq becomes

### BeanFactory

你可以通过@BeanProperty注解来标注val和var。这样就会生成类似于POJO的getter/setter方法。假如你想要访问isFoo变量，使用BooleanBeanProperty注解。那么难以理解的foo$eq就可以换成：

<pre class="brush: java; gutter: true">
setFoo("newfoo");
getFoo();
</pre>


h3. Exceptions

Scala doesn't have checked exceptions.  Java does.  This is a philosophical debate we won't get into, but it *does* matter when you want to catch an exception in Java.  The definitions of dangerFoo and dangerBar demonstrate this.  In Java I can't do this

### <a name="exception">异常</a>

Scala里没有受检异常（checked exception），但是Java里有。这是一个语言层面上的问题，我们这里不进行讨论，但是在Java里你对异常进行捕获的时候你还是要注意的。dangerFoo和dangerBar的定义里对这进行了示范。在Java里，你不能这样做。

<pre class="brush: java; gutter: true">
        // exception erasure!
        // 异常擦除！
        try {
            s.dangerFoo();
        } catch (IOException e) {
            // UGLY
            // 非常丑陋
        }

</pre>

Java complains that the body of s.dangerFoo never throws IOException.  We can hack around this by catching Throwable, but that's lame.

Java编译器会因为s.dangerFoo不会抛出IOException而报错。我们可以通过捕获Throwable来绕过这个错误，但是这样做没多大用处。

Instead, as a good Scala citizen it's a decent idea to use the throws annotation like we did on dangerBar.  This allows us to continue using checked exceptions in Java land.

不过，作为一个Scala用户，比较正式的方式是使用throws注解，就像我们之前在dangerBar上的一样。这个手段使得我们能够使用Java里的受检异常（checked exception）。

h3. Further Reading

A full list of Scala annotations for supporting Java interop can be found here http://www.scala-lang.org/node/106.

### 延伸阅读

支持Java互操作的注解的完整列表在[http://www.scala-lang.org/node/106](http://www.scala-lang.org/node/106)。

h2. Traits

How do you get an interface + implementation?  Let's take a simple trait definition and look

## <a name="trait">Trait</a>

我们怎样可以得到一个接口和对应的实现呢？我们简单看看trait的定义


<pre class="brush: java; gutter: true">
trait MyTrait {
  def traitName:String
  def upperTraitName = traitName.toUpperCase
}
</pre>

This trait has one abstract method (traitName) and one implemented method (upperTraitName).  What does Scala generate for us?  An interface named MyTrait, and a companion implementation named MyTrait$class.

这个trait有一个抽象的方法（traitName）和一个已实现的方法（upperTraitName）。对于这样的trait，Scala会生成什么样的代码呢？它会生成一个MyTrait接口，同时还会生成对应的实现类MyTrait$class。

The implementation of MyTrait is what you'd expect

MyTrait的实现和你猜想的差不多：

<pre class="brush: java; gutter: true">
[local ~/projects/interop/target/scala_2.8.1/classes/com/twitter/interop]$ javap MyTrait
Compiled from "Scalaisms.scala"
public interface com.twitter.interop.MyTrait extends scala.ScalaObject{
    public abstract java.lang.String traitName();
    public abstract java.lang.String upperTraitName();
}
</pre>

The implementation of MyTrait$class is more interesting

不过MyTrait$class的实现更加有趣：

<pre class="brush: java; gutter: true">
[local ~/projects/interop/target/scala_2.8.1/classes/com/twitter/interop]$ javap MyTrait\$class
Compiled from "Scalaisms.scala"
public abstract class com.twitter.interop.MyTrait$class extends java.lang.Object{
    public static java.lang.String upperTraitName(com.twitter.interop.MyTrait);
    public static void $init$(com.twitter.interop.MyTrait);
}
</pre>

MyTrait$class has only static methods that take an instance of MyTrait.  This gives us a clue as to how to extend a Trait in Java.

MyTrait$class类只有一个静态的接受一个MyTrait实例作为参数的方法。这样给了我们在Java如何实现trait的一条线索。

Our first try is the following

首先要做的是：

<pre class="brush: java; gutter: true">
package com.twitter.interop;

public class JTraitImpl implements MyTrait {
    private String name = null;

    public JTraitImpl(String name) {
        this.name = name;
    }

    public String traitName() {
        return name;
    }
}
</pre>

And we get the following error

然后我们会得到下面的错误：

<pre class="brush: java; gutter: true">
[info] Compiling main sources...
[error] /Users/mmcbride/projects/interop/src/main/java/com/twitter/interop/JTraitImpl.java:3: com.twitter.interop.JTraitImpl is not abstract and does not override abstract method upperTraitName() in com.twitter.interop.MyTrait
[error] public class JTraitImpl implements MyTrait {
[error]        ^
</pre>

We _could_ just implement this ourselves.  But there's a sneakier way.

我们_可以_自己来实现他们。不过还有一种更加诡异的方式。

<pre class="brush: java; gutter: true">
package com.twitter.interop;

    public String upperTraitName() {
        return MyTrait$class.upperTraitName(this);
    }
</pre>

We can just delegate this call to the generated Scala implementation.  We can also override it if we want.

我们只需要把相应的方法调用代理到Scala的实现上。并且我们还可以根据实际需要进行重写。


h2.  Objects

Objects are the way Scala implements static methods/singletons.  Using them from Java is a bit odd.  There isn't a stylistically perfect way to use them, but in Scala 2.8 it's not terrible

A Scala object is compiled to a class that has a trailing "$".  Let's set up a class and a companion object

## <a name="object">对象</a>

在Scala里，是用对象来实现静态方法和单例模式的。如果在Java里使用它们就会显得比较怪。在语法上没有什么比较优雅的方式来使用它们，但是在Scala 2.8里就没有那么麻烦了。

Scala对象会被编译成一个名称带有“$”后缀的类。我们来创建一个类以及对应的对象（Object）。我们来创建一个类以及对应的伴生对象（companion object)。

<pre class="brush: java; gutter: true">
class TraitImpl(name: String) extends MyTrait {
  def traitName = name
}

object TraitImpl {
  def apply = new TraitImpl("foo")
  def apply(name: String) = new TraitImpl(name)
}
</pre>

We can naively access this in Java like so

我们可以通过下面这种奇妙的方式在Java里访问它：

<pre class="brush: java; gutter: true">
MyTrait foo = TraitImpl$.MODULE$.apply("foo");
</pre>

Now you may be asking yourself, WTF?  This is a valid response.  Let's look at what's actually inside TraitImpl$

现在你也许会问自己，这究竟是神马？这是一个很正常的反应。我们现在一起来看看TraintImpl$内部究竟是怎么实现的。

<pre class="brush: java; gutter: true">
local ~/projects/interop/target/scala_2.8.1/classes/com/twitter/interop]$ javap TraitImpl\$
Compiled from "Scalaisms.scala"
public final class com.twitter.interop.TraitImpl$ extends java.lang.Object implements scala.ScalaObject{
    public static final com.twitter.interop.TraitImpl$ MODULE$;
    public static {};
    public com.twitter.interop.TraitImpl apply();
    public com.twitter.interop.TraitImpl apply(java.lang.String);
}
</pre>

There actually aren't any static methods.  Instead it has a static member named MODULE$.  The method implementations delegate to this member.  This makes access ugly, but workable if you know to use MODULE$.

其实它里面没有任何静态方法。相反，它还有一个静态成员叫做MODULE$。实际上方法的调用都是代理到这个成员变量上的。这种实现使得访问起来觉得比较恶心，但是如果你知道怎么使用MODULE$的话，其实还是很实用的。

h3.  Forwarding Methods

In Scala 2.8 dealing with Objects got quite a bit easier.  If you have a class with a companion object, the 2.8 compiler generates forwarding methods on the companion class.  So if you built with 2.8, you can access methods in the TraitImpl Object like so

### 转发方法（Forwarding Method）

在Scala 2.8里，处理Object会比较简单点。如果你有一个类以及对应的伴生对象（companion object），2.8 的编译器会在伴生对象里生成转发方法。如果使用2.8的编译器进行构建，那么你可以通过下面的方法来访问TraitImpl对象：

<pre class="brush: java; gutter: true">
MyTrait foo = TraitImpl.apply("foo");
</pre>

h2. Closures Functions

One of Scala's most important features is the treatment of functions as first class citizens.  Let's define a class that defines some methods that take functions as arguments.

## <a name="closure">闭包函数</a>

Scala最重要的一个特点就是把函数作为一等公民。我们来定义一个类，它里面包含一些接收函数作为参数的方法。

<pre class="brush: java; gutter: true">
class ClosureClass {
  def printResult[T](f: => T) = {
    println(f)
  }

  def printResult[T](f: String => T) = {
    println(f("HI THERE"))
  }
}
</pre>

In Scala I can call this like so

在Scala里我可以这样调用：

<pre class="brush: java; gutter: true">
val cc = new ClosureClass
cc.printResult { "HI MOM" }
</pre>

In Java it's not so easy, but it's not terrible either.  Let's see what ClosureClass actually compiled to:

但是在Java里却没有这么简单，不过也没有想象的那么复杂。我们来看看ClosureClass最终到底编译成怎样：

<pre class="brush: java; gutter: true">
[local ~/projects/interop/target/scala_2.8.1/classes/com/twitter/interop]$ javap ClosureClass
Compiled from "Scalaisms.scala"
public class com.twitter.interop.ClosureClass extends java.lang.Object implements scala.ScalaObject{
    public void printResult(scala.Function0);
    public void printResult(scala.Function1);
    public com.twitter.interop.ClosureClass();
}
</pre>

This isn't so scary.  "f: => T" translates to "Function0", and "f: String => T" translates to "Function1".  Scala actually defines Function0 through Function22, supporting this stuff up to 22 arguments.  Which really should be enough.

Now we just need to figure out how to get those things going in Java.  Turns out Scala provides an AbstractFunction0 and an AbstractFunction1 we can pass in like so

这个看起来也不是很可怕。"f: => T" 转换成"Function0"，"f: String => T" 转换成 "Function1"。Scala定义了从Function0到Function22，一直支持到22个参数。这么多确实已经足够了。

现在我们只需要弄清楚，怎么在Java去实现这个功能。事实上，Scala提供了AbstractFunction0和AbstractFunction1，我们可以这样来传参：


<pre class="brush: java; gutter: true">
    @Test public void closureTest() {
        ClosureClass c = new ClosureClass();
        c.printResult(new AbstractFunction0() {
                public String apply() {
                    return "foo";
                }
            });
        c.printResult(new AbstractFunction1<String, String>() {
                public String apply(String arg) {
                    return arg + "foo";
                }
            });
    }
</pre>

Note that we can use generics to parameterize arguments.

注意我们还可以使用泛型来参数化参数的类型。

