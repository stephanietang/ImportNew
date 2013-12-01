## What is final in Java? Final variable , Method and Class Example

Final in java is very important keyword and can be applied to class, method, and variables in Java. In this java final tutorial we will see what is final keyword in Java, what does it mean by making final variable, final method and final class in java and what are primary benefits of using final keywords in Java and finally some examples of final in Java. Final is often used along with static keyword in Java to make static final constant and you will see how final in Java can increase performance of Java application.

Example of Final variable, Final method and Class in Java

What is final keyword in Java?

Final keyword, final variable, final method and class in java exampleFinal is a keyword or reserved word in java and can be applied to member variables, methods, class and local variables in Java. Once you make a reference final you are not allowed to change that reference and compiler will verify this and raise compilation error if you try to re-initialized final variables in java.

What is final variable in Java?
Any variable either member variable or local variable (declared inside method or block) modified by final keyword is called final variable. Final variables are often declare with static keyword in java and treated as constant. Here is an example of final variable in Java


<pre class="brush: java; gutter: true">
	public static final String LOAN = "loan";
	LOAN = new String("loan") //invalid compilation error
</pre>

Final variables are by default read-only.

### What is final method in Java
Final keyword in java can also be applied to methods. A java method with final keyword is called final method and it can not be overridden in sub-class. You should make a method final in java if you think it’s complete and its behavior should remain constant in sub-classes. Final methods are faster than non-final methods because they are not required to be resolved during run-time and they are bonded on compile time. Here is an example of final method in Java:

<pre class="brush: java; gutter: true">
	class PersonalLoan{
		public final String getName(){
	    	return "personal loan";
		}
	}
	
	class CheapPersonalLoan extends PersonalLoan{
		@Override
		public final String getName(){
	    	return "cheap personal loan"; //compilation error: overridden method is final
	    }
	}

</pre>

### What is final Class in Java

Java class with final modifier is called final class in Java. Final class is complete in nature and can not be sub-classed or inherited. Several classes in Java are final e.g. String, Integer and other wrapper classes. Here is an example of final class in java

<pre class="brush: java; gutter: true">
	final class PersonalLoan{

	}

	class CheapPersonalLoan extends PersonalLoan{  //compilation error: cannot inherit from final class
 
}
</pre>

### Benefits of final keyword in Java

Here are few benefits or advantage of using final keyword in Java:

1. Final keyword improves performance. Not just JVM can cache final variable but also application can cache frequently use final variables.

2. Final variables are safe to share in multi-threading environment without additional synchronization overhead.

3. Final keyword allows JVM to optimized method, variable or class.


### Final and Immutable Class in Java

Final keyword helps to write immutable class. Immutable classes are the one which can not be modified once it gets created and String is primary example of immutable and final class which I have discussed in detail on Why String is final or immutable in Java. Immutable classes offer several benefits one of them is that they are effectively read-only and can be safely shared in between multiple threads without any synchronization overhead. You can not make a class immutable without making it final and hence final keyword is required to make a class immutable in java.

### Example of Final in Java

Java has several system classes in JDK which are final, some example of final classes are String, Integer, Double and other wrapper classes. You can also use final keyword to make your code better whenever it required. See relevant section of java final tutorial for example of final variable, final method and final class in Java.


### Important points on final in Java

1. Final keyword can be applied to member variable, local variable, method or class in Java.
2. Final member variable must be initialized at the time of declaration or inside constructor, failure to do so will result in compilation error.

3. You can not reassign value to final variable in Java.

4. Local final variable must be initializing during declaration.

5. Only final variable is accessible inside anonymous class in Java.

6. Final method can not be overridden in Java.

7. Final class can not be inheritable in Java.

8. Final is different than finally keyword which is used on Exception handling in Java.

9. Final should not be confused with finalize() method which is declared in object class and called before an object is garbage collected by JVM.

10. All variable declared inside java interface are implicitly final.

11. Final and abstract are two opposite keyword and a final class can not be abstract in java.

12. Final methods are bonded during compile time also called static binding.

13. Final variables which is not initialized during declaration are called blank final variable and must be initialized on all constructor either explicitly or by calling this(). Failure to do so compiler will complain as "final variable (name) might not be initialized".

14. Making a class, method or variable final in Java helps to improve performance because JVM gets an opportunity to make assumption and optimization.


15. As per Java code convention final variables are treated as constant and written in all Caps e.g.

<pre class="brush: java; gutter: true">
	private final int COUNT=10;	
</pre>

16. Making a collection reference variable final means only reference can not be changed but you can add, remove or change object inside collection. For example:

<pre class="brush: java; gutter: true">
	private final List Loans = new ArrayList();
	list.add(“home loan”);  //valid
	list.add("personal loan"); //valid
	loans = new Vector();  //not valid	
</pre>

That’s all on final in Java. We have seen what final variable, final method is and final class in Java and what does those mean. In Summary whenever possible start using final in java it would result in better and faster code.


http://javarevisited.blogspot.hk/2011/12/final-variable-method-class-java.html