英文原文: [Scala School](http://twitter.github.io/scala_school/sbt.html)，翻译：[ImportNew](http://www.importnew.com) - [朱伟杰](http://www.importnew.com/author/zhuweijie)



This lesson covers SBT! Specific topics include:
* creating an sbt project
* basic commands
* the sbt console
* continuous command execution
* customizing your project
* custom commands
* quick tour of sbt source (if time)

这个章节会讲解SBT(Simple Build Tool)!包含的主题有：
* 创建一个sbt工程
* 基本命令
* sbt控制台
* 连续执行命令
* 自定义工程
* 自定义命令
* sbt代码简介（如果时间允许的话）


h2. About SBT

SBT is a modern build tool. While it is written in Scala and provides many Scala conveniences, it is a general purpose build tool.

## 关于SBT

SBT是一个现代构建工具。它是用Scala编写的，并且针对Scala也提供了很多方便快捷的功能。它也是一个通用的构建工具。

h2. Why SBT?

* Sane(ish) dependency management
** Ivy for dependency management
** Only-update-on-request model
* Full Scala language support for creating tasks
* Continuous command execution
* Launch REPL in project context

## 为什么使用SBT?

* 强大的依赖管理功能
	* Ivy用来管理依赖
	* 一个只会根据需求更新的模型
* 所有任务的创建都支持Scala
* 可连续执行命令
* 可以在工程的上下文里启动REPL


h2. Getting Started

* Download the jar:http://code.google.com/p/simple-build-tool/downloads/list
* Create an sbt shell script that calls the jar, e.g.

## 开始

* 下载jar包：[http://code.google.com/p/simple-build-tool/downloads/list](http://code.google.com/p/simple-build-tool/downloads/list)
* 创建一个stb shell脚本来调用jar包，例如：


<pre class="brush: java; gutter: true">
java -Xmx512M -jar sbt-launch.jar "$@"
</pre>

* make sure it's executable and in your path
* run sbt to create your project

* 保证以上命令能够正确执行，它已经放在了path下
* 运行sbt来创建工程

<pre class="brush: java; gutter: true">
[local ~/projects]$ sbt
Project does not exist, create new project? (y/N/s) y
Name: sample
Organization: com.twitter
Version [1.0]: 1.0-SNAPSHOT
Scala version [2.7.7]: 2.8.1
sbt version [0.7.4]:      
Getting Scala 2.7.7 ...
:: retrieving :: org.scala-tools.sbt#boot-scala
	confs: [default]
	2 artifacts copied, 0 already retrieved (9911kB/221ms)
Getting org.scala-tools.sbt sbt_2.7.7 0.7.4 ...
:: retrieving :: org.scala-tools.sbt#boot-app
	confs: [default]
	15 artifacts copied, 0 already retrieved (4096kB/167ms)
[success] Successfully initialized directory structure.
Getting Scala 2.8.1 ...
:: retrieving :: org.scala-tools.sbt#boot-scala
	confs: [default]
	2 artifacts copied, 0 already retrieved (15118kB/386ms)
[info] Building project sample 1.0-SNAPSHOT against Scala 2.8.1
[info]    using sbt.DefaultProject with sbt 0.7.4 and Scala 2.7.7
> 
</pre>

Note that it's good form to start out with a SNAPSHOT version of your project.

从一个SNAPSHORT版本来开始你的工程是一个不错的方式。

h2. Project Layout

* project - project definition files
** project/build/<yourproject>.scala - the main project definition file
** project/build.properties - project, sbt and scala version definitions
* src/main - your app code goes here, in a subdirectory indicating the
code's language (e.g. src/main/scala, src/main/java)
* src/main/resources - static files you want added to your jar
  (e.g. logging config)
* src/test - like src/main, but for tests
* lib_managed - the jar files your project depends on. Populated by sbt update
* target - the destination for generated stuff (e.g. generated thrift
  code, class files, jars)

## 工程结构

* project - 工程定义文件
  * project/build/<yourproject>.scala - 主要的工程定义文件
  * project/build.properties - 工程，sbt以及scala版本定义
* src/main - 你的应用代码放在这里，不同的子目录名称表示不同的编程语言（例如，src/main/scala,src/main/java)
* src/main/resources - 你想添加到jar包里的静态文件（例如日志配置文件）
* lib_managed - 你的工程所依赖的jar文件。会在sbt更新的时候添加到该目录
* target - 最终生成的文件存放的目录（例如，生成的thrift代码，class文件，jar文件）

h2. Adding Some Code

We'll be creating a simple JSON parser for simple tweets.  Add the
following code to
src/main/scala/com/twitter/sample/SimpleParser.scala

## 添加一些代码

我们会创建一个简单的json解析器来解析简单的tweet。添加下面的代码到
src/main/scala/com/twitter/sample/SimpleParser.scala

<pre class="brush: java; gutter: true">
package com.twitter.sample

case class SimpleParsed(id: Long, text: String)

class SimpleParser {

  val tweetRegex = "\"id\":(.*),\"text\":\"(.*)\"".r

  def parse(str: String) = {
    tweetRegex.findFirstMatchIn(str) match {
      case Some(m) => {
        val id = str.substring(m.start(1), m.end(1)).toInt
        val text = str.substring(m.start(2), m.end(2))
        Some(SimpleParsed(id, text))
      }
      case _ => None
    }
  }
}
</pre>

This is ugly and buggy, but should compile.

这段代码很丑并且有bug，但是它可以通过编译。

h2. Testing in the Console

SBT can be used both as a command line script and as a build console.  We'll be primarily using it as a build console, but most commands can be run standalone by passing the command as an argument to SBT, e.g.

## 在控制台里进行测试

SBT可以被用作命令行脚本也可以被用作是构建控制台。我们主要把它用作构建控制台，不过大部分的命令都可以单独作为参数传给SBT，例如：


<pre class="brush: java; gutter: true">
sbt test
</pre>
Note that if a command takes arguments, you need to quote the entire
argument path, e.g.

注意，如果一个命令接受参数，你需要给参数加上引号

<pre class="brush: java; gutter: true">
sbt 'test-only com.twitter.sample.SampleSpec'
</pre>


It's weird that way.

这种方式很古怪。

Anyway. To start working with our code, launch sbt

暂时不管它。现在启动sbt来构建我们的代码：

<pre class="brush: java; gutter: true">
[local ~/projects/sbt-sample]$ sbt
[info] Building project sample 1.0-SNAPSHOT against Scala 2.8.1
[info]    using sbt.DefaultProject with sbt 0.7.4 and Scala 2.7.7
> 
</pre>

SBT allows you to start a Scala REPL with all your project
dependencies loaded. It compiles your project source before launching
the console, providing us a quick way to bench test our parser.

SBT允许你启动会在你启动Scala REPL的时候加载所有的依赖。它会在启动控制台前先编译工程代码，这样更加便于我们测试我们的解析器了。

<pre class="brush: java; gutter: true">
> console
[info] 
[info] == compile ==
[info]   Source analysis: 0 new/modified, 0 indirectly invalidated, 0 removed.
[info] Compiling main sources...
[info] Nothing to compile.
[info]   Post-analysis: 3 classes.
[info] == compile ==
[info] 
[info] == copy-test-resources ==
[info] == copy-test-resources ==
[info] 
[info] == test-compile ==
[info]   Source analysis: 0 new/modified, 0 indirectly invalidated, 0 removed.
[info] Compiling test sources...
[info] Nothing to compile.
[info]   Post-analysis: 0 classes.
[info] == test-compile ==
[info] 
[info] == copy-resources ==
[info] == copy-resources ==
[info] 
[info] == console ==
[info] Starting scala interpreter...
[info] 
Welcome to Scala version 2.8.1.final (Java HotSpot(TM) 64-Bit Server VM, Java 1.6.0_22).
Type in expressions to have them evaluated.
Type :help for more information.

scala> 
</pre>

Our code has compiled, and we're provide the typical Scala prompt.  We'll create a new parser, an exemplar tweet, and ensure it "works"

代码编译完成，并且提供了经典的Scala命令行提示符。我们会创建一个新的解析器，一个示例的tweet，并且保证它能够正常“工作”。

<pre class="brush: java; gutter: true">
scala> import com.twitter.sample._            
import com.twitter.sample._

scala> val tweet = """{"id":1,"text":"foo"}"""
tweet: java.lang.String = {"id":1,"text":"foo"}

scala> val parser = new SimpleParser          
parser: com.twitter.sample.SimpleParser = com.twitter.sample.SimpleParser@71060c3e

scala> parser.parse(tweet)                    
res0: Option[com.twitter.sample.SimpleParsed] = Some(SimpleParsed(1,"foo"}))

