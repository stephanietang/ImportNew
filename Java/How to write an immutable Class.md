## How to write an immutable Class?

## 如何写一个不可变类？

Immutable objects are instances whose state doesn’t change after it has been initialized. For example, String is an immutable class and once instantiated its value never changes.

Read: [Why String in immutable in Java]()

Immutable objects are good for caching purpose because you don’t need to worry about the value changes. Other benefit of immutable class is that it is inherently thread-safe, so you don’t need to worry about thread safety in case of multi-threaded environment.

Read: Java Thread Tutorial and Java Multi-Threading Interview Questions.

Here I am providing a way to create immutable class via an example for better understanding.

To create a class immutable, you need to follow following steps:

Declare the class as final so it can’t be extended.
Make all fields private so that direct access is not allowed.
Don’t provide setter methods for variables
Make all mutable fields final so that it’s value can be assigned only once.
Initialize all the fields via a constructor performing deep copy.
Perform cloning of objects in the getter methods to return a copy rather than returning the actual object reference.
To understand points 4 and 5, let’s run the sample Final class that works well and values doesn’t get altered after instantiation.

FinalClassExample.java
<pre>
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
     * Accessor function for mutable objects
     */
    public HashMap<String, String> getTestMap() {
        //return testMap;
        return (HashMap<String, String>) testMap.clone();
    }
 
    /**
     * Constructor performing Deep Copy
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
     * Constructor performing Shallow Copy
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
     * To test the consequences of Shallow Copy and how to avoid it with Deep Copy for creating immutable classes
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

Output of the above program is:

<pre>
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

Now lets comment the constructor providing deep copy and uncomment the constructor providing shallow copy. Also uncomment the return statement in getTestMap() method that returns the actual object reference and then execute the program once again.

<pre>
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

As you can see from the output, HashMap values got changed because of shallow copy in the constructor and providing direct reference to the original object in the getter function.

If I have missed something here, feel free to comment.

Further Reading: If the immutable class has a lot of attributes and some of them are optional, we can use [builder pattern to create immutable classes](http://www.journaldev.com/1432/how-to-create-immutable-class-in-java-using-builder-pattern).

http://www.journaldev.com/129/how-to-write-an-immutable-class

http://javarevisited.blogspot.hk/2011/12/final-variable-method-class-java.html
