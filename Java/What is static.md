## 什么是静态(static)?什么是静态方法，静态变量，静态块和静态类？
 
static是Java中的一个关键字，我们不能声明类或者包为静态的。static用于下面四种情况。

静态变量：我们可以将类级别的变量声明为static。静态变量是属于类的，而不是属于类创建的对象或实例。因为静态变量被类的所有实例共用，所以非线程安全的。通常静态变量还和关键字final一起用，作为所有对象共用的资源或常量。如果静态变量不是私有的，那么可以通过ClassName.variableName来访问它。

<pre>
//静态变量的例子
private static int count;
public static String str;
public static final String DB_USER = "myuser";
</pre>

静态方法：类似于静态变量，静态方法也属于类，不属于实例的。静态类只能访问类的静态变量，或调用类的静态方法。通常静态方法作为工具方法，被其它类使用，而不需要创建类的实例。譬如集合类、Wrapper类(String, Integer等)和工具类(java.util中的类)都有很多静态方法。通常java程序的开始就是一个main()方法，它就是个静态方法。

<pre>
//静态方法的例子
public static void setCount(int count) {
    if(count > 0)
    StaticExample.count = count;
}
 
//静态工具方法
public static int addInts(int i, int...js){
    int sum=i;
    for(int x : js) sum+=x;
    return sum;
}
</pre>

静态块：静态块就是类加载器加载对象时，要执行的一组语句。它用于初始化静态变量。通常用于类加载的时候创建静态资源。我们在静态块中不能访问非静态变量。我们可以在一个类中有多个静态块，尽管这么做没什么意义。静态块只会在类加载到内存中的时候执行一次。

<pre>
static{
    //在类被加载的时候用于初始化资源
    System.out.println("StaticExample static block");
    //仅能访问静态变量和静态方法
    str="Test";
    setCount(2);
}
</pre>

静态类：我们对嵌套类使用static关键字。static不能用于最外层的类。静态的嵌套类和其它外层的类别无二致，嵌套只是为了方便打包。
延伸阅读：[嵌套类](http://www.journaldev.com/996/java-nested-classes-java-inner-class-static-nested-class-local-inner-class-and-anonymous-inner-class)

让我们来看一个使用static关键字的例子：
 
StaticExample.java
<pre>
package com.journaldev.misc;
 
public class StaticExample {
 
    //静态块
    static{
        //在类被加载的时候用于初始化某些资源
        System.out.println("StaticExample static block");
        //仅能访问静态变量和静态方法
        str="Test";
        setCount(2);
    }
     
    //可以在一个类中有多个静态块
    static{
        System.out.println("StaticExample static block2");
    }
     
    //静态变量
    private static int count; //保持私有，仅能靠setter方法访问
    public static String str;
     
    public int getCount() {
        return count;
    }
 
    //静态方法
    public static void setCount(int count) {
        if(count > 0)
        StaticExample.count = count;
    }
     
    //静态工具方法
    public static int addInts(int i, int...js){
        int sum=i;
        for(int x : js) sum+=x;
        return sum;
    }
 
    //静态类的例子，方便打包之用
    public static class MyStaticClass{
        public int count;
         
    }
 
}
</pre>

让我们用一个测试程序来看看如何使用静态变量，静态方法以及静态类。

TestStatic.java
<pre>
package com.journaldev.misc;
 
public class TestStatic {
 
    public static void main(String[] args) {
        StaticExample.setCount(5);
         
        //非私有的静态变量可以通过类名来访问
        StaticExample.str = "abc";
        StaticExample se = new StaticExample();
        System.out.println(se.getCount());
        //类的静态变量和实例的静态变量是一样的
        System.out.println(StaticExample.str +" is same as "+se.str);
        System.out.println(StaticExample.str == se.str);
         
        //静态嵌套类和其他外层类一样
        StaticExample.MyStaticClass myStaticClass = new StaticExample.MyStaticClass();
        myStaticClass.count=10;
         
        StaticExample.MyStaticClass myStaticClass1 = new StaticExample.MyStaticClass();
        myStaticClass1.count=20;
         
        System.out.println(myStaticClass.count);
        System.out.println(myStaticClass1.count);
    }
     
}
</pre>

测试程序的输出：

<pre>
StaticExample static block
StaticExample static block2
5
abc is same as abc
true
10
20
</pre>

注意静态块仅在类加载到内存时执行了一次。

http://www.journaldev.com/1365/static-in-java-methods-variables-block-class
