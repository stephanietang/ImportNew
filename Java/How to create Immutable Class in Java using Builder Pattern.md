## How to create Immutable Class in Java using Builder Pattern

In my last post, I explained about Builder Pattern in Java. Sometime back I wrote an article about how to create immutable class in java. In this post, we will see how easy its to create immutable classes using Builder pattern. Using builder pattern to create immutable class is a good approach when the number of arguments in the Constructor is more that can cause confusion in their ordering.

### Immutable Class using Builder Pattern

Here is the implementation of an immutable class using Builder pattern.

ImmutableClass.java
<pre>
package com.journaldev.design.builder;
 
import java.util.HashMap;
 
public class ImmutableClass {
     
    //required fields
    private int id;
    private String name;
     
    //optional fields
    private HashMap<String, String> properties;
    private String company;
     
    public int getId() {
        return id;
    }
 
    public String getName() {
        return name;
    }
 
    public HashMap<String, String> getProperties() {
        //return cloned object to avoid changing it by the client application
        return (HashMap<String, String>) properties.clone();
    }
 
    public String getCompany() {
        return company;
    }
 
    private ImmutableClass(ImmutableClassBuilder builder) {
        this.id = builder.id;
        this.name = builder.name;
        this.properties = builder.properties;
        this.company = builder.company;
    }
     
        //Builder class
    public static class ImmutableClassBuilder{
        //required fields
        private int id;
        private String name;
         
        //optional fields
        private HashMap<String, String> properties;
        private String company;
         
        public ImmutableClassBuilder(int i, String nm){
            this.id=i;
            this.name=nm;
        }
         
        public ImmutableClassBuilder setProperties(HashMap<String,String> hm){
            this.properties = (HashMap<String, String>) hm.clone();
            return this;
        }
         
        public ImmutableClassBuilder setCompany(String comp){
            this.company = comp;
            return this;
        }
         
        public ImmutableClass build(){
            return new ImmutableClass(this);
        }
    }
}
</pre>

Here is the test program to check if the object we create is immutable or not.

ImmutableBuilderTest.java
<pre>
package com.journaldev.design.test;
 
import java.util.HashMap;
 
import com.journaldev.design.builder.ImmutableClass;
 
public class ImmutableBuilderTest {
 
    public static void main(String[] args) {
         
        // Getting immutable class only with required parameters
        ImmutableClass immutableClass = new ImmutableClass.ImmutableClassBuilder(1, "Pankaj").build();
         
        HashMap<String,String> hm = new HashMap<String, String>();
        hm.put("Name", "Pankaj");
        hm.put("City", "San Jose");
        // Getting immutable class with optional parameters
        ImmutableClass immutableClass1 = new ImmutableClass.ImmutableClassBuilder(1, "Pankaj")
                                                     .setCompany("Apple").setProperties(hm).build();
         
        //Testing immutability
        HashMap<String,String> hm1 = immutableClass1.getProperties();
         
        //lets modify the Object passed as argument or get from the Object
        hm1.put("test", "test");
        hm.put("test", "test");
         
        //check that immutable class properties are not changed
        System.out.println(immutableClass1.getProperties());
    }
 
}
</pre>

### Important Points

- The immutable class should have only getter methods.
- The immutable class will have a private constructor with Builder object as parameter that will be used to create the immutable class.
- If the immutable class attributes are not immutable, for example HashMap, we should perform deep copy or cloning to avoid modification of its attributes.
- Using Builder pattern is easy when number of optional attributes are more in the immutable class.
