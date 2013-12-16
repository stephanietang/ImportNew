## 如何使用建造者模式(Builder Pattern)创建不可变类

我写过一篇《如何创建不可变类》。这篇文章中，我们将看到如何使用建造者模式创建不可变类。当构造器中的参数很多时，并且参数的顺序会给人造成困扰的时候，那么使用建造者模式来创建不可变类就是非常好的方法了。


### 使用建造者模式来创建不可变类

下面是使用建造者模式来创建不可变类的例子：

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

下面的测试代码为我们测试到底创建的对象是不是不可变的。

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
         
        //check that immutable class properties are not changed
        System.out.println(immutableClass1.getProperties());
    }
 
}
</pre>

### 重要的知识点

- 不可变类只有getter方法
- 不可变类只有一个私有的构造器，以Builder对象作为参数来创建不可变对象
- 如果不可变类的成员变量是可变的（譬如HashMap），我们需要使用深拷贝(deep copy)或者克隆来防止成员变量被更改
- 当可选的成员变量很多的时候，使用建造者模式创建不可变类是不错的方法

http://www.journaldev.com/1432/how-to-create-immutable-class-in-java-using-builder-pattern