scala> 
</pre>

h2. Adding Dependencies

Our simple parser works for this very small set of inputs, but we want to add tests and break it.  The first step is adding the specs test library and a real JSON parser to our project.  To do this we have to go beyond the default SBT project layout and create a project.

SBT considers Scala files in the project/build directory to be project definitions.  Add the following to project/build/SampleProject.scala

## 添加依赖

这个简单的解析器对于这点输入内容是可以正常工作的，但是我们还需要加入测试代码并且对它进行一些改造。首先要做的就是把specs测试库以及一个真正的JSON解析器加入到我们的工程里来。为了达到这个目标，我们需要在默认的工程结构上进行改造，然后创建项目。把下面的内容添加到project/build/SampleProject.scala里：

<pre class="brush: java; gutter: true">
import sbt._

class SampleProject(info: ProjectInfo) extends DefaultProject(info) {
  val jackson = "org.codehaus.jackson" % "jackson-core-asl" % "1.6.1"
  val specs = "org.scala-tools.testing" % "specs_2.8.0" % "1.6.5" % "test"
}
</pre> 

A project definition is an SBT class.  In our case we extend SBT's DefaultProject.

You declare dependencies by specifying a val that is a dependency.  SBT uses reflection to scan all the dependency vals in your project and build up a dependency tree at build time.  The syntax here may be new, but this is equivalent to the maven dependency

