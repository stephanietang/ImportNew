How ClassLoader Works in Java
 
Java class loaders are used to load classes at runtime. ClassLoader in Java works on three principle: delegation, visibility and uniqueness. Delegation principle forward request of class loading to parent class loader and only loads the class, if parent is not able to find or load class. Visibility principle allows child class loader to see all the classes loaded by parent ClassLoader, but parent class loader can not see classes loaded by child. Uniqueness principle allows to load a class exactly once, which is basically achieved by delegation and ensures that child ClassLoader doesn't reload the class already loaded by parent. Correct understanding of class loader is must to resolve issues like NoClassDefFoundError in Java and java.lang.ClassNotFoundException, which are related to class loading. ClassLoader is also an important topic in advanced Java Interviews, where good knowledge of working of Java ClassLoader and How classpath works in Java  is expected from Java programmer. I have always seen questions like, Can one class be loaded by two different ClassLoader in Java on various Java Interviews.  In this Java programming tutorial, we will learn what is ClassLoader in Java, How ClassLoader works in Java and some specifics about Java ClassLoader.

Java类加载器的作用就是在运行时加载类。Java类加载器基于三个机制：委托、可见性和单一性。委托机制是指将加载一个类的请求交给父类加载器，如果这个父类加载器不能够找到或者加载这个类，那么再加载它。可见性的原理是子类的加载器可以看见所有的父类加载器加载的类，而父类加载器看不到子类加载器加载的类。单一性原理是指仅加载一个类一次，这是由委托机制确保子类加载器不会再次加载父类加载器加载过的类。正确理解类加载器能够帮你解决NoClassDefFoundError和java.lang.ClassNotFoundException，因为它们和类的加载相关。类加载器通常也是比较高级的Java面试中的重要考题，Java类加载器和工作原理以及classpath如何运作的经常被问到。Java面试题中也经常出现“一个类是否能被两个不同类加载器加载”这样的问题。这篇教程中，我们将学到类加载器是什么，它的工作原理以及一些关于类加载器的知识点。


What is ClassLoader in Java

ClassLoader in Java is a class which is used to load class files in Java. Java code is compiled into class file by javac compiler and JVM executes Java program, by executing byte codes written in class file. ClassLoader is responsible for loading class files from file system, network or any other source. There are three default class loader used in Java, Bootstrap , Extension and System or Application class loader. Every class loader has a predefined location, from where they loads class files. Bootstrap ClassLoader is responsible for loading standard JDK class files from rt.jar and it is parent of all class loaders in Java. Bootstrap class loader don't have any parents, if you call String.class.getClassLoader() it will return null and any code based on that may throw NullPointerException in Java. Bootstrap class loader is also known as Primordial ClassLoader in Java.  Extension ClassLoader delegates class loading request to its parent, Bootstrap and if unsuccessful, loads class form jre/lib/ext directory or any other directory pointed by java.ext.dirs system property. Extension ClassLoader in JVM is implemented by  sun.misc.Launcher$ExtClassLoader. Third default class loader used by JVM to load Java classes is called System or Application class loader and it is responsible for loading application specific classes from CLASSPATH environment variable, -classpath or -cp command line option, Class-Path attribute of Manifest file inside JAR. Application class loader is a child of Extension ClassLoader and its implemented by sun.misc.Launcher$AppClassLoader class. Also, except Bootstrap class loader, which is implemented in native language mostly in C,  all  Java class loaders are implemented using java.lang.ClassLoader.

什么是类加载器

类加载器是一个用来加载类文件的类。Java源代码通过javac编译器编译成类文件。然后JVM来执行类文件中的字节码来执行程序。类加载器负责加载文件系统、网络或其他来源的类文件。有三种默认使用的类加载器：Bootstrap类加载器、Extension类加载器和System类加载器（或者叫作Application类加载器）。每种类加载器都有设定好从哪里加载类。

- Bootstrap类加载器负责加载rt.jar中的JDK类文件，它是所有类加载器的父加载器。Bootstrap类加载器没有任何父类加载器，如果你调用String.class.getClassLoader()，会返回null，任何基于此的代码会抛出NUllPointerException异常。Bootstrap加载器被称为初始类加载器。
- 而Extension将加载类的请求先委托给它的父加载器，也就是Bootstrap，如果没有成功加载的话，再从jre/lib/ext目录下或者java.ext.dirs系统属性定义的目录下加载类。Extension加载器由sun.misc.Launcher$ExtClassLoader实现。
- 第三种默认的加载器就是System类加载器（又叫作Application类加载器）了。它负责从classpath环境变量中加载某些应用相关的类，classpath环境变量通常由-classpath或-cp命令行选项来定义，或者是JAR中的Manifest的classpath属性。Application类加载器是Extension类加载器的子加载器。通过sun.misc.Launcher$AppClassLoader实现。

