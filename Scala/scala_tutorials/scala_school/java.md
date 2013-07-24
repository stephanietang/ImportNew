è‹±æ–‡åŸæ–‡: [Scala School](http://twitter.github.io/scala_school/java.html)ï¼Œç¿»è¯‘ï¼š[ImportNew](http://www.importnew.com) - [æœ±ä¼Ÿæ°](http://www.importnew.com/author/zhuweijie)

This lesson covers Java interoperability.

* Javap
* Classes
* Exceptions
* Traits
* Objects
* Closures and Functions
* Variance

è¿™ä¸ªç« èŠ‚ä¸»è¦è®²è§£å’ŒJavaè¿›è¡Œäº’æ“ä½œã€‚

* Javap
* Classes
* Excepitons
* Traits
* Objects
* Closureså’ŒFunctions
* Variance

h2. Javap

javap is a tool that ships with the JDK.  Not the JRE.  There's a difference.  Javap decompiles class definitions and shows you what's inside.  Usage is pretty simple

## Javap

<<<<<<< HEAD
javapæ˜¯JDKé™„å¸¦çš„ä¸€ä¸ªå·¥å…·ï¼Œè€Œä¸åŒ…å«åœ¨JREä¸­ã€‚Javapåç¼–è¯‘classæ–‡ä»¶ï¼Œå¹¶ä¸”å‘ä½ å±•ç¤ºå®ƒçš„å†…å®¹ã€‚ç”¨èµ·æ¥å¾ˆç®€å•ã€‚
=======
javapÊÇJDK¸½´øµÄÒ»¸ö¹¤¾ß£¬¶ø²»ÊÇJRE¡£ËüÃÇÖ®¼ä»¹ÊÇÓĞ²î±ğµÄ¡£Javap·´±àÒëclassÎÄ¼ş£¬²¢ÇÒÏòÄãÕ¹Ê¾ËüÀïÃæ·ÅµÄÊÇÊ²Ã´¡£ÓÃÆğÀ´ºÜ¼òµ¥¡£
>>>>>>> 5741bfc6b00f7bc8c7f4f5a9da27bf330d39cda2

<pre class="brush: java; gutter: true">
[local ~/projects/interop/target/scala_2.8.1/classes/com/twitter/interop]$ javap MyTrait
Compiled from "Scalaisms.scala"
public interface com.twitter.interop.MyTrait extends scala.ScalaObject{
    public abstract java.lang.String traitName();
    public abstract java.lang.String upperTraitName();
}
</pre>

If you're hardcore you can look at byte code

å¦‚æœä½ æƒ³äº†è§£åº•å±‚çš„è¯ï¼Œä½ å¯ä»¥æŸ¥çœ‹å¯¹åº”çš„å­—èŠ‚ç 

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

å¦‚æœä½ åœ¨Javaå¹³å°ä¸Šæœ‰ä»€ä¹ˆé—®é¢˜ï¼Œä½ å¯ä»¥é€šè¿‡javapæ¥æ’æŸ¥ã€‚

h2. Classes

The four major items to consider when using a Scala _class_ from Java are

* Class parameters
* Class vals
* Class vars
* Exceptions

We'll construct a simple scala class to show the full range of entities

## ç±»

ä»Javaçš„è§’åº¦æ¥ä½¿ç”¨Scalaçš„\_class\_éœ€è¦æ³¨æ„çš„å››ä¸ªè¦ç‚¹å¦‚ä¸‹ï¼š

* ç±»å‚æ•°
* ç±»å¸¸é‡
* ç±»å˜é‡
* å¼‚å¸¸

æˆ‘ä»¬æ¥åˆ›å»ºä¸€ä¸ªç®€å•çš„scalaç±»æ¥å±•ç¤ºè¿™å‡ ä¸ªè¦ç‚¹

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

### ç±»å‚æ•°

* é»˜è®¤æƒ…å†µä¸‹ï¼Œç±»å‚æ•°å®é™…ä¸Šå°±æ˜¯Javaé‡Œæ„é€ å‡½æ•°çš„å‚æ•°ã€‚è¿™å°±æ„å‘³ç€ä½ ä¸èƒ½åœ¨è¿™ä¸ªclassä¹‹å¤–è®¿é—®å®ƒä»¬ã€‚
* æŠŠç±»å‚æ•°å®šä¹‰æˆä¸€ä¸ªval/varçš„æ–¹å¼å’Œä¸‹é¢çš„ä»£ç ç›¸åŒ

<pre class="brush: java; gutter: true">
class SimpleClass(acc_: String) {
  val acc = acc_
}
</pre>
which makes it accessible from Java code just like other vals

ä¸‹é¢å°±å¯ä»¥é€šè¿‡Javaä»£ç æ¥è®¿é—®å®ƒä»¬äº†ã€‚

h3. Vals

* vals get a method defined for access from Java.  You can access the value of the val "foo" via the method "foo()"

### å¸¸é‡ï¼ˆValï¼‰

* å¸¸é‡ï¼ˆvalï¼‰éƒ½ä¼šå®šä¹‰æœ‰å¯¹åº”çš„ä¾›Javaä»£ç è®¿é—®çš„æ–¹æ³•ã€‚ä½ å¯ä»¥é€šè¿‡"foo()"æ–¹æ³•æ¥å–å¾—å¸¸é‡ï¼ˆvalï¼‰â€œfooâ€çš„å€¼ã€‚

h3. Vars

* vars get a method <name>_$eq defined.  You can call it like so

### å˜é‡ï¼ˆVarï¼‰

* å˜é‡ï¼ˆvarï¼‰ä¼šå¤šå®šä¹‰ä¸€ä¸ª<name>\_$eqæ–¹æ³•ã€‚ä½ å¯ä»¥è¿™æ ·è°ƒç”¨æ¥è®¾ç½®å˜é‡çš„å€¼ï¼š

<pre class="brush: java; gutter: true">
foo$_eq("newfoo");
</pre>

h3. BeanProperty

You can annotate vals and vars with the @BeanProperty annotation.  This generates getters/setters that look like POJO getter/setter definitions.  If you want the isFoo variant, use the BooleanBeanProperty annotation.  The ugly foo$_eq becomes

### BeanFactory

ä½ å¯ä»¥é€šè¿‡@BeanPropertyæ³¨è§£æ¥æ ‡æ³¨valå’Œvarã€‚è¿™æ ·å°±ä¼šç”Ÿæˆç±»ä¼¼äºPOJOçš„getter/setteræ–¹æ³•ã€‚å‡å¦‚ä½ æƒ³è¦è®¿é—®isFooå˜é‡ï¼Œä½¿ç”¨
BooleanBeanPropertyæ³¨è§£ã€‚é‚£ä¹ˆéš¾ä»¥ç†è§£çš„foo$eqå°±å¯ä»¥æ¢æˆï¼š

<pre class="brush: java; gutter: true">
setFoo("newfoo");
getFoo();
</pre>


h3. Exceptions

Scala doesn't have checked exceptions.  Java does.  This is a philosophical debate we won't get into, but it *does* matter when you want to catch an exception in Java.  The definitions of dangerFoo and dangerBar demonstrate this.  In Java I can't do this

### å¼‚å¸¸

Scalaé‡Œæ²¡æœ‰å—æ£€å¼‚å¸¸ï¼ˆchecked exceptionï¼‰,ä½†æ˜¯Javaé‡Œæœ‰ã€‚è¿™æ˜¯ä¸€ä¸ªè¯­è¨€å±‚é¢ä¸Šçš„é—®é¢˜ï¼Œæˆ‘ä»¬è¿™é‡Œä¸è¿›è¡Œè®¨è®ºï¼Œä½†æ˜¯åœ¨Javaé‡Œä½ å¯¹å¼‚å¸¸è¿›è¡Œæ•è·çš„æ—¶å€™ä½ è¿˜æ˜¯è¦æ³¨æ„çš„ã€‚dangerFooå’ŒdangerBarçš„å®šä¹‰é‡Œå¯¹è¿™è¿›è¡Œäº†ç¤ºèŒƒã€‚åœ¨Javaé‡Œï¼Œä½ ä¸èƒ½è¿™æ ·åš

<pre class="brush: java; gutter: true">
        // exception erasure!
        // å¼‚å¸¸æ“¦é™¤ï¼
        try {
            s.dangerFoo();
        } catch (IOException e) {
            // UGLY
            // éå¸¸ä¸‘é™‹
        }

</pre>

Java complains that the body of s.dangerFoo never throws IOException.  We can hack around this by catching Throwable, but that's lame.

Javaç¼–è¯‘å™¨ä¼šå› ä¸ºs.dangerFooä¸ä¼šæŠ›å‡ºIOExceptionè€ŒæŠ¥é”™ã€‚æˆ‘ä»¬å¯ä»¥é€šè¿‡catch Throwableæ¥ç»•è¿‡è¿™ä¸ªé”™è¯¯ï¼Œä½†æ˜¯è¿™æ ·åšæ²¡å¤šå¤§ç”¨å¤„ã€‚

Instead, as a good Scala citizen it's a decent idea to use the throws annotation like we did on dangerBar.  This allows us to continue using checked exceptions in Java land.

ä¸è¿‡ï¼Œä½œä¸ºä¸€ä¸ªScalaç”¨æˆ·ï¼Œæ¯”è¾ƒæ­£å¼çš„æ–¹å¼æ˜¯ä½¿ç”¨throwsæ³¨è§£ï¼Œå°±åƒæˆ‘ä»¬ä¹‹å‰åœ¨dangerBarä¸Šçš„ä¸€æ ·ã€‚è¿™ä¸ªæ‰‹æ®µä½¿å¾—æˆ‘ä»¬èƒ½å¤Ÿä½¿ç”¨Javaé‡Œçš„å—æ£€å¼‚å¸¸ï¼ˆchecked exceptionï¼‰ã€‚

h3. Further Reading

A full list of Scala annotations for supporting Java interop can be found here http://www.scala-lang.org/node/106.

### å»¶ä¼¸é˜…è¯»

æ”¯æŒJavaäº’æ“ä½œçš„æ³¨è§£çš„å®Œæ•´åˆ—è¡¨åœ¨[http://www.scala-lang.org/node/106](http://www.scala-lang.org/node/106)ã€‚

h2. Traits

How do you get an interface + implementation?  Let's take a simple trait definition and look

## Trait

æˆ‘ä»¬æ€æ ·å¯ä»¥å¾—åˆ°ä¸€ä¸ªæ¥å£å’Œå¯¹åº”çš„å®ç°å‘¢ï¼Ÿæˆ‘ä»¬ç®€å•çœ‹çœ‹traitçš„å®šä¹‰


<pre class="brush: java; gutter: true">
trait MyTrait {
  def traitName:String
  def upperTraitName = traitName.toUpperCase
}
</pre>

This trait has one abstract method (traitName) and one implemented method (upperTraitName).  What does Scala generate for us?  An interface named MyTrait, and a companion implementation named MyTrait$class.

è¿™ä¸ªtraitæœ‰ä¸€ä¸ªæŠ½è±¡çš„æ–¹æ³•ï¼ˆtraitNameï¼‰å’Œä¸€ä¸ªå·²å®ç°çš„æ–¹æ³•ï¼ˆupperTraitNameï¼‰ã€‚å¯¹äºè¿™æ ·çš„traitï¼ŒScalaä¼šç”Ÿæˆä»€ä¹ˆæ ·çš„ä»£ç å‘¢ï¼Ÿå®ƒä¼šç”Ÿæˆä¸€ä¸ªMyTraitæ¥å£ï¼ŒåŒæ—¶è¿˜ä¼šç”Ÿæˆå¯¹åº”çš„å®ç°ç±»MyTrait$classã€‚

The implementation of MyTrait is what you'd expect

MyTraitçš„å®ç°å’Œä½ çŒœæƒ³çš„å·®ä¸å¤šï¼š

<pre class="brush: java; gutter: true">
[local ~/projects/interop/target/scala_2.8.1/classes/com/twitter/interop]$ javap MyTrait
Compiled from "Scalaisms.scala"
public interface com.twitter.interop.MyTrait extends scala.ScalaObject{
    public abstract java.lang.String traitName();
    public abstract java.lang.String upperTraitName();
}
</pre>

The implementation of MyTrait$class is more interesting

ä¸è¿‡MyTrait$classçš„å®ç°æ›´åŠ æœ‰è¶£ï¼š

<pre class="brush: java; gutter: true">
[local ~/projects/interop/target/scala_2.8.1/classes/com/twitter/interop]$ javap MyTrait\$class
Compiled from "Scalaisms.scala"
public abstract class com.twitter.interop.MyTrait$class extends java.lang.Object{
    public static java.lang.String upperTraitName(com.twitter.interop.MyTrait);
    public static void $init$(com.twitter.interop.MyTrait);
}
</pre>

MyTrait$class has only static methods that take an instance of MyTrait.  This gives us a clue as to how to extend a Trait in Java.

MyTrait$classç±»åªæœ‰ä¸€ä¸ªé™æ€çš„æ¥å—ä¸€ä¸ªMyTraitå®ä¾‹ä½œä¸ºå‚æ•°çš„æ–¹æ³•ã€‚è¿™æ ·ç»™äº†æˆ‘ä»¬åœ¨Javaå¦‚ä½•å®ç°traitçš„ä¸€æ¡çº¿ç´¢ã€‚

Our first try is the following

é¦–å…ˆè¦åšçš„æ˜¯ï¼š

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

ç„¶åæˆ‘ä»¬ä¼šå¾—åˆ°ä¸‹é¢çš„é”™è¯¯ï¼š

<pre class="brush: java; gutter: true">
[info] Compiling main sources...
[error] /Users/mmcbride/projects/interop/src/main/java/com/twitter/interop/JTraitImpl.java:3: com.twitter.interop.JTraitImpl is not abstract and does not override abstract method upperTraitName() in com.twitter.interop.MyTrait
[error] public class JTraitImpl implements MyTrait {
[error]        ^
</pre>

We _could_ just implement this ourselves.  But there's a sneakier way.

æˆ‘ä»¬_å¯ä»¥_è‡ªå·±æ¥å®ç°ä»–ä»¬ã€‚ä¸è¿‡è¿˜æœ‰ä¸€ç§æ›´åŠ è¯¡å¼‚çš„æ–¹å¼ã€‚

<pre class="brush: java; gutter: true">
package com.twitter.interop;

    public String upperTraitName() {
        return MyTrait$class.upperTraitName(this);
    }
</pre>

We can just delegate this call to the generated Scala implementation.  We can also override it if we want.

æˆ‘ä»¬åªéœ€è¦æŠŠç›¸åº”çš„æ–¹æ³•è°ƒç”¨ä»£ç†åˆ°Scalaçš„å®ç°ä¸Šã€‚å¹¶ä¸”æˆ‘ä»¬è¿˜å¯ä»¥æ ¹æ®å®é™…éœ€è¦è¿›è¡Œé‡å†™ã€‚


h2.  Objects

Objects are the way Scala implements static methods/singletons.  Using them from Java is a bit odd.  There isn't a stylistically perfect way to use them, but in Scala 2.8 it's not terrible

A Scala object is compiled to a class that has a trailing "$".  Let's set up a class and a companion object

## å¯¹è±¡ï¼ˆObjectï¼‰

åœ¨Scalaé‡Œï¼Œå¯¹è±¡æ˜¯ç”¨æ¥å®ç°é™æ€æ–¹æ³•å’Œå•ä¾‹æ¨¡å¼çš„ã€‚åœ¨Javaé‡Œä½¿ç”¨å®ƒä»¬å°±æ˜¾å¾—æ¯”è¾ƒæ€ªã€‚åœ¨è¯­æ³•ä¸Šæ²¡æœ‰ä»€ä¹ˆæ¯”è¾ƒä¼˜é›…çš„æ–¹å¼æ¥ä½¿ç”¨å®ƒä»¬ï¼Œä½†æ˜¯åœ¨Scala 2.8é‡Œå°±æ²¡æœ‰é‚£ä¹ˆéº»çƒ¦äº†ã€‚

Scalaå¯¹è±¡ä¼šè¢«ç¼–è¯‘æˆä¸€ä¸ªåç§°å¸¦æœ‰â€œ$â€åç¼€çš„ç±»ã€‚æˆ‘ä»¬æ¥åˆ›å»ºä¸€ä¸ªç±»ä»¥åŠå¯¹åº”çš„å¯¹è±¡ï¼ˆObjectï¼‰ã€‚æˆ‘ä»¬æ¥åˆ›å»ºä¸€ä¸ªç±»ä»¥åŠå¯¹åº”çš„ä¼´ç”Ÿå¯¹è±¡ï¼ˆcompanion object)ã€‚

<pre class="brush: java; gutter: true">
class TraitImpl(name: String) extends MyTrait {
  def traitName = name
}

object TraitImpl {
  def apply = new TraitImpl("foo")
  def apply(name: String) = new TraitImpl(name)
}
</pre>

<<<<<<< HEAD
We can naèŒ‚vely access this in Java like so
=======
We can naively access this in Java like so

ÎÒÃÇ¿ÉÒÔÍ¨¹ıÏÂÃæÕâÖÖÆæÃîµÄ·½Ê½ÔÚJavaÀï·ÃÎÊËü£º
>>>>>>> 5741bfc6b00f7bc8c7f4f5a9da27bf330d39cda2

<pre class="brush: java; gutter: true">
MyTrait foo = TraitImpl$.MODULE$.apply("foo");
</pre>

Now you may be asking yourself, WTF?  This is a valid response.  Let's look at what's actually inside TraitImpl$

ÏÖÔÚÄãÒ²Ğí»áÎÊ×Ô¼º£¬Õâ¾¿¾¹ÊÇÉñÂí£¿ÕâÊÇÒ»¸öºÜÕı³£µÄ·´Ó¦¡£ÎÒÃÇÏÖÔÚÒ»ÆğÀ´¿´¿´TraintImpl$ÄÚ²¿¾¿¾¹ÊÇÔõÃ´ÊµÏÖµÄ¡£

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

ÆäÊµËüÀïÃæÃ»ÓĞÈÎºÎ¾²Ì¬·½·¨¡£Ïà·´£¬Ëü»¹ÓĞÒ»¸ö¾²Ì¬³ÉÔ±½Ğ×öMODULE$¡£Êµ¼ÊÉÏ·½·¨µÄµ÷ÓÃ¶¼ÊÇ´úÀíµ½Õâ¸ö³ÉÔ±±äÁ¿ÉÏµÄ¡£ÕâÖÖÊµÏÖÊ¹µÃ·ÃÎÊÆğÀ´¾õµÃ±È½Ï¶ñĞÄ£¬µ«ÊÇÈç¹ûÄãÖªµÀÔõÃ´Ê¹ÓÃMODULE$µÄ»°£¬ÆäÊµ»¹ÊÇºÜÊµÓÃµÄ¡£

h3.  Forwarding Methods

In Scala 2.8 dealing with Objects got quite a bit easier.  If you have a class with a companion object, the 2.8 compiler generates forwarding methods on the companion class.  So if you built with 2.8, you can access methods in the TraitImpl Object like so

### ×ª·¢·½·¨£¨Forwarding Method£©

ÔÚScala 2.8Àï£¬´¦ÀíObject»á±È½Ï¼òµ¥µã¡£Èç¹ûÄãÓĞÒ»¸öÀàÒÔ¼°¶ÔÓ¦µÄ°éÉú¶ÔÏó£¨companion object£©£¬2.8 µÄ±àÒëÆ÷»áÔÚ°éÉú¶ÔÏóÀïÉú³É×ª·¢·½·¨¡£Èç¹ûÊ¹ÓÃ2.8µÄ±àÒëÆ÷½øĞĞ¹¹½¨£¬ÄÇÃ´Äã¿ÉÒÔÍ¨¹ıÏÂÃæµÄ·½·¨À´·ÃÎÊTraitImpl¶ÔÏó£º

<pre class="brush: java; gutter: true">
MyTrait foo = TraitImpl.apply("foo");
</pre>

h2. Closures Functions

One of Scala's most important features is the treatment of functions as first class citizens.  Let's define a class that defines some methods that take functions as arguments.

## ±Õ°üº¯Êı

Scala×îÖØÒªµÄÒ»¸öÌØµã¾ÍÊÇ°Ñº¯Êı×÷ÎªÒ»µÈ¹«Ãñ¡£ÎÒÃÇÀ´¶¨ÒåÒ»¸öÀà£¬ËüÀïÃæ°üº¬Ò»Ğ©½ÓÊÕº¯Êı×÷Îª²ÎÊıµÄ·½·¨¡£

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

ÔÚScalaÀïÎÒ¿ÉÒÔÕâÑùµ÷ÓÃ£º

<pre class="brush: java; gutter: true">
val cc = new ClosureClass
cc.printResult { "HI MOM" }
</pre>

In Java it's not so easy, but it's not terrible either.  Let's see what ClosureClass actually compiled to:

µ«ÊÇÔÚJavaÀïÈ´Ã»ÓĞÕâÃ´¼òµ¥£¬²»¹ıÒ²Ã»ÓĞÏëÏóµÄÄÇÃ´¸´ÔÓ¡£ÎÒÃÇÀ´¿´¿´ClosureClass×îÖÕµ½µ×±àÒë³ÉÔõÑù£º

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

Õâ¸ö¿´ÆğÀ´Ò²²»ÊÇºÜ¿ÉÅÂ¡£"f: => T" ×ª»»³É"Function0"£¬"f: String => T" ×ª»»³É "Function1"¡£Scala¶¨ÒåÁË´ÓFunction0µ½Function22£¬Ò»Ö±Ö§³Öµ½22¸ö²ÎÊı¡£ÕâÃ´¶àÈ·ÊµÒÑ¾­×ã¹»ÁË¡£

ÏÖÔÚÎÒÃÇÖ»ĞèÒªÅªÇå³ş£¬ÔõÃ´ÔÚJavaÈ¥ÊµÏÖÕâ¸ö¹¦ÄÜ¡£ÊÂÊµÉÏ£¬ScalaÌá¹©ÁËAbstractFunction0ºÍAbstractFunction1£¬ÎÒÃÇ¿ÉÒÔÕâÑùÀ´´«²Î£º


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

×¢ÒâÎÒÃÇ»¹¿ÉÒÔÊ¹ÓÃ·ºĞÍÀ´²ÎÊı»¯²ÎÊıµÄÀàĞÍ¡£

