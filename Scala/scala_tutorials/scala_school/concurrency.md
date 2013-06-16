英文原文: [Scala School](http://twitter.github.io/scala_school/concurrency.html)，翻译：[ImportNew](http://www.importnew.com) - [朱伟杰](http://www.importnew.com/author/zhuweijie)

* "Runnable/Callable":#runnable
* "Threads":#Thread
* "Executors/ExecutorService":#executor
* "Futures":#Future
* "Thread Safety Problem":#danger
* "Example: Search Engine":#example
* "Solutions":#solutions

* <a href="#runnable">Runnable/Callable</a>
* <a href="#threads">Thread</a>
* <a href="#executor">Executors/ExecutorService</a>
* <a href="#future">Future</a>
* <a href="#danger">线程安全问题</a>
* <a href="#example">示例：搜索引擎</a>
* <a href="#solutions">解决方案</a>


h2(#runnable). Runnable/Callable

Runnable has a single method that returns no value.

## <a name="runnable">Runnable/Callable</a>

Runnable只有一个没有返回值的方法

<pre>
trait Runnable {
  def run(): Unit
}
</pre>

Callable is similar to run except that it returns a value

Callable的方法和run类似，只不过它有一个返回值

<pre>
trait Callable[V] {
  def call(): V
}
</pre>


h2(#Thread). Threads

Scala concurrency is built on top of the Java concurrency model.

On Sun JVMs, with a IO-heavy workload, we can run tens of thousands of threads on a single machine.

A Thread takes a Runnable.  You have to call @start@ on a Thread in order for it to run the Runnable.

## <a name="threads">线程</a>

Scala的并发是建立在Java的并发模型上的。

在Sun的JVM上，对于一个IO密集型的任务，我们可以在单机上运行成千上万的线程。

Thread是通过Runnable构造的。要运行一个Runnable的run方法，你需要调用对应线程的`start`方法。

<pre>
scala> val hello = new Thread(new Runnable {
  def run() {
    println("hello world")
  }
})
hello: java.lang.Thread = Thread[Thread-3,5,main]

scala> hello.start
hello world

</pre>

When you see a class implementing Runnable, you know it's intended to run in a Thread somewhere by somebody.

当你看见一个实现Runnable的类，你应该明白它会被放到一个线程里去执行的。

h3. Something single-threaded

Here's a code snippet that works but has problems.

### 一段单线程的代码

下面是一段代码片，它可以运行，但是会有问题。

<pre>
import java.net.{Socket, ServerSocket}
import java.util.concurrent.{Executors, ExecutorService}
import java.util.Date

class NetworkService(port: Int, poolSize: Int) extends Runnable {
  val serverSocket = new ServerSocket(port)

  def run() {
    while (true) {
      // This will block until a connection comes in.
      // 这里会阻塞直到有连接进来
      val socket = serverSocket.accept()
      (new Handler(socket)).run()
    }
  }
}

class Handler(socket: Socket) extends Runnable {
  def message = (Thread.currentThread.getName() + "\n").getBytes

  def run() {
    socket.getOutputStream.write(message)
    socket.getOutputStream.close()
  }
}

(new NetworkService(2020, 2)).run
</pre>

Each request will respond with the name of the current Thread, which is always @main@.

The main drawback with this code is that only one request at a time can be answered!

You could put each request in a Thread.  Simply change


每个请求都会把当前线程的名称`main`作为响应。

这段代码最大的问题在于一次只能够响应一个请求！

你可以对每个请求都单独用一个线程来响应。只需要把

<pre>
(new Handler(socket)).run()
</pre>

to

改成

<pre>
(new Thread(new Handler(socket))).start()
</pre>

but what if you want to reuse threads or have other policies about thread behavior?

但是如果你想要复用线程或者对于线程的行为要做一些其他的控制呢?

h2(#executor). Executors

With the release of Java 5, it was decided that a more abstract interface to Threads was required.

You can get an @ExecutorService@ using static methods on the @Executors@ object.  Those methods provide you to configure an @ExecutorService@ with a variety of policies such as thread pooling.

Here's our old blocking network server written to allow concurrent requests.

## Executors

随着Java 5的发布，对于线程的管理需要一个更加抽象的接口。

你可以通过`Executors`对象的静态方法来取得一个`ExecutorService`对象。这些方法可以让你使用各种不同的策略来配置一个`ExecutorService`，例如线程池。

下面是我们之前的阻塞式网络服务器，现在改写成可以支持并发请求。


<pre>
import java.net.{Socket, ServerSocket}
import java.util.concurrent.{Executors, ExecutorService}
import java.util.Date

class NetworkService(port: Int, poolSize: Int) extends Runnable {
  val serverSocket = new ServerSocket(port)
  val pool: ExecutorService = Executors.newFixedThreadPool(poolSize)

  def run() {
    try {
      while (true) {
        // This will block until a connection comes in.
        val socket = serverSocket.accept()
        pool.execute(new Handler(socket))
      }
    } finally {
      pool.shutdown()
    }
  }
}

class Handler(socket: Socket) extends Runnable {
  def message = (Thread.currentThread.getName() + "\n").getBytes

  def run() {
    socket.getOutputStream.write(message)
    socket.getOutputStream.close()
  }
}

(new NetworkService(2020, 2)).run
</pre>

Here's a transcript connecting to it showing how the internal threads are re-used.

下面是连接的大致情况，可以这个我们可以大致了解内部的线程是怎么进行复用的。

<pre>
$ nc localhost 2020
pool-1-thread-1

$ nc localhost 2020
pool-1-thread-2

$ nc localhost 2020
pool-1-thread-1

$ nc localhost 2020
pool-1-thread-2
</pre>


h2(#Future). Futures

A @Future@ represents an asynchronous computation.  You can wrap your computation in a Future and when you need the result, you simply call a blocking @get()@ method on it.  An @Executor@ returns a @Future@. If you use the Finagle RPC system, you use @Future@ instances to hold results that might not have arrived yet.

A @FutureTask@ is a Runnable and is designed to be run by an @Executor@


## <a name="Future">Futures</a>

一个`Future`代表一次异步计算的操作。你可以把你的操作包装在一个Future里，当你需要结果的时候，你只需要简单调用一个阻塞的`get()`方法就好了。一个`Executor`返回一个`Future`。如果你使用Finagle RPC的话，你可以使用`Future`的实例来保存还没有到达的结果。

`FutureTask`是一个可运行的任务，并且被设计成由`Executor`进行运行。


<pre>
val future = new FutureTask[String](new Callable[String]() {
  def call(): String = {
    searcher.search(target);
}})
executor.execute(future)
</pre>

Now I need the results so let's block until its done.

现在我需要结果，那就只能阻塞到直到结果返回。

<pre>
val blockingResult = future.get()
</pre>

*See Also* <a href="finagle.html">Scala School's Finagle page</a> has plenty of examples of using <code>Future</code>s, including some nice ways to combine them. Effective Scala has opinions about <a href="http://twitter.github.com/effectivescala/#Twitter's standard libraries-Futures">Futures</a> .

**参考** <a href="finagle.html">Scala School中关于Finagle的章节</a>有大量使用`Future`的示例，也有一些组合使用的例子。Effective Scala中也有关于<a href="http://twitter.github.com/effectivescala/#Twitter's standard libraries-Futures">Futures</a>的内容。

h2(#danger). Thread Safety Problem

## <a name="danger">线程安全问题</a>

<pre>
class Person(var name: String) {
  def set(changedName: String) {
    name = changedName
  }
}
</pre>

This program is not safe in a multi-threaded environment.  If two threads have references to the same instance of Person and call @set@, you can't predict what @name@ will be at the end of both calls.

In the Java memory model, each processor is allowed to cache values in its L1 or L2 cache so two threads running on different processors can each have their own view of data.

Let's talk about some tools that force threads to keep a consistent view of data.

这个程序在多线程的环境下是不安全的。如果两个线程都有同一个Person示例的引用，并且都调用`set`方法，你没法预料在两个调用都结束的时候`name`会是什么。

在Java的内存模型里，每个处理器都允许在它的L1或者L2 cache里缓存变量，所以两个在不同处理器上运行的线程对于相同的数据有种不同的视图。

下面我们来讨论一下可以强制线程的数据视图保持一致的工具。



h3. Three tools

h4. synchronization

Mutexes provide ownership semantics.  When you enter a mutex, you own it.  The most common way of using a mutex in the JVM is by synchronizing on something.  In this case, we'll synchronize on our Person.

In the JVM, you can synchronize on any instance that's not null.

### 三个工具

#### 同步
Mutex提供了锁定资源的语法。当你进入一个mutex的时候，你会获得它。在JVM里使用mutex最常用的方式就是在一个对象上进行同步访问。在这里，我们会在Person上进行同步访问。

在JVM里，你可以对任何非null的对象进行同步访问。

<pre>
class Person(var name: String) {
  def set(changedName: String) {
    this.synchronized {
      name = changedName
    }
  }
}
</pre>

h4. volatile

#### volatile

With Java 5's change to the memory model, volatile and synchronized are basically identical except with volatile, nulls are allowed.

@synchronized@ allows for more fine-grained locking.  @volatile@ synchronizes on every access.

随着Java 5对于内存模型的改变，volatile和synchronized的作用基本相同，除了一点，volatile也可以用在null上。

`synchronized`提供了更加细粒度的加锁控制。而`volatile`直接是对每次访问进行控制。




<pre>
class Person(@volatile var name: String) {
  def set(changedName: String) {
    name = changedName
  }
}
</pre>

h4. AtomicReference

Also in Java 5, a whole raft of low-level concurrency primitives were added. One of them is an @AtomicReference@ class

#### AtomaticReference

同样的，在Java 5中新增了一系列底层的并发原语。`AtomicReference`类就是其中一个。

<pre>
import java.util.concurrent.atomic.AtomicReference

class Person(val name: AtomicReference[String]) {
  def set(changedName: String) {
    name.set(changedName)
  }
}
</pre>

h4. Does this cost anything?

@AtomicReference is the most costly of these two choices since you have to go through method dispatch to access values.

@volatile@ and @synchronized@ are built on top of Java's built-in monitors.  Monitors cost very little if there's no contention.  Since @synchronized@ allows you more fine-grained control over when you synchronize, there will be less contention so @synchronized@ tends to be the cheapest option.

When you enter synchronized points, access volatile references, or deference AtomicReferences, Java forces the processor to flush their cache lines and provide a consistent view of data.

PLEASE CORRECT ME IF I'M WRONG HERE.  This is a complicated subject, I'm sure there will be a lengthy classroom discussion at this point.

#### 它们都有额外的消耗吗？

`AutomicReference`是这两种方式中最耗性能的，因为如果你要取得对应的值，则需要经过方法分派(method dispatch)的过程。

`volatile`和`synchronized`都是通过Java内置的monitor来实现的。在没有竞争的情况下，monitor对性能的影响非常小。由于`synchronized`允许你对代码进行更加细粒度的加锁控制，这样就可以减小加锁区，进而减小竞争，因此`synchronized`应该是最佳的选择。

当你进入同步块，访问volatile引用，或者引用AtomicReference，Java会强制要求处理器刷新它们的缓存流水线，从而保证数据的一致性。

如果我这里说错了，请指正出来。这是一个很复杂的主题，对于这个主题肯定需要花费大量的时间来进行讨论。


h3. Other neat tools from Java 5

As I mentioned with @AtomicReference@, Java 5 brought many great tools along with it.

### 其他来自Java 5的优秀工具

之前提到了`AtomicReference`，除了它之外，Java 5还提供了很多其他有用的工具。


h4. CountDownLatch

A @CountDownLatch@ is a simple mechanism for multiple threads to communicate with each other.

#### CountDownLatch

`CountDownLatch`是供多个进程进行通信的一个简单机制。

<pre>
val doneSignal = new CountDownLatch(2)
doAsyncWork(1)
doAsyncWork(2)

doneSignal.await()
println("both workers finished!")
</pre>

Among other things, it's great for unit tests.  Let's say you're doing some async work and want to ensure that functions are completing.  Simply have your functions @countDown@ the latch and @await@ in the test.

除此之外，它对于单元测试也是很有用的。假设你在做一些异步的工作，并且你想要保证所有的功能都完成了。你只需要让你的函数都对latch进行`countDown`操作，然后在你的测试代码里进行`await`。


h4. AtomicInteger/Long

Since incrementing Ints and Longs is such a common task, @AtomicInteger@ and @AtomicLong@ were added.

#### AtomicInteger/Long

由于对于Int和Long的自增操作比较常见，所以就增加了`AtomicInteger`和`AtomicLong`。

h4. AtomicBoolean

I probably don't have to explain what this would be for.

#### AtomicBoolean

我想我没有必要来解释这个的作用了。

h4. ReadWriteLocks

@ReadWriteLock@ lets you take reader and writer locks.  reader locks only block when a writer lock is taken.

#### 读写锁(ReadWriteLock)

`ReadWriteLock`可以实现读写锁，读操作只会在写者加锁的时候进行阻塞。

h2(#example). Let's build an unsafe search engine

Here's a simple inverted index that isn't thread-safe.  Our inverted index maps parts of a name to a given User.

This is written in a naive way assuming only single-threaded access.

Note the alternative default constructor @this()@ that uses a @mutable.HashMap@

## 我们来构建一个非线程安全的搜索引擎

这是一个简单的非线程安全的倒排索引。我们这个倒排索引把名字的一部分映射到指定的用户。

下面是原生的假设只有单线程访问的写法。

注意这里的使用`mutable.HashMap`的另一个构造函数`this()`。

<pre>
import scala.collection.mutable

case class User(name: String, id: Int)

class InvertedIndex(val userMap: mutable.Map[String, User]) {

  def this() = this(new mutable.HashMap[String, User])

  def tokenizeName(name: String): Seq[String] = {
    name.split(" ").map(_.toLowerCase)
  }

  def add(term: String, user: User) {
    userMap += term -> user
  }

  def add(user: User) {
    tokenizeName(user.name).foreach { term =>
      add(term, user)
    }
  }
}
</pre>

I've left out how to get users out of our index for now.  We'll get to that later.

我把具体怎么根据索引获取用户的方法暂时省略掉了，我们后面会来进行补充。

h2(#solutions). Let's make it safe

In our inverted index example above, userMap is not guaranteed to be safe.  Multiple clients could try to add items at the same time and have the same kinds of visibility errors we saw in our first @Person@ example.

Since userMap isn't thread-safe, how we do we keep only a single thread at a time mutating it?

You might consider locking on userMap while adding.

## 我们来让它变得安全

在上面的倒排索引的示例里，userMap是没法保证线程安全的。多个客户端可以同时尝试去添加元素，这样会产生和之前`Person`示例里相似的问题。

因为userMap本身不是线程安全的，那么我们怎么能够保证每次只有一个线程对它进行修改呢？

你需要在添加元素的时候给userMap加锁。



<pre>
def add(user: User) {
  userMap.synchronized {
    tokenizeName(user.name).foreach { term =>
      add(term, user)
    }
  }
}
</pre>

Unfortunately, this is too coarse.  Always try to do as much expensive work outside of the mutex as possible.  Remember what I said about locking being cheap if there is no contention.  If you do less work inside of a block, there will be less contention.

不幸的是，上面的做法有点太粗糙了。能在互斥量(mutex)外面做的工作尽量都放在外面做。记住我之前说过，如果没有竞争的话，加锁的代价是非常小的。如果你在临界区尽量少做操作，那么竞争就会非常少。

<pre>
def add(user: User) {
  // tokenizeName was measured to be the most expensive operation.
  // tokenizeName 这个操作是最耗时的。
  val tokens = tokenizeName(user.name)

  tokens.foreach { term =>
    userMap.synchronized {
      add(term, user)
    }
  }
}
</pre>

h2. SynchronizedMap

We can mixin synchronization with a mutable HashMap using the SynchronizedMap trait.

We can extend our existing InvertedIndex to give users an easy way to build the synchronized index.

## SynchronizedMap

我们可以通过使用SynchronizedMap trait来使得一个可变的(mutable)HashMap具有同步机制。

我们可以扩展之前的InvertedIndex，给用户提供一种构建同步索引的简单方法。

<pre>
import scala.collection.mutable.SynchronizedMap

class SynchronizedInvertedIndex(userMap: mutable.Map[String, User]) extends InvertedIndex(userMap) {
  def this() = this(new mutable.HashMap[String, User] with SynchronizedMap[String, User])
}
</pre>

If you look at the implementation, you realize that it's simply synchronizing on every method so while it's safe, it might not have the performance you're hoping for.

如果你去看具体的实现的话，你会发现SynchronizedMap只是在每个方法上都加上了同步访问，因此它的安全是以牺牲性能为代价的。

h2.  Java ConcurrentHashMap

Java comes with a nice thread-safe ConcurrentHashMap.  Thankfully, we can use JavaConverters to give us nice Scala semantics.



In fact, we can seamlessly layer our new, thread-safe InvertedIndex as an extension of the old unsafe one.

## Java ConcurrentHashMap

Java里有一个很不错的线程安全的ConcurrentHashMap。幸运的是，JavaConverter可以使得我们通过Scala的语法来使用它。

实际上，我们可以无缝地把我们新的，线程安全的InvertedIndex作为老的非线程安全的一个扩展。


<pre>
import java.util.concurrent.ConcurrentHashMap
import scala.collection.JavaConverters._

class ConcurrentInvertedIndex(userMap: collection.mutable.ConcurrentMap[String, User])
    extends InvertedIndex(userMap) {

  def this() = this(new ConcurrentHashMap[String, User] asScala)
}
</pre>

h2. Let's load our InvertedIndex

h3. The naive way

## 现在来加载我们的InvertedIndex

### 最原始的方法

<pre>

trait UserMaker {
  def makeUser(line: String) = line.split(",") match {
    case Array(name, userid) => User(name, userid.trim().toInt)
  }
}

class FileRecordProducer(path: String) extends UserMaker {
  def run() {
    Source.fromFile(path, "utf-8").getLines.foreach { line =>
      index.add(makeUser(line))
    }
  }
}
</pre>

For every line in our file, we call @makeUser@ and then @add@ it to our InvertedIndex.  If we use a concurrent InvertedIndex, we can call add in parallel and since makeUser has no side-effects, it's already thread-safe.

We can't read a file in parallel but we _can_ build the User and add it to the index in parallel.

对于文件里的每一行字符串，我们通过调用`makeUser`来生成一个User，然后通过`add`添加到InvertedIndex里。如果我们并发访问一个InvertedIndex，我们可以并行调用add方法，因为makeUser方法没有副作用，它本身就是线程安全的。

我们不能并行读取一个文件，但是我们_可以_并行构造User，并且并行将它添加到索引里。

h3.  A solution: Producer/Consumer

A common pattern for async computation is to separate producers from consumers and have them only communicate via a @Queue@.  Let's walk through how that would work for our search engine indexer.

### 解决方案：生产者/消费者

<pre>
import java.util.concurrent.{BlockingQueue, LinkedBlockingQueue}

// Concrete producer
class Producer[T](path: String, queue: BlockingQueue[T]) extends Runnable {
  def run() {
    Source.fromFile(path, "utf-8").getLines.foreach { line =>
      queue.put(line)
    }
  }
}

// Abstract consumer
// 抽象的消费者
abstract class Consumer[T](queue: BlockingQueue[T]) extends Runnable {
  def run() {
    while (true) {
      val item = queue.take()
      consume(item)
    }
  }

  def consume(x: T)
}

val queue = new LinkedBlockingQueue[String]()

// One thread for the producer

//一个生产者线程

val producer = new Producer[String]("users.txt", q)
new Thread(producer).start()

trait UserMaker {
  def makeUser(line: String) = line.split(",") match {
    case Array(name, userid) => User(name, userid.trim().toInt)
  }
}

class IndexerConsumer(index: InvertedIndex, queue: BlockingQueue[String]) extends Consumer[String](queue) with UserMaker {
  def consume(t: String) = index.add(makeUser(t))
}

// Let's pretend we have 8 cores on this machine.

// 假设我们的机器有8个核

val cores = 8
val pool = Executors.newFixedThreadPool(cores)

// Submit one consumer per core.

// 每个核设置一个消费者

for (i <- i to cores) {
  pool.submit(new IndexerConsumer[String](index, q))
}
</pre>