一个工程的定义就是一个SBT类。在这里我们继承了SBT的DefaultProject类。

这里你可以通过一个常量来指定具体的依赖。SBT使用在构建期通过反射来扫描你工程里所有的依赖常量，并且生成一个依赖树。这个语法可能比较新，不过它和下面的maven依赖是等同的：

<pre class="brush: java; gutter: true">
<dependency>
  <groupId>org.codehaus.jackson</groupId>
  <artifactId>jackson-core-asl</artifactId>
  <version>1.6.1</version>
</dependency>
<dependency>
  <groupId>org.scala-tools.testing</groupId>
  <artifactId>specs_2.8.0</artifactId>
  <version>1.6.5</version>
  <scope>test</scope>
</dependency>
</pre>

Now we can pull down dependencies for our project.  From the command line (not the sbt console), run sbt update

现在我们可以把依赖的库下载下来了。从命令行（而不是sbt控制台）里运行sbt update

<pre class="brush: java; gutter: true">
[local ~/projects/sbt-sample]$ sbt update
[info] Building project sample 1.0-SNAPSHOT against Scala 2.8.1
[info]    using SampleProject with sbt 0.7.4 and Scala 2.7.7
[info] 
[info] == update ==
[info] :: retrieving :: com.twitter#sample_2.8.1 [sync]
[info] 	confs: [compile, runtime, test, provided, system, optional, sources, javadoc]
[info] 	1 artifacts copied, 0 already retrieved (2785kB/71ms)
[info] == update ==
[success] Successful.
[info] 
[info] Total time: 1 s, completed Nov 24, 2010 8:47:26 AM
[info] 
[info] Total session time: 2 s, completed Nov 24, 2010 8:47:26 AM
[success] Build completed successfully.
</pre>

You'll see that sbt retrieved the specs library.  You'll now also have a lib_managed directory, and lib_managed/scala_2.8.1/test will have specs_2.8.0-1.6.5.jar

你会看到sbt解析出了specs库。 你的工程下面现在有了lib_managed目录,并且在lib_managed/scala_2.8.1/test目录下会有specs_2.8.0-1.6.5.jar文件。

