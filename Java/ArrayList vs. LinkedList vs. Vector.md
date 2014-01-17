## ArrayList vs. LinkedList vs. Vector

### 1. List概览

List，就像它的名字暗示的一样，是一组排列有序的元素。当我们讨论List的时候，很容易将它和Set作比较。Set是一组唯一的而且排列无序的元素。

下图是集合类的层次结构图。你可以总体上知道我们今天讨论的主题。

### 2. ArrayList vs. LinkedList vs. Vector

从上图可知，它们都实现了List接口。它们的用法差不多，主要的区别在于它们对于不同操作的操作速度不同。

ArrayList是可以改变大小的数组。当有元素添加到ArrayList中去时，它的大小动态的增加。元素可以直接通过get()和set()方法进行访问，因为ArrayList实际上是数组。LinkedList是个双向链表。它的add()和remove()方法比ArrayList快，但是get()和set()方法却比ArrayList慢。Vector和ArrayList类似，但是Vector是同步的。如果在单线程的环境下，使用ArrayList是更好的选择（节省同步的开销）。当添加元素的时候，Vector和ArrayList需要更多的空间：Vector每次将数组的大小增加一倍，而ArrayList每次增加50%。

LinkedList还实现了Queue接口，这样就比ArrayList和Vector多出了一些方法如offer(), peek(), poll()等。

注意：ArrayList的初始化大小很小。我们应该设置一个比较高的初始化大小，这样可以避免重新改变大小。

### 3. ArrayList的例子

<pre class="brush: java; gutter: true">
ArrayList<Integer> al = new ArrayList<Integer>();
al.add(3);
al.add(2);		
al.add(1);
al.add(4);
al.add(5);
al.add(6);
al.add(6);
 
Iterator<Integer> iter1 = al.iterator();
while(iter1.hasNext()){
	System.out.println(iter1.next());
}
</pre>

### 4. LinkedList example

<pre class="brush: java; gutter: true">
LinkedList<Integer> ll = new LinkedList<Integer>();
ll.add(3);
ll.add(2);		
ll.add(1);
ll.add(4);
ll.add(5);
ll.add(6);
ll.add(6);
 
Iterator<Integer> iter2 = ll.iterator();
while(iter2.hasNext()){
	System.out.println(iter2.next());
}
</pre>

由上可见，它们的用法相同，主要的区别在于它们内部的实现，以及操作的复杂度的不同。

### 5. Vector

Vector几乎和ArrayList相等，主要的区别在于Vector是同步的。正因为此，Vector比ArrayList的开销更大。通常大部分程序员都使用ArrayList，他们可以写代码进行同步。

### 6. ArrayList vs. LinkedList的性能比较

时间复杂度如下：

- * add() in the table refers to add(E e), and remove() refers to remove(int index)

ArrayList has O(n) time complexity for arbitrary indices of add/remove, but O(1) for the operation at the end of the list.
LinkedList has O(n) time complexity for arbitrary indices of add/remove, but O(1) for operations at end/beginning of the List.

- add()

I use the following code to test their performance:

<pre class="brush: java; gutter: true">
ArrayList<Integer> arrayList = new ArrayList<Integer>();
LinkedList<Integer> linkedList = new LinkedList<Integer>();
 
// ArrayList add
long startTime = System.nanoTime();
 
for (int i = 0; i < 100000; i++) {
	arrayList.add(i);
}
long endTime = System.nanoTime();
long duration = endTime - startTime;
System.out.println("ArrayList add:  " + duration);
 
// LinkedList add
startTime = System.nanoTime();
 
for (int i = 0; i < 100000; i++) {
	linkedList.add(i);
}
endTime = System.nanoTime();
duration = endTime - startTime;
System.out.println("LinkedList add: " + duration);
 
// ArrayList get
startTime = System.nanoTime();
 
for (int i = 0; i < 10000; i++) {
	arrayList.get(i);
}
endTime = System.nanoTime();
duration = endTime - startTime;
System.out.println("ArrayList get:  " + duration);
 
// LinkedList get
startTime = System.nanoTime();
 
for (int i = 0; i < 10000; i++) {
	linkedList.get(i);
}
endTime = System.nanoTime();
duration = endTime - startTime;
System.out.println("LinkedList get: " + duration);
 
 
 
// ArrayList remove
startTime = System.nanoTime();
 
for (int i = 9999; i >=0; i--) {
	arrayList.remove(i);
}
endTime = System.nanoTime();
duration = endTime - startTime;
System.out.println("ArrayList remove:  " + duration);
 
 
 
// LinkedList remove
startTime = System.nanoTime();
 
for (int i = 9999; i >=0; i--) {
	linkedList.remove(i);
}
endTime = System.nanoTime();
duration = endTime - startTime;
System.out.println("LinkedList remove: " + duration);
</pre>

And the output is:

<pre class="brush: plain; gutter: true">
ArrayList add:  13265642
LinkedList add: 9550057
ArrayList get:  1543352
LinkedList get: 85085551
ArrayList remove:  199961301
LinkedList remove: 85768810
</pre>
The difference of their performance is obvious. LinkedList is faster in add and remove, but slower in get. Based on the complexity table and testing results, we can figure out when to use ArrayList or LinkedList. In brief, LinkedList should be preferred if:

there are no large number of random access of element
there are a large number of add/remove operations
