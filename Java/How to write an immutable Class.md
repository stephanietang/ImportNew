## 如何写一个不可变类？

不可变的对象指的是一旦创建之后，它的状态就不能改变。String类就是个不可变类，它的对象一旦创建之后，值就不能被改变了。

阅读更多: [为什么String类是不可变的](http://www.importnew.com/7440.html)

不可变对象对于缓存是非常好的选择，因为你不需要担心它的值会被更改。不可变类的另外一个好处是它自身是线程安全的，你不需要考虑多线程环境下的线程安全问题。

阅读更多: [Java线程教程](http://www.journaldev.com/1079/java-thread-tutorial)以及[Java多线程面试问题](http://www.journaldev.com/1162/java-multi-threading-concurrency-interview-questions-with-answers)

Here I am providing a way to create immutable class via an example for better understanding.

下面是创建不可变类的方法，我也给出了代码，加深理解。

要创建不可变类，要实现下面几个步骤：

1. 将类声明为final，所以它不能被继承
2. 将所有的成员声明为私有的，这样就不允许直接访问这些成员
3. 对变量不要提供setter方法
4. 将所有可变的成员声明为final，这样只能对它们赋值一次
5. 通过构造器初始化所有成员，进行深拷贝(deep copy)
6. 在getter方法中，不要直接返回对象本身，而是克隆对象，并返回对象的拷贝

为了理解第5和第6条，我将使用FinalClassExample来阐明。

FinalClassExample.java
<pre class="brush: java; gutter: true">
package com.journaldev.java;
 
import java.util.HashMap;
import java.util.Iterator;
 
public final class FinalClassExample {
 
    private final int id;
     
    private final String name;
     
    private final HashMap<String,String> testMap;
     
    public int getId() {
        return id;
    }
 
 
    public String getName() {
        return name;
    }
 
    /**
     * 可变对象的访问方法
     */
    public HashMap<String, String> getTestMap() {
        //return testMap;
        return (HashMap<String, String>) testMap.clone();
    }
 
    /**
     * 实现深拷贝(deep copy)的构造器
     * @param i
     * @param n
     * @param hm
     */
     
    public FinalClassExample(int i, String n, HashMap<String,String> hm){
        System.out.println("Performing Deep Copy for Object initialization");
        this.id=i;
        this.name=n;
        HashMap<String,String> tempMap=new HashMap<String,String>();
        String key;
        Iterator<String> it = hm.keySet().iterator();
        while(it.hasNext()){
            key=it.next();
            tempMap.put(key, hm.get(key));
        }
        this.testMap=tempMap;
    }
     
     
    /**
     * 实现浅拷贝(shallow copy)的构造器
     * @param i
     * @param n
     * @param hm
     */
    /**
    public FinalClassExample(int i, String n, HashMap<String,String> hm){
        System.out.println("Performing Shallow Copy for Object initialization");
        this.id=i;
        this.name=n;
        this.testMap=hm;
    }
    */
     
    /**
     * 测试浅拷贝的结果
     * 为了创建不可变类，要使用深拷贝
     * @param args
     */
    public static void main(String[] args) {
        HashMap<String, String> h1 = new HashMap<String,String>();
        h1.put("1", "first");
        h1.put("2", "second");
         
        String s = "original";
         
        int i=10;
         
        FinalClassExample ce = new FinalClassExample(i,s,h1);
         
        //Lets see whether its copy by field or reference
        System.out.println(s==ce.getName());
        System.out.println(h1 == ce.getTestMap());
        //print the ce values
        System.out.println("ce id:"+ce.getId());
        System.out.println("ce name:"+ce.getName());
        System.out.println("ce testMap:"+ce.getTestMap());
        //change the local variable values
        i=20;
        s="modified";
        h1.put("3", "third");
        //print the values again
        System.out.println("ce id after local variable change:"+ce.getId());
        System.out.println("ce name after local variable change:"+ce.getName());
        System.out.println("ce testMap after local variable change:"+ce.getTestMap());
         
        HashMap<String, String> hmTest = ce.getTestMap();
        hmTest.put("4", "new");
         
        System.out.println("ce testMap after changing variable from accessor methods:"+ce.getTestMap());
 
    }
 
}
</pre>

输出：

<pre class="brush: plain; gutter: true">
Performing Deep Copy for Object initialization
true
false
ce id:10
ce name:original
ce testMap:{2=second, 1=first}
ce id after local variable change:10
ce name after local variable change:original
ce testMap after local variable change:{2=second, 1=first}
ce testMap after changing variable from accessor methods:{2=second, 1=first}
</pre>

现在我们注释掉深拷贝的构造器，取消对浅拷贝构造器的注释。也对getTestMap()方法中的返回语句取消注释，返回实际的对象引用。然后再一次执行代码。

<pre class="brush: plain; gutter: true">
Performing Shallow Copy for Object initialization
true
true
ce id:10
ce name:original
ce testMap:{2=second, 1=first}
ce id after local variable change:10
ce name after local variable change:original
ce testMap after local variable change:{3=third, 2=second, 1=first}
ce testMap after changing variable from accessor methods:{3=third, 2=second, 1=first, 4=new}
</pre>

从输出可以看出，HashMap的值被更改了，因为构造器实现的是浅拷贝，而且在getter方法中返回的是原本对象的引用。

If I have missed something here, feel free to comment.
如果我漏掉了某些细节，请指出。

更多阅读: 如果不可变类有许多成员，而其中一些是可选的，那么我们可以[使用构造模式(builder pattern)来创建不可变类](http://www.journaldev.com/1432/how-to-create-immutable-class-in-java-using-builder-pattern)。

http://www.journaldev.com/129/how-to-write-an-immutable-class