h2. Adding Tests

Now that we have a test library added, put the following code in
src/test/scala/com/twitter/sample/SimpleParserSpec.scala


## 添加测试用例

现在我们添加了测试二方库，把下面的代码添加到src/test/scala/com/twitter/sample/SimpleParserSpec.scala里

<pre class="brush: java; gutter: true">
package com.twitter.sample

import org.specs._

object SimpleParserSpec extends Specification {
  "SimpleParser" should {
    val parser = new SimpleParser()
    "work with basic tweet" in {
      val tweet = """{"id":1,"text":"foo"}"""
      parser.parse(tweet) match {
        case Some(parsed) => {
          parsed.text must be_==("foo")
          parsed.id must be_==(1)
        }
        case _ => fail("didn't parse tweet")
      }
    }
  }
}
</pre>

In the sbt console, run test

在sbt控制台里，运行test命令

<pre class="brush: java; gutter: true">
> test
[info] 
[info] == compile ==
[info]   Source analysis: 0 new/modified, 0 indirectly invalidated, 0 removed.
[info] Compiling main sources...
[info] Nothing to compile.
[info]   Post-analysis: 3 classes.
[info] == compile ==
[info] 
[info] == test-compile ==
[info]   Source analysis: 0 new/modified, 0 indirectly invalidated, 0 removed.
[info] Compiling test sources...
[info] Nothing to compile.
[info]   Post-analysis: 10 classes.
[info] == test-compile ==
[info] 
[info] == copy-test-resources ==
[info] == copy-test-resources ==
[info] 
[info] == copy-resources ==
[info] == copy-resources ==
[info] 
[info] == test-start ==
[info] == test-start ==
[info] 
[info] == com.twitter.sample.SimpleParserSpec ==
[info] SimpleParserSpec
[info] SimpleParser should
[info]   + work with basic tweet
[info] == com.twitter.sample.SimpleParserSpec ==
[info] 
[info] == test-complete ==
[info] == test-complete ==
[info] 
[info] == test-finish ==
[info] Passed: : Total 1, Failed 0, Errors 0, Passed 1, Skipped 0
[info]  
[info] All tests PASSED.
[info] == test-finish ==
[info] 
[info] == test-cleanup ==
[info] == test-cleanup ==
[info] 
[info] == test ==
[info] == test ==
[success] Successful.
[info] 
[info] Total time: 0 s, completed Nov 24, 2010 8:54:45 AM
> 
</pre>

Our test works!  Now we can add more.  One of the nice things SBT provides is a way to run triggered actions.  Prefacing an action with a tilde starts up a loop that runs the action any time source files change.  Lets run ~test and see what happens.


我们的测试用例执行了！现在我们可以添加更多的测试用例。SBT提供的一个很好的功能就是自动运行被触发的动作。它会预先启动一个循环，不断检测代码改动，一旦有改动就执行相应的动作。我们来运行~test命令，看看有什么效果。

<pre class="brush: java; gutter: true">
[info] == test ==
[success] Successful.
[info] 
[info] Total time: 0 s, completed Nov 24, 2010 8:55:50 AM
1. Waiting for source changes... (press enter to interrupt)
</pre>

Now let's add the following test cases

现在，我们添加下面的测试用例：

<pre class="brush: java; gutter: true">
    "reject a non-JSON tweet" in {
      val tweet = """"id":1,"text":"foo""""
      parser.parse(tweet) match {
        case Some(parsed) => fail("didn't reject a non-JSON tweet")
        case e => e must be_==(None)
      }
    }

    "ignore nested content" in {
      val tweet = """{"id":1,"text":"foo","nested":{"id":2}}"""
      parser.parse(tweet) match {
        case Some(parsed) => {
          parsed.text must be_==("foo")
          parsed.id must be_==(1)
        }
        case _ => fail("didn't parse tweet")
      }
    }

    "fail on partial content" in {
      val tweet = """{"id":1}"""
      parser.parse(tweet) match {
        case Some(parsed) => fail("didn't reject a partial tweet")
        case e => e must be_==(None)
      }
    }
</pre>

