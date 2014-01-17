## 集合类接口和类层次关系图

### 1. Collection vs Collections

首先，“Collection”和“Collections”是两个不同的概念。你可以从下面的层次关系图中看到，"Collection"是集合层次的顶层接口，而"Collections"是提供了操作集合类型的静态方法的类。

### 2. 集合类层次关系图

下图是集合类的层次关系图


### 3. Map的类层次结构关系图

下图是Map的类层次结构关系图



### 4. 集合类总结



### 5. 代码示例

下面是一个简单的集合的例子：

<pre class="brush: java; gutter: true">
List<String> a1 = new ArrayList<String>();
a1.add("Program");
a1.add("Creek");
a1.add("Java");
a1.add("Java");
System.out.println("ArrayList Elements");
System.out.print("\t" + a1 + "\n");
 
List<String> l1 = new LinkedList<String>();
l1.add("Program");
l1.add("Creek");
l1.add("Java");
l1.add("Java");
System.out.println("LinkedList Elements");
System.out.print("\t" + l1 + "\n");
 
Set<String> s1 = new HashSet<String>(); // or new TreeSet() will order the elements;
s1.add("Program");
s1.add("Creek");
s1.add("Java");
s1.add("Java");
s1.add("tutorial");
System.out.println("Set Elements");
System.out.print("\t" + s1 + "\n");
 
Map<String, String> m1 = new HashMap<String, String>(); // or new TreeMap() will order based on keys
m1.put("Windows", "2000");
m1.put("Windows", "XP");
m1.put("Language", "Java");
m1.put("Website", "programcreek.com");
System.out.println("Map Elements");
System.out.print("\t" + m1);
</pre>

输出:

<pre class="brush: plain; gutter: true">
ArrayList Elements
	[Program, Creek, Java, Java]
LinkedList Elements
	[Program, Creek, Java, Java]
Set Elements
	[tutorial, Creek, Program, Java]
Map Elements
	{Windows=XP, Website=programcreek.com, Language=Java}
</pre>

http://www.programcreek.com/2009/02/the-interface-and-class-hierarchy-for-collections/
