## 可以重写静态方法吗？

### 问：你可以重写静态方法吗？

答：如果从重写方法会有什么特点来看，我们是不能重写静态方法的。虽然就算你重写静态方法，编译器也不会报错。也就是说，如果你试图重写静态方法，Java不会阻止你这么做，但你却得不到预期的结果（重写仅对非静态方法有用）。重写指的是根据运行时对象的类型来决定调用哪个方法，而不是根据编译时的类型。让我们猜一猜为什么静态方法是比较特殊的？因为它们是类的方法，所以它们在编译阶段就使用编译出来的类型进行绑定了。使用对象引用来访问静态方法只是Java设计者给程序员的自由。我们应该直接使用类名来访问静态方法，而不要使用对象引用来访问。

例子，让我们看一个例子，来看看重写静态方法会发生什么：

<pre>
class SuperClass{
......
public static void staticMethod(){
System.out.println("SuperClass: inside staticMethod");
}
......
}

public class SubClass extends SuperClass{
......
//overriding the static method
public static void staticMethod(){
System.out.println("SubClass: inside staticMethod");
}

......
public static void main(String []args){
......
SuperClass superClassWithSuperCons = new SuperClass();
SuperClass superClassWithSubCons = new SubClass();
SubClass subClassWithSubCons = new SubClass();

superClassWithSuperCons.staticMethod();
superClassWithSubCons.staticMethod();
subClassWithSubCons.staticMethod();
...
}

}
</pre>

输出：

<pre>
SuperClass: inside staticMethod
SuperClass: inside staticMethod
SubClass: inside staticMethod
</pre>

注意第二行输出。假设staticMethod方法被重写了，它的结果应该和第三行一样，也是调用运行时的类型SubClass的staticMethod()，而不是SuperClass的staticMethod()方法。这也就证明了静态方法是在编译阶段使用了编译类型信息，进行静态绑定的。

http://geekexplains.blogspot.hk/2008/06/can-you-override-static-methods-in-java.html