After we save our file, SBT detects our changes, runs tests, and informs us our parser is lame

一旦我们保存好文件，SBT会检测到改动，它会运行测试代码，并且告诉我们parser的实现有问题。

<pre class="brush: java; gutter: true">
[info] == com.twitter.sample.SimpleParserSpec ==
[info] SimpleParserSpec
[info] SimpleParser should
[info]   + work with basic tweet
[info]   x reject a non-JSON tweet
[info]     didn't reject a non-JSON tweet (Specification.scala:43)
[info]   x ignore nested content
[info]     'foo","nested":{"id' is not equal to 'foo' (SimpleParserSpec.scala:31)
[info]   + fail on partial content
</pre>

So let's rework our JSON parser to be real

那么我们就来重构JSON parser的代码，让它更接近真实的parser。

<pre class="brush: java; gutter: true">
package com.twitter.sample

import org.codehaus.jackson._
import org.codehaus.jackson.JsonToken._

case class SimpleParsed(id: Long, text: String)

class SimpleParser {

  val parserFactory = new JsonFactory()

  def parse(str: String) = {
    val parser = parserFactory.createJsonParser(str)
    if (parser.nextToken() == START_OBJECT) {
      var token = parser.nextToken()
      var textOpt:Option[String] = None
      var idOpt:Option[Long] = None
      while(token != null) {
        if (token == FIELD_NAME) {
          parser.getCurrentName() match {
            case "text" => {
              parser.nextToken()
              textOpt = Some(parser.getText())
            }
            case "id" => {
              parser.nextToken()
              idOpt = Some(parser.getLongValue())
            }
            case _ => // noop
          }
        }
        token = parser.nextToken()
      }
      if (textOpt.isDefined && idOpt.isDefined) {
        Some(SimpleParsed(idOpt.get, textOpt.get))
      } else {
        None
      }
    } else {
      None
    }
  }
}
</pre>

This is a simple Jackson parser.  When we save, SBT recompiles our code and reruns our tests.  Getting better!

这是一个简单的Json解析器。当我们保存代码时，SBT会编译我们的代码并且运行测试代码。越来越方便了！

<pre class="brush: java; gutter: true">
info] SimpleParser should
[info]   + work with basic tweet
[info]   + reject a non-JSON tweet
[info]   x ignore nested content
[info]     '2' is not equal to '1' (SimpleParserSpec.scala:32)
[info]   + fail on partial content
[info] == com.twitter.sample.SimpleParserSpec ==
</pre>

Uh oh.  We need to check for nested objects.  Let's add some ugly
guards to our token reading loop.

噢。我们需要考虑嵌套的对象。我们来给读入token的循环加上恶心的处理代码。

<pre class="brush: java; gutter: true">
  def parse(str: String) = {
    val parser = parserFactory.createJsonParser(str)
    var nested = 0
    if (parser.nextToken() == START_OBJECT) {
      var token = parser.nextToken()
      var textOpt:Option[String] = None
      var idOpt:Option[Long] = None
      while(token != null) {
        if (token == FIELD_NAME && nested == 0) {
          parser.getCurrentName() match {
            case "text" => {
              parser.nextToken()
              textOpt = Some(parser.getText())
            }
            case "id" => {
              parser.nextToken()
              idOpt = Some(parser.getLongValue())
            }
            case _ => // noop
          }
        } else if (token == START_OBJECT) {
          nested += 1
        } else if (token == END_OBJECT) {
          nested -= 1
        }
        token = parser.nextToken()
      }
      if (textOpt.isDefined && idOpt.isDefined) {
        Some(SimpleParsed(idOpt.get, textOpt.get))
      } else {
        None
      }
    } else {
      None
    }
  }
</pre>

And... it works!

好了...现在没问题了！

h2. Packaging and Publishing

At this point we can run the package command to generate a jar file.  However we may want to share our jar with other teams.  To do this we'll build on StandardProject, which gives us a big head start.

The first step is include StandardProject as an SBT plugin.  Plugins are a way to introduce dependencies to your build, rather than your project.  These dependencies are defined in project/plugins/Plugins.scala.  Add the following to the Plugins.scala file.

