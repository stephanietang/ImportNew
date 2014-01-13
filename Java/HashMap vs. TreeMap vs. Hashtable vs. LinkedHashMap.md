## HashMap vs. TreeMap vs. Hashtable vs. LinkedHashMap

Map是最重要的数据结构。这篇文章中，我会带你们看看HashMap, TreeMap, HashTable和LinkedHashMap的区别。

### 1. Map概览

Java SE中有四种常见的Map实现——HashMap, TreeMap, Hashtable和LinkedHashMap。如果我们使用一句话来分别概括它们的特点，就是：

- HashMap就是一张hash表，键和值都没有排序。
- TreeMap以红-黑树结构为基础，键值按顺序排列。
- LinkedHashMap保存了插入时的顺序。
- Hashtable是同步的(而HashMap是不同步的)。所以如果线程安全的环境下应该使用HashMap，因为Hashtable对同步有额外的开销。

2. HashMap

如果HashMap的键(key)是自定义的对象，那么需要按规则定义它的equals()和hashCode()方法。

<pre class="brush: java; gutter: true">
class Dog {
	String color;
 
	Dog(String c) {
		color = c;
	}
	public String toString(){	
		return color + " dog";
	}
}
 
public class TestHashMap {
	public static void main(String[] args) {
		HashMap<Dog, Integer> hashMap = new HashMap<Dog, Integer>();
		Dog d1 = new Dog("red");
		Dog d2 = new Dog("black");
		Dog d3 = new Dog("white");
		Dog d4 = new Dog("white");
 
		hashMap.put(d1, 10);
		hashMap.put(d2, 15);
		hashMap.put(d3, 5);
		hashMap.put(d4, 20);
 
		//print size
		System.out.println(hashMap.size());
 
		//loop HashMap
		for (Entry<Dog, Integer> entry : hashMap.entrySet()) {
			System.out.println(entry.getKey().toString() + " - " + entry.getValue());
		}
	}
}
</pre>

输出:

<pre class="brush: plain; gutter: true">
4
white dog - 5
black dog - 15
red dog - 10
white dog - 20
</pre>


注意，我们错误的将"white dogs"添加了两次，但是HashMap却接受了两只"white dogs"。这不合理（因为HashMap的键不应该重复），我们会搞不清楚真正有多少白色的狗存在。

Dog类应该定义如下：

<pre class="brush: java; gutter: true">
class Dog {
	String color;
 
	Dog(String c) {
		color = c;
	}
 
	public boolean equals(Object o) {
		return ((Dog) o).color == this.color;
	}
 
	public int hashCode() {
		return color.length();
	}
 
	public String toString(){	
		return color + " dog";
	}
}
</pre>

现在输出结果如下:

<pre class="brush: plain; gutter: true">
3
red dog - 10
white dog - 20
black dog - 15
</pre>

输出结果如上是因为HashMap不允许有两个相等的元素存在。默认情况下（也就是类没有实现hashCode()和equals()方法时），会使用Object类中的这两个方法。Object类中的hashCode()对于不同的对象会返回不同的整数，而只有两个引用指向的同样的对象时equals()才会返回true。如果你不是很了解hashCode()和equals()的规则，可以看看[这篇文章](http://www.programcreek.com/2011/07/java-equals-and-hashcode-contract/)。

来看看[HashMap最常用的方法](http://www.programcreek.com/2013/04/frequently-used-methods-of-java-hashmap/)，如迭代、打印等。

### 3. TreeMap

TreeMap的键按顺序排列。让我们先看个例子看看什么叫作“键按顺序排列”。

<pre class="brush: java; gutter: true">
class Dog {
	String color;
 
	Dog(String c) {
		color = c;
	}
	public boolean equals(Object o) {
		return ((Dog) o).color == this.color;
	}
 
	public int hashCode() {
		return color.length();
	}
	public String toString(){	
		return color + " dog";
	}
}
 
public class TestTreeMap {
	public static void main(String[] args) {
		Dog d1 = new Dog("red");
		Dog d2 = new Dog("black");
		Dog d3 = new Dog("white");
		Dog d4 = new Dog("white");
 
		TreeMap<Dog, Integer> treeMap = new TreeMap<Dog, Integer>();
		treeMap.put(d1, 10);
		treeMap.put(d2, 15);
		treeMap.put(d3, 5);
		treeMap.put(d4, 20);
 
		for (Entry<Dog, Integer> entry : treeMap.entrySet()) {
			System.out.println(entry.getKey() + " - " + entry.getValue());
		}
	}
}
</pre>

输出:
<pre class="brush: plain; gutter: true">
Exception in thread "main" java.lang.ClassCastException: collection.Dog cannot be cast to java.lang.Comparable
	at java.util.TreeMap.put(Unknown Source)
	at collection.TestHashMap.main(TestHashMap.java:35)
</pre>

因为TreeMap按照键的顺序进行排列对象，所以键的对象之间需要能够比较，所以就要实现Comparable接口。你可以使用String作为键，String已经实现了Comparable接口。

我们来修改下Dog类，让它实现Comparable接口。

<pre class="brush: java; gutter: true">
class Dog implements Comparable<Dog>{
	String color;
	int size;
 
	Dog(String c, int s) {
		color = c;
		size = s;
	}
 
	public String toString(){	
		return color + " dog";
	}
 
	@Override
	public int compareTo(Dog o) {
		return  o.size - this.size;
	}
}
 
public class TestTreeMap {
	public static void main(String[] args) {
		Dog d1 = new Dog("red", 30);
		Dog d2 = new Dog("black", 20);
		Dog d3 = new Dog("white", 10);
		Dog d4 = new Dog("white", 10);
 
		TreeMap<Dog, Integer> treeMap = new TreeMap<Dog, Integer>();
		treeMap.put(d1, 10);
		treeMap.put(d2, 15);
		treeMap.put(d3, 5);
		treeMap.put(d4, 20);
 
		for (Entry<Dog, Integer> entry : treeMap.entrySet()) {
			System.out.println(entry.getKey() + " - " + entry.getValue());
		}
	}
}
</pre>

输出:

<pre class="brush: plain; gutter: true">
red dog - 10
black dog - 15
white dog - 20
</pre>

结果根据键的排列顺序进行输出，在我们的例子中根据size排序的。

如果我们将“Dog d4 = new Dog(“white”, 10);”替换成“Dog d4 = new Dog(“white”, 40);”，那么输出会变成：

<pre class="brush: plain; gutter: true">
white dog - 20
red dog - 10
black dog - 15
white dog - 5
</pre>

这是因为TreeMap使用compareTo()方法来比较键值的大小，size不相等的狗是不同的狗。

### 4. Hashtable

Java文档写道：

HashMap类和Hashtable类几乎相同，不同之处在于HashMap是不同步的，也允许接受null键和null值。

### 5. LinkedHashMap

LinkedHashMap is a subclass of HashMap. That means it inherits the features of HashMap. In addition, the linked list preserves the insertion-order.

Let’s replace the HashMap with LinkedHashMap using the same code used for HashMap.

LinkedHashMap是HashMap的子类，所以LinkedHashMap继承了HashMap的一些属性，它在HashMap基础上增加的特性就是保存了插入对象的顺序。

<pre class="brush: java; gutter: true">
class Dog {
	String color;
 
	Dog(String c) {
		color = c;
	}
 
	public boolean equals(Object o) {
		return ((Dog) o).color == this.color;
	}
 
	public int hashCode() {
		return color.length();
	}
 
	public String toString(){	
		return color + " dog";
	}
}
 
public class TestHashMap {
	public static void main(String[] args) {
 
		Dog d1 = new Dog("red");
		Dog d2 = new Dog("black");
		Dog d3 = new Dog("white");
		Dog d4 = new Dog("white");
 
		LinkedHashMap<Dog, Integer> linkedHashMap = new LinkedHashMap<Dog, Integer>();
		linkedHashMap.put(d1, 10);
		linkedHashMap.put(d2, 15);
		linkedHashMap.put(d3, 5);
		linkedHashMap.put(d4, 20);
 
		for (Entry<Dog, Integer> entry : linkedHashMap.entrySet()) {
			System.out.println(entry.getKey() + " - " + entry.getValue());
		}		
	}
}
</pre>

输出:
<pre class="brush: plain; gutter: true">
red dog - 10
black dog - 15
white dog - 20
</pre>

如果我们使用HashMap的话，输出将会如下，会打乱插入的顺序：

<pre class="brush: plain; gutter: true">
red dog - 10
white dog - 20
black dog - 15
</pre>

http://www.programcreek.com/2013/03/hashmap-vs-treemap-vs-hashtable-vs-linkedhashmap/
