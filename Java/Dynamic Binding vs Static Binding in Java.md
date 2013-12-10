## 动态绑定 vs 静态绑定
 
### 动态绑定(又名后期绑定)

动态绑定是指编译器在编译阶段不知道要调用哪个方法，直到运行时才能确定。让我们用个例子来解释。譬如我们有一个叫作'SuperClass'的父类，还有一个继承它的子类'SubClass'。现在SuperClass引用也可以赋给SubClass类型的对象。如果SuperClass中有个someMethod()的方法，而子类也重写了这个方法，那么当调用SuperClass引用的这个方法的时候，编译器不知道该调用父类还是子类的方法，因为编译器不知道对象到底是什么类型，只有到运行时才知道这个引用指向什么对象。

<pre>
...
SuperClass superClass1 = new SuperClass();
SuperClass superClass2 = new SubClass();
...

superClass1.someMethod(); // SuperClass version is called
superClass2.someMethod(); // SubClass version is called
....
</pre>

我们可以看到虽然对象引用superClass1和superClass2都是SuperClass类型的，但是在运行时它们分别指向SuperClass和SubClass类型的对象。

所以在编译阶段，编译器不清楚调用引用的someMethod()到底是调用子类还是父类的该方法。

所以方法的动态绑定是基于实际的对象类型，而不是它们声明的对象引用类型。

### 静态绑定(又名前期绑定)


如果编译器可以在编译阶段就完成绑定，就叫作静态绑定或前期绑定。基本上实例方法都在运行时绑定，所有的静态方法都在编译时绑定，所以静态方法是静态绑定的。因为静态方法是属于类的方法，可以通过类名来访问(我们也应该使用类名来访问静态方法，而不是使用对象引用来访问)，所以要访问它们就必须在编译阶段就使用编译类型信息来进行绑定。这也就解释了[为什么静态方法实际上不能被重写](http://geekexplains.blogspot.hk/2008/06/can-you-override-static-methods-in-java.html)。

更多阅读：[可以重写静态方法吗？](http://geekexplains.blogspot.hk/2008/06/can-you-override-static-methods-in-java.html)

类似的，访问成员变量也是静态绑定的，因为Java不支持(实际上是不鼓励)成员变量的多态行为。下面看个例子：

<pre>
class SuperClass{
...
public String someVariable = "Some Variable in SuperClass";
...
}

class SubClass extends SuperClass{
...
public String someVariable = "Some Variable in SubClass";
...
}
...
...

SuperClass superClass1 = new SuperClass();
SuperClass superClass2 = new SubClass();

System.out.println(superClass1.someVariable);
System.out.println(superClass2.someVariable);
...

</pre>

输出：

<pre>
Some Variable in SuperClass
Some Variable in SuperClass
</pre>


我们可以发现成员变量由对象引用声明的类型决定，是由编译器在编译阶段就知道的信息，所以是静态绑定。另外一个静态绑定的例子是私有的方法，因为它们不会被继承，编译器在编译阶段就完成私有方法的绑定了。

http://geekexplains.blogspot.hk/2008/06/dynamic-binding-vs-static-binding-in.html