## 打包和发布

现在，我们可以运行package命令来生成一个jar文件了。不过，我们可能需要和其他的团队分享我们的jar文件。要达到这个目的，我们需要基于StandardProject来构建，这个需要从头开始，并且过程有点复杂。

第一步是需要把StandardProject作为一个SBT插件添加进行来。插件是一种引入依赖的方式，只不过它是针对于你的构建而不是项目。这些依赖都定义在project/plugins/Plugins.scala里。把下面的内容添加到Plugins.scala文件。

<pre class="brush: java; gutter: true">
import sbt._

class Plugins(info: ProjectInfo) extends PluginDefinition(info) {
  val twitterMaven = "twitter.com" at "http://maven.twttr.com/"
  val defaultProject = "com.twitter" % "standard-project" % "0.7.14"
}
</pre>

Note that we've specified a maven repository as well as a dependency.  That's because the standard project library is hosted by us, which isn't one of the default repos sbt checks.

We'll also update our project definition to extend StandardProject, include an SVN publishing trait, and define the repository we wish to publish to.  Alter SampleProject.scala to the following

注意我们把一个maven仓库也作为依赖添加进来。这是因为这个标准项目库是我们维护的，而不是在sbt的默认仓库里。

同时，我们也需要更新我们的工程定义，让他扩展StandProject类，还需要扩展一个SubversionPublisher trait，同时也需要定义我们打算发布的仓库。把SampleProject.scala修改成如下所示：

<pre class="brush: java; gutter: true">
import sbt._
import com.twitter.sbt._

class SampleProject(info: ProjectInfo) extends StandardProject(info) with SubversionPublisher {
  val jackson = "org.codehaus.jackson" % "jackson-core-asl" % "1.6.1"
  val specs = "org.scala-tools.testing" % "specs_2.8.0" % "1.6.5" % "test"

  override def subversionRepository = Some("http://svn.local.twitter.com/maven/")
}
</pre>

Now if we run the publish action we'll see the following

现在我们来执行publish的动作，会看到下面的结果

<pre class="brush: java; gutter: true">
[info] == deliver ==
IvySvn Build-Version: null
IvySvn Build-DateTime: null
[info] :: delivering :: com.twitter#sample;1.0-SNAPSHOT :: 1.0-SNAPSHOT :: release :: Wed Nov 24 10:26:45 PST 2010
[info] 	delivering ivy file to /Users/mmcbride/projects/sbt-sample/target/ivy-1.0-SNAPSHOT.xml
[info] == deliver ==
[info] 
[info] == make-pom ==
[info] Wrote /Users/mmcbride/projects/sbt-sample/target/sample-1.0-SNAPSHOT.pom
[info] == make-pom ==
[info] 
[info] == publish ==
[info] :: publishing :: com.twitter#sample
[info] Scheduling publish to http://svn.local.twitter.com/maven/com/twitter/sample/1.0-SNAPSHOT/sample-1.0-SNAPSHOT.jar
[info] 	published sample to com/twitter/sample/1.0-SNAPSHOT/sample-1.0-SNAPSHOT.jar
[info] Scheduling publish to http://svn.local.twitter.com/maven/com/twitter/sample/1.0-SNAPSHOT/sample-1.0-SNAPSHOT.pom
[info] 	published sample to com/twitter/sample/1.0-SNAPSHOT/sample-1.0-SNAPSHOT.pom
[info] Scheduling publish to http://svn.local.twitter.com/maven/com/twitter/sample/1.0-SNAPSHOT/ivy-1.0-SNAPSHOT.xml
[info] 	published ivy to com/twitter/sample/1.0-SNAPSHOT/ivy-1.0-SNAPSHOT.xml
[info] Binary diff deleting com/twitter/sample/1.0-SNAPSHOT
[info] Commit finished r977 by 'mmcbride' at Wed Nov 24 10:26:47 PST 2010
[info] Copying from com/twitter/sample/.upload to com/twitter/sample/1.0-SNAPSHOT
[info] Binary diff finished : r978 by 'mmcbride' at Wed Nov 24 10:26:47 PST 2010
[info] == publish ==
[success] Successful.
[info] 
[info] Total time: 4 s, completed Nov 24, 2010 10:26:47 AM
</pre>

