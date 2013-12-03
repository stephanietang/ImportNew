Dynamic Binding vs Static Binding in Java

 

Difference between Dynamic Binding & Static Binding in Java

Dynamic Binding or Late Binding

Dynamic Binding refers to the case where compiler is not able to resolve the call and the binding is done at runtime only. Let's try to understand this. Suppose we have a class named 'SuperClass' and another class named 'SubClass' extends it. Now a 'SuperClass' reference can be assigned to an object of the type 'SubClass' as well. If we have a method (say 'someMethod()') in the 'SuperClass' which we override in the 'SubClass' then a call of that method on a 'SuperClass' reference can only be resolved at runtime as the compiler can't be sure of what type of object this reference would be pointing to at runtime.

...
SuperClass superClass1 = new SuperClass();
SuperClass superClass2 = new SubClass();
...

superClass1.someMethod(); // SuperClass version is called
superClass2.someMethod(); // SubClass version is called
....

Here, we see that even though both the object references superClass1 and superClass2 are of type 'SuperClass' only, but at run time they refer to the objects of types 'SuperClass' and 'SubClass' respectively.

Hence, at compile time the compiler can't be sure if the call to the method 'someMethod()' on these references actually refer to which version of the method - the super class version or the sub class version.

Thus, we see that dynamic binding in Java simply binds the method calls (inherited methods only as they can be overriden in a sub class and hence compiler may not be sure of which version of the method to call) based on the actual object type and not on the declared type of the object reference.

Static Binding or Early Binding

If the compiler can resolve the binding at the compile time only then such a binding is called Static Binding or Early Binding. All the instance method calls are always resolved at runtime, but all the static method calls are resolved at compile time itself and hence we have static binding for static method calls. Because static methods are class methods and hence they can be accessed using the class name itself (in fact they are encourgaed to be used using their corresponding class names only and not by using the object references) and therefore access to them is required to be resolved during compile time only using the compile time type information. That's the reason why static methods can not actually be overriden. Read more - Can you override static methods in Java?

Similarly, access to all the member variables in Java follows static binding as Java doesn't support (in fact, it discourages) polymorphic behavior of member variables. For example:-

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

Output:-
Some Variable in SuperClass
Some Variable in SuperClass

We can observe that in both the cases, the member variable is resolved based on the declared type of the object reference only, which the compiler is capable of finding as early as at the compile time only and hence a static binding in this case. Another example of static binding is that of 'private' methods as they are never inherited and the compile can resolve calls to any private method at compile time only.

http://geekexplains.blogspot.hk/2008/06/dynamic-binding-vs-static-binding-in.html