除了Bootstrap类加载器是大部分由C来写的，其他的类加载器都是通过java.lang.ClassLoader来实现的。

In short here is the location from which Bootstrap, Extension and Application ClassLoader load Class files.

总结一下，下面是三种类加载器加载类文件的地方：

1) Bootstrap类加载器 - JRE/lib/rt.jar

2) Extension类加载器 - JRE/lib/ext或者java.ext.dirs指向的目录

3) Application类加载器 - CLASSPATH环境变量, 由-classpath或-cp选项定义,或者是JAR中的Manifest的classpath属性定义.

How ClassLoader works in Java

What is ClassLoader in Java, How classloader works in JavaAs I explained earlier Java ClassLoader works in three principles : delegation, visibility and uniqueness. In this section we will see those rules in detail and understand working of Java ClassLoader with example. By the way here is a diagram which explains How ClassLoader load class in Java using delegation.

类加载器的工作原理

我之前已经提到过了，类加载器的工作原理基于三个机制：委托、可见性和单一性。这一节，我们来详细看看这些规则，并用一个实例来理解工作原理。下面显示的是类加载器使用委托机制的工作原理。



How class loader works in Java - class loading

Delegation principles

As discussed on when a class is loaded and initialized in Java, a class is loaded in Java, when its needed. Suppose you have an application specific class called Abc.class, first request of loading this class will come to Application ClassLoader which will delegate to its parent Extension ClassLoader which further delegates to Primordial or Bootstrap class loader. Primordial will look for that class in rt.jar and since that class is not there, request comes to Extension class loader which looks on jre/lib/ext directory and tries to locate this class there, if class is found there than Extension class loader will load that class and Application class loader will never load that class but if its not loaded by extension class-loader than Application class loader loads it from Classpath in Java. Remember Classpath is used to load class files while PATH is used to locate executable like javac or java command.

委托机制

当一个类加载和初始化的时候，类仅在有需要加载的时候被加载。假设你有一个应用需要的类叫作Abc.class，首先加载这个类的请求由Application类加载器委托给它的父类加载器Extension类加载器，然后再委托给Bootstrap类加载器。Bootstrap类加载器会先看看rt.jar中有没有这个类，因为并没有这个类，所以这个请求由回到Extension类加载器，它会查看jre/lib/ext目录下有没有这个类，如果这个类被Extension类加载器找到了，那么它将被加载，而Application类加载器不会加载这个类；而如果这个类没有被Extension类加载器找到，那么再由Application类加载器从classpath中寻找。记住classpath定义的是类文件的加载目录，而PATH是定义的是可执行程序如javac，java等的执行路径。

Visibility Principle

According to visibility principle, Child ClassLoader can see class loaded by Parent ClassLoader but vice-versa is not true. Which mean if class Abc is loaded by Application class loader than trying to load class ABC explicitly using extension ClassLoader will throw either java.lang.ClassNotFoundException. as shown in below Example

可见性机制

根据可见性机制，子类加载器可以看到父类加载器加载的类，而反之则不行。所以下面的例子中，当Abc.class已经被Application类加载器加载过了，然后如果想要使用Extension类加载器加载这个类，将会抛出java.lang.ClassNotFoundException异常。


<pre class="brush: java; gutter: true">
    package test;

    import java.util.logging.Level;
    import java.util.logging.Logger;

    /**
     * Java program to demonstrate How ClassLoader works in Java,
     * in particular about visibility principle of ClassLoader.
     *
     * @author Javin Paul
     */

    public class ClassLoaderTest {
      
        public static void main(String args[]) {
            try {          
                //printing ClassLoader of this class
                System.out.println("ClassLoaderTest.getClass().getClassLoader() : "
                                     + ClassLoaderTest.class.getClassLoader());

              
                //trying to explicitly load this class again using Extension class loader
                Class.forName("test.ClassLoaderTest", true 
                                ,  ClassLoaderTest.class.getClassLoader().getParent());
            } catch (ClassNotFoundException ex) {
                Logger.getLogger(ClassLoaderTest.class.getName()).log(Level.SEVERE, null, ex);
            }
        }

    }
</pre>


Output:

