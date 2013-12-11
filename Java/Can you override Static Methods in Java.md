## Can you override Static Methods in Java?

## 可以重写静态方法吗？

### Question: Can you override Static Methods in Java?

### 问：你可以重写静态方法吗？

Answer: Well... the answer is NO if you think from the perspective of how an overriden method should behave in Java. But, you don't get any compiler error if you try to override a static method. That means, if you try to override, Java doesn't stop you doing that; but you certainly don't get the same effect as you get for non-static methods. Overriding in Java simply means that the particular method would be called based on the run time type of the object and not on the compile time type of it (which is the case with overriden static methods). Okay... any guesses for the reason why do they behave strangely? Because they are class methods and hence access to them is always resolved during compile time only using the compile time type information. Accessing them using object references is just an extra liberty given by the designers of Java and we should certainly not think of stopping that practice only when they restrict it :-)

答：如果从重写方法会有什么特点来看，我们是不能重写静态方法的。虽然就算你重写静态方法，编译器也不会报错。也就是说，如果你试图重写静态方法，Java不会阻止你这么做，但你却得不到预期的结果（重写仅对非静态方法有用）。重写指的是根据运行时对象的类型来决定调用哪个方法，而不是根据编译时的类型。

Example: let's try to see what happens if we try overriding a static method:-

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
Output:-
<pre>
SuperClass: inside staticMethod
SuperClass: inside staticMethod
SubClass: inside staticMethod
</pre>

Notice the second line of the output. Had the staticMethod been overriden this line should have been identical to the third line as we're invoking the 'staticMethod()' on an object of Runtime Type as 'SubClass' and not as 'SuperClass'. This confirms that the static methods are always resolved using their compile time type information only.

http://geekexplains.blogspot.hk/2008/06/can-you-override-static-methods-in-java.html