And (after some time), we can go to binaries.local.twitter.com:http://binaries.local.twitter.com/maven/com/twitter/sample/1.0-SNAPSHOT/ to see our published jar.

然后（过一段时间），我们可以在binaries.local.twitter.com:http://binaries.local.twitter.com/maven/com/twitter/sample/1.0-SNAPSHOT/下面看到我们发布的jar包。

h2. Adding Tasks

Tasks are Scala functions.  The simplest way to add a task is to include a val in your project definition using the task method, e.g.

## 添加任务

任务都是Scala函数。添加任务最简单的方式是包含一个通过task方法定义的常量，例如：


<pre class="brush: java; gutter: true">
lazy val print = task {log.info("a test action"); None}
</pre>

If you want dependencies and a description you can add them like this

如果你要添加依赖，同时添加一个描述，你可以按照下面的方式来添加：

<pre class="brush: java; gutter: true">
lazy val print = task {log.info("a test action"); None}.dependsOn(compile) describedAs("prints a line after compile")
</pre>

If we reload our project and run the print action we'll see the following

如果我们重新加载工程，然后运行print动作，就会看到下面的内容：

<pre class="brush: java; gutter: true">
> print
[info] 
[info] == print ==
[info] a test action
[info] == print ==
[success] Successful.
[info] 
[info] Total time: 0 s, completed Nov 24, 2010 11:05:12 AM
> 
</pre>

So it works.  If you're defining a task in a single project this works just fine.  However if you're defining this in a plugin it's fairly inflexible.  I may want to

这样确实可以的。如果你在一个单一的工程里创建这样的任务是没问题的。但是如果你在插件里这样定义的话，是非常不灵活的。我可能这样做

<pre class="brush: java; gutter: true">
lazy val print = printAction
def printAction = printTask.dependsOn(compile) describedAs("prints a line after compile")
def printTask = task {log.info("a test action"); None}
</pre>

This allows consumers to override the task itself, the dependencies and/or description of the task, or the action.  Most built in SBT actions follow this pattern.  As an example, we can modify the builtin package task to print the current timestamp by doing the following

这样使得用户可以自己重写任务，依赖，任务的描述，或是任务的行为。SBT大部分的内置行为都是这种模式的。作为范例，我们可以修改内置的package任务，让他在做下面的任务时打印出时间戳。

<pre class="brush: java; gutter: true">
lazy val printTimestamp = task { log.info("current time is " + System.currentTimeMillis); None}
override def packageAction = super.packageAction.dependsOn(printTimestamp)
</pre>

There are many examples in StandardProject of tweaking SBT defaults and adding custom tasks.

在StandarProject里有很多对于SBT默认配置的调整和自定义任务的示例。

h2. Quick Reference

## 速查手册

h3. Common Commands

* actions - show actions available for this project
* update - downloads dependencies
* compile - compiles source
* test - runs tests
* package - creates a publishable jar file
* publish-local - installs the built jar in your local ivy cache
* publish - pushes your jar to a remote repo (if configured)

### 常用命令
* actions - 显示对当前工程可用的命令
* update - 下载依赖
* compile - 编译代码
* test - 运行测试代码
* package - 创建一个可发布的jar包
* publish-local - 把构建出来的jar包安装到本地的ivy缓存
* publish - 把jar包发布到远程仓库（如果配置了的话)

h3. Moar Commands

* test-failed - run any specs that failed
* test-quick - run any specs that failed and/or had dependencies updated
* clean-cache - remove all sorts of sbt cached stuff.  Like clean for sbt
* clean-lib - remove everything in lib_managed

### 更多命令
* test-failed -  运行失败的spec
* test-quick - 运行所有失败的以及/或者是由依赖更新的spec
* clean-cache - 清除所有的sbt缓存。类似于sbt的clean命令
* clean-lib - 删除lib_managed下的所有内容


h3. Project Layout

TBD
### 工程结构
未完待续