<pre class="brush: plain; gutter: true">
ClassLoaderTest.getClass().getClassLoader() : sun.misc.Launcher$AppClassLoader@601bb1
16/08/2012 2:43:48 AM test.ClassLoaderTest main
SEVERE: null
java.lang.ClassNotFoundException: test.ClassLoaderTest
        at java.net.URLClassLoader$1.run(URLClassLoader.java:202)
        at java.security.AccessController.doPrivileged(Native Method)
        at java.net.URLClassLoader.findClass(URLClassLoader.java:190)
        at sun.misc.Launcher$ExtClassLoader.findClass(Launcher.java:229)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:306)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:247)
        at java.lang.Class.forName0(Native Method)
        at java.lang.Class.forName(Class.java:247)
        at test.ClassLoaderTest.main(ClassLoaderTest.java:29)  
</pre>

Uniqueness Principle

According to this principle a class loaded by Parent should not be loaded by Child ClassLoader again. Though its completely possible to write class loader which violates Delegation and Uniqueness principles and loads class by itself, its not something which is beneficial. You should follow all  class loader principle while writing your own ClassLoader.

单一性机制

根据这个机制，父加载器加载过的类不能被子加载器加载第二次。虽然重写违反委托和单一性机制的类加载器是可能的，但这样做并不可取。你写自己的类加载器的时候应该严格遵守这三条机制。

How to load class explicitly in Java

Java provides API to explicitly load a class by Class.forName(classname) and Class.forName(classname, initialized, classloader), remember JDBC code which is used to load JDBC drives we have seen in Java program to Connect Oracle database. As shown in above example you can pass name of ClassLoader which should be used to load that particular class along with binary name of class. Class is loaded by calling loadClass() method of java.lang.ClassLoader class which calls findClass() method to locate bytecodes for corresponding class. In this example Extension ClassLoader uses java.net.URLClassLoader which search for class files and resources in JAR and directories. any search path which is ended using "/" is considered directory. If findClass() does not found the class than it throws java.lang.ClassNotFoundException and if it finds it calls defineClass() to convert bytecodes into a .class instance which is returned to the caller.

如何显式的加载类

Java提供了显式加载类的API：Class.forName(classname)和Class.forName(classname, initialized, classloader)。就像上面的例子中，你可以指定类加载器的名称以及要加载的类的名称。类的加载是通过调用java.lang.ClassLoader的loadClass()方法，而loadClass()方法则调用了findClass()方法来定位相应类的字节码。在这个例子中Extension类加载器使用了java.net.URLClassLoader，它从JAR和目录中进行查找类文件，所有以"/"结尾的查找路径被认为是目录。如果findClass()没有找到那么它会抛出java.lang.ClassNotFoundException异常，而如果找到的话则会调用defineClass()将字节码转化成类实例，然后返回。

Where to use ClassLoader in Java

ClassLoader in Java is a powerful concept and used at many places. One of the popular example of ClassLoader is AppletClassLoader which is used to load class by Applet, since Applets are mostly loaded from internet rather than local file system, By using separate ClassLoader you can also loads same class from multiple sources and they will be treated as different class in JVM. J2EE uses multiple class loaders to load class from different location like classes from WAR file will be loaded by Web-app ClassLoader while classes bundled in EJB-JAR is loaded by another class loader. Some web server also supports hot deploy functionality which is implemented using ClassLoader. You can also use ClassLoader to load classes from database or any other persistent store.

That's all about What is ClassLoader in Java and How ClassLoader works in Java. We have seen delegation, visibility and uniqueness principles which is quite important to debug or troubleshoot any ClassLoader related issues in Java. In summary knowledge of How ClassLoader works in Java is must for any Java developer or architect to design Java application and packaging.

什么地方使用类加载器

类加载器是个很强大的概念，很多地方被运用。最经典的例子就是AppletClassLoader，它被用来加载Applet使用的类，而Applets大部分是在网上使用，而非本地的操作系统使用。使用不同的类加载器，你可以从不同的源地址加载同一个类，它们被视为不同的类。J2EE使用多个类加载器加载不同地方的类，例如WAR文件由Web-app类加载器加载，而EJB-JAR中的类由另外的类加载器加载。有些服务器也支持热部署，这也由类加载器实现。你也可以使用类加载器来加载数据库或者其他持久层的数据。

以上是关于类加载器的工作原理。我们已经知道了委托、可见性以及单一性原理，这些对于调试类加载器相关问题时至关重要。这些对于Java程序员和架构师来说都是必不可少的知识。


http://javarevisited.blogspot.hk/2012/12/how-classloader-works-in-java.html