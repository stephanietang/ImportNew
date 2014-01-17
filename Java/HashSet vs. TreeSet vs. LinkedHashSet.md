## HashSet vs. TreeSet vs. LinkedHashSet

Set集合不包含重复的元素，这是使用Set的主要原因。有三种常见的Set实现——HashSet, TreeSet和LinkedHashSet。什么时候使用它们，使用哪个是个重要的问题。总体而言，如果你需要一个访问快速的Set，你应该使用HashSet；当你需要一个排序的Set，你应该使用TreeSet；当你需要记录下插入时的顺序时，你应该使用LinedHashSet。

### 1. Set接口

Set接口继承了Collection接口。Set集合中不能包含重复的元素，每个元素必须是唯一的。你只需将元素加入set中，重复的元素会自动移除。

### 2. HashSet vs. TreeSet vs. LinkedHashSet

HashSet是采用hash表来实现的。其中的元素没有按顺序排列，add()、remove()以及contains()等方法都是复杂度为O(1)的方法。

TreeSet是采用树结构实现(红黑树算法)。元素是按顺序进行排列，但是add()、remove()以及contains()等方法都是复杂度为O(log (n))的方法。它还提供了一些方法来处理排序的set，如first(), last(), headSet(), tailSet()等等。

LinkedHashSet介于HashSet和TreeSet之间。它也是一个hash表，但是同时维护了一个双链表来记录插入的顺序。基本方法的复杂度为O(1)。

### 3. TreeSet的例子

<pre class="brush: java; gutter: true">
TreeSet<Integer> tree = new TreeSet<Integer>();
tree.add(12);
tree.add(63);
tree.add(34);
tree.add(45);
 
Iterator<Integer> iterator = tree.iterator();
System.out.print("Tree set data: ");
while (iterator.hasNext()) {
    System.out.print(iterator.next() + " ");
}
</pre>

输出如下：

Tree set data: 12 34 45 63 

现在让我们定义一个Dog类：

class Dog {
	int size;
 
	public Dog(int s) {
		size = s;
	}
 
	public String toString() {
		return size + "";
	}
}

我们将“dog”添加到TreeSet中：

<pre class="brush: java; gutter: true">
import java.util.Iterator;
import java.util.TreeSet;
 
public class TestTreeSet {
	public static void main(String[] args) {
		TreeSet<Dog> dset = new TreeSet<Dog>();
		dset.add(new Dog(2));
		dset.add(new Dog(1));
		dset.add(new Dog(3));
 
		Iterator<Dog> iterator = dset.iterator();
 
		while (iterator.hasNext()) {
			System.out.print(iterator.next() + " ");
		}
	}
}
</pre>

编译正常，但是运行时出错：

<pre class="brush: plain; gutter: true">
Exception in thread "main" java.lang.ClassCastException: collection.Dog cannot be cast to java.lang.Comparable
	at java.util.TreeMap.put(Unknown Source)
	at java.util.TreeSet.add(Unknown Source)
	at collection.TestTreeSet.main(TestTreeSet.java:22)
</pre>

因为TreeSet是有序的，Dog类必须实现java.lang.Comparable的compareTo()方法才行:

<pre class="brush: java; gutter: true">
class Dog implements Comparable<Dog>{
	int size;
 
	public Dog(int s) {
		size = s;
	}
 
	public String toString() {
		return size + "";
	}
 
	@Override
	public int compareTo(Dog o) {
	        return size - o.size;
	}
}
</pre>

输出:
<pre class="brush: plain; gutter: true">
1 2 3 
</pre>

### 4. HashSet的例子

<pre class="brush: java; gutter: true">
HashSet<Dog> dset = new HashSet<Dog>();
dset.add(new Dog(2));
dset.add(new Dog(1));
dset.add(new Dog(3));
dset.add(new Dog(5));
dset.add(new Dog(4));
Iterator<Dog> iterator = dset.iterator();
while (iterator.hasNext()) {
	System.out.print(iterator.next() + " ");
}
</pre>

输出：

<pre class="brush: plain; gutter: true">
5 3 2 1 4 
</pre>

注意输出顺序是不确定的。

### 5. LinkedHashSet的例子

<pre class="brush: java; gutter: true">
LinkedHashSet<Dog> dset = new LinkedHashSet<Dog>();
dset.add(new Dog(2));
dset.add(new Dog(1));
dset.add(new Dog(3));
dset.add(new Dog(5));
dset.add(new Dog(4));
Iterator<Dog> iterator = dset.iterator();
while (iterator.hasNext()) {
	System.out.print(iterator.next() + " ");
}
</pre>

输出的顺序时确定的，就是插入的顺序。
<pre class="brush: plain; gutter: true">
2 1 3 5 4 
</pre>

### 6. 性能测试

下面的代码测试了以上三个类的add()方法的性能。

<pre class="brush: java; gutter: true">
public static void main(String[] args) {
 
	Random r = new Random();
 
	HashSet<Dog> hashSet = new HashSet<Dog>();
	TreeSet<Dog> treeSet = new TreeSet<Dog>();
	LinkedHashSet<Dog> linkedSet = new LinkedHashSet<Dog>();
 
	// start time
	long startTime = System.nanoTime();
 
	for (int i = 0; i < 1000; i++) {
		int x = r.nextInt(1000 - 10) + 10;
		hashSet.add(new Dog(x));
	}
	// end time
	long endTime = System.nanoTime();
	long duration = endTime - startTime;
	System.out.println("HashSet: " + duration);
 
	// start time
	startTime = System.nanoTime();
	for (int i = 0; i < 1000; i++) {
		int x = r.nextInt(1000 - 10) + 10;
		treeSet.add(new Dog(x));
	}
	// end time
	endTime = System.nanoTime();
	duration = endTime - startTime;
	System.out.println("TreeSet: " + duration);
 
	// start time
	startTime = System.nanoTime();
	for (int i = 0; i < 1000; i++) {
		int x = r.nextInt(1000 - 10) + 10;
		linkedSet.add(new Dog(x));
	}
	// end time
	endTime = System.nanoTime();
	duration = endTime - startTime;
	System.out.println("LinkedHashSet: " + duration);
 
}
</pre>

从输出看来，HashSet是最快的：

<pre class="brush: plain; gutter: true">
HashSet: 2244768
TreeSet: 3549314
LinkedHashSet: 2263320
</pre>

* 这个测试并不是非常精确，但足以反映基本的情况。

http://www.programcreek.com/2013/03/hashset-vs-treeset-vs-linkedhashset/